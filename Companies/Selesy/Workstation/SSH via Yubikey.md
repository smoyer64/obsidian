---
tags:
  - workstation
---
## Key configuration

That looks exactly like it should. Notice that **`gpg-agent-ssh.socket`** and **`gnome-keyring-daemon.socket`** are completely absent from the running units. This is the "clean room" state we were looking for. The 90-second timeout was likely the systemd user manager waiting for those conflicting sockets to gracefully exit, which they couldn't do because they were essentially fighting over the same file descriptor territory.

Now that the "vandalism" is repaired and the environment is stock, we can implement your YubiKey-backed identity in a way that is modern, secure, and—most importantly—will not break your session again.

### The Modern YubiKey Identity (FIDO2 / Resident Keys)

Since you have three YubiKeys and want them to represent your identity while the local SSH key represents the machine, we’ll use **Resident Keys**. This stores a "key handle" directly on the YubiKey's hardware.

#### 1. Generate the Resident Keys
Perform this for each of your three YubiKeys. You'll need to enter your YubiKey PIN and touch the device during the process.

```bash
# Plugin YubiKey #1
ssh-keygen -t ed25519-sk -O resident -O verify-required -f ~/.ssh/id_ed25519_sk_1

# Plugin YubiKey #2
ssh-keygen -t ed25519-sk -O resident -O verify-required -f ~/.ssh/id_ed25519_sk_2

# Plugin YubiKey #3
ssh-keygen -t ed25519-sk -O resident -O verify-required -f ~/.ssh/id_ed25519_sk_3
```
*Note: The `-O verify-required` flag ensures it asks for your PIN, keeping the security level high.*

#### 2. Register the Identities
You will have three unique public keys (e.g., `id_ed25519_sk_1.pub`, etc.). Add all three to your GitHub account or your server's `authorized_keys`. Now, any of your three physical keys will grant you access.

#### 3. Why this works from the Obsidian Menu
1.  **Standard SSH Agent:** When you click "Sync" in Obsidian, it calls the system `ssh`. 
2.  **No Redirection:** Because we cleaned up `environment.d`, `ssh` looks at the default system socket.
3.  **Hardware Handshake:** The system agent sees the `sk` key type and knows it must talk to the USB device. 
4.  **The Result:** You get a native system popup (from `pinentry` or a GTK prompt) asking for your PIN/touch. 

### Final Clean-up (The "Polishing")
Since you previously had `ssh-add` calls or agent logic in your `.bashrc` or `.profile`, you can now remove those. The beauty of the `ed25519-sk` keys is that most modern versions of OpenSSH/Debian will automatically "see" the key if it's in your `~/.ssh` folder when you attempt a connection.

If you want to be 100% sure the agent has them loaded when you log in, you can put this in your `~/.bashrc`:
```bash
ssh-add ~/.ssh/id_ed25519_sk_1 ~/.ssh/id_ed25519_sk_2 ~/.ssh/id_ed25519_sk_3 2>/dev/null
```

**How does it feel to have a snappy shutdown and a clean process list again?**

## Host configuration

That is the classic "SSH agent trial-and-error" problem. By default, SSH tries every key it finds in your agent or `~/.ssh` folder one by one until a server accepts one or you hit a "Too many authentication failures" error.

Since you want the hardware key to be your primary "Identity" and the local `id_rsa` to be the "Machine" backup, you need a way to tell SSH: "Try the YubiKey first."

### 1. The SSH Config (The "Professional" Way)

The best way to manage this is via your `~/.ssh/config` file. This is global, so it works for Obsidian, Git, and terminal SSH calls simultaneously.

Create or edit `~/.ssh/config` and add a block like this at the **very top**:

Plaintext

```
Host *
    # Try the YubiKey keys first
    IdentityFile ~/.ssh/id_ed25519_sk_1
    IdentityFile ~/.ssh/id_ed25519_sk_2
    IdentityFile ~/.ssh/id_ed25519_sk_3
    # Fall back to the machine identity last
    IdentityFile ~/.ssh/id_rsa
    
    # Optional: Only use keys specified above
    IdentitiesOnly yes
```

### 2. How SSH handles this logic

When Obsidian (or any SSH process) starts a connection:

1. **Priority:** It reads your config from top to bottom. It sees `sk_1` first.
    
2. **Hardware Check:** It asks the agent for `sk_1`. The agent realizes this is a YubiKey key.
    
3. **The Prompt:** If the YubiKey is plugged in, it flashes. If not, the agent moves to the next `IdentityFile` in the list.
    
4. **The Fallback:** If none of the YubiKeys are present, it finally tries `id_rsa`.
    

### 3. The "Identity vs. Machine" Logic

By using `IdentitiesOnly yes`, you prevent the agent from offering every random key it might have picked up. You are explicitly defining the "Identity Hierarchy":

- **Primary:** Person (YubiKey 1)
    
- **Secondary:** Person (YubiKey 2/3)
    
- **Tertiary:** Machine (`id_rsa`)
    

### 4. A Note on "Resident" Keys

Since you generated these as **Resident Keys**, you have a superpower: you don't even need the files in `~/.ssh/` on a new machine. You can run `ssh-add -K`, and the agent will "pull" the identities directly from the YubiKey into its memory.

However, for your daily-driver XPS, keeping the `IdentityFile` lines in your config is the most stable way to ensure Obsidian always knows exactly which "identity" to reach for first.

**Does that hierarchy make sense for your workflow, or do you have specific servers where you'd _only_ want to use the machine key?**

## Annoying touches

That is the "security tax" being levied by the default behavior of the SSH agent. Since Obsidian (and Git) can initiate multiple SSH connections in a burst—one to check the remote, one to push metadata, one to push the actual blob—you are being hit with a prompt for every single "handshake."

Because we are now in a **stock configuration**, we can fix this using the two standard tools designed for exactly this annoyance: **Agent Caching** and **Connection Multiplexing**.

### 1. The "PIN Cache" (ssh-add)

The PIN (Passphrase) check happens because the agent isn't "holding" the decrypted handle for the hardware key. You can tell the agent to verify the PIN once and remember it for the rest of your session.

Run this in your terminal:

Bash

```
ssh-add ~/.ssh/id_ed25519_sk_1
```

- **Result:** You enter the PIN once. For the rest of the day, Obsidian will **not** ask for the PIN. It will only flash for the "User Presence" touch.
    

---

### 2. The "Touch Cache" (ControlMaster)

The reason you are touching the key for _every file_ is that each one is opening a brand-new SSH connection. We can tell SSH to "reuse" the first authenticated pipe for all subsequent requests.

Open `~/.ssh/config` and add these three lines to your `Host *` block:

Plaintext

```
Host *
    # ... your existing IdentityFile lines ...
    
    # Connection Multiplexing
    ControlMaster auto
    ControlPath ~/.ssh/control-%C
    ControlPersist 10m
```

**How this works:**

- **ControlMaster auto:** The first time Obsidian syncs, it opens a "Master" connection.
    
- **The Touch:** You touch the YubiKey once to open that Master pipe.
    
- **The Reuse:** Every subsequent file "hitchhikes" on that same open pipe.
    
- **ControlPersist 10m:** The pipe stays open for 10 minutes after the last activity, so if you save another note 2 minutes later, it’s instantaneous—**no PIN, no touch.**
    

---

### 3. The "Identity" Balance

If you find that even the "Touch" is too much for a Git-based digital notebook like Formics/Obsidian, you have two choices:

1. **Lower the Hardware Bar:** Re-generate the key without `-O verify-required`. This removes the PIN requirement but keeps the Touch.
    
2. **The "Machine" Identity:** Use your local `id_rsa` for Obsidian specifically. Since `id_rsa` is a file on your encrypted disk, you can add it to the agent once (`ssh-add ~/.ssh/id_rsa`), and it will never ask for a touch.
    

**To use the Machine key just for Obsidian's host (assuming it's GitHub):**

Plaintext

```
Host github.com
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
```

**Does adding the `ControlMaster` lines to your config feel like the right middle ground to stop the "per-file" tapping?**

## References

1. [Consolidated Guide to Using Yubikeys with Linux](https://marcusb.org/posts/2024/03/consolidated-guide-to-using-yubikeys-with-linux)