When configuring a new computer, use this document to "finish" the setup (prior to running the `workstation` Ansible playbook.)
## GPG and SSH configuration

To implement a unified **GPG-as-SSH-Agent** workflow with your YubiKey and `pass`, follow these instructions. This setup ensures you enter your YubiKey PIN **once** at login, which then automatically primes the SSH key for the Obsidian AppImage.

### 1. Configure the GPG Agent

This tells GPG to act as an SSH agent and sets the cache timeout so you aren't re-entering your PIN constantly.

**File:** `~/.gnupg/gpg-agent.conf`

Plaintext

```
# Enable SSH support within GPG
enable-ssh-support

# Set PIN cache to 8 hours
default-cache-ttl 28800
max-cache-ttl 28800
default-cache-ttl-ssh 28800
max-cache-ttl-ssh 28800

# Use a GUI pinentry for XFCE
pinentry-program /usr/bin/pinentry-gtk-2
```

---

### 2. Set the Environment Globally

This ensures all GUI applications (like the Obsidian AppImage) know to use the GPG socket instead of the standard SSH agent. Since you are on Debian, `.xsessionrc` is the most reliable place for this.

**File:** `~/.xsessionrc`

Bash

```
# Point SSH to the GPG-managed socket
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)

# Ensure the agent is launched
gpgconf --launch gpg-agent

# Tell GPG which TTY to use for pinentry if it falls back to terminal
export GPG_TTY=$(tty)
```

---

### 3. Create the Autostart Entry

This "primes" the agent at login. It pulls the passphrase from your YubiKey-encrypted `pass` store and loads the SSH key into the GPG agent's memory.

**File:** `~/.config/autostart/ssh-unlock.desktop`

_(Manage this in `chezmoi` as `dot_config/autostart/ssh-unlock.desktop`)_

Ini, TOML

```
[Desktop Entry]
Type=Application
Name=SSH Key Unlock
Comment=Prime SSH key from pass via YubiKey at login
# The 'DISPLAY=' ensures ssh-add doesn't try to open its own prompt
Exec=bash -c "pass show keys/ssh-passphrase | DISPLAY= ssh-add ~/.ssh/id_ed25519"
OnlyShowIn=XFCE;
StartupNotify=false
Terminal=false
```

---

### 4. Implementation Checklist

To finalize the switch and avoid conflicts:

1. **Disable the old Agent:** In XFCE **Settings > Session and Startup > Application Autostart**, uncheck "SSH Agent (Gnome Keyring)" if it is present.
    
2. **Restart GPG:** Run `gpgconf --kill gpg-agent` followed by `gpgconf --launch gpg-agent` to apply the new config.
    
3. **Clean Up:** Remove any `eval $(ssh-agent)` calls from your `.bashrc` or `.zshrc` to prevent them from overwriting the `SSH_AUTH_SOCK` you set in `.xsessionrc`.
    

### Why this works for Obsidian

When you log in, XFCE runs the `.desktop` file. You will see a GUI PIN prompt. Once you enter it:

1. GPG uses the YubiKey to decrypt the `pass` entry.
    
2. The passphrase is fed into the GPG-SSH agent.
    
3. When you launch the **Obsidian AppImage**, it inherits the `SSH_AUTH_SOCK` from your session.
    
4. The Obsidian Git plugin asks the socket for a key; GPG provides it immediately without a second prompt.
    

Does this consolidated plan cover everything you need for your `chezmoi` templates?
### How the Persistence Works

When you log in, the XFCE session manager reads `.xsessionrc` and sets the `SSH_AUTH_SOCK`. Because every application you launch (Obsidian, your Terminal, etc.) is a "child" of that initial session, they **inherit** that variable automatically.

- **GUI Applications:** Obsidian (the AppImage) inherits the variable directly from the X session. It will work perfectly without any further configuration.
    
- **Terminals:** When you open a terminal, it also inherits the variable from the desktop. However, there is a catch: if your `.bashrc` or `.zshrc` contains code that manually starts a _new_ `ssh-agent`, it will **overwrite** the one inherited from the session, breaking the link to GPG.
    

---

### Ensuring Persistence in your `chezmoi` Dotfiles

To make sure your terminal and GUI apps stay in sync, you should structure your shell configs to **respect** the existing socket if it's already set.

#### 1. Update your `.bashrc` (or `.zshrc`)

Add this logic to your shell configuration. It checks if a socket is already defined (by `.xsessionrc`) before trying to do anything else.

Bash

```
# Only set the GPG-SSH socket if it isn't already inherited from the session
if [ -z "$SSH_AUTH_SOCK" ]; then
    export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
fi

# Ensure GPG knows which terminal to use for pin-entry if you run pass in CLI
export GPG_TTY=$(tty)

# Refresh the GPG TTY mapping so GUI and CLI don't conflict
gpg-connect-agent updatestartuptty /bye > /dev/null 2>&1
```

#### 2. The `.profile` vs `.bash_profile`

On Debian, `.profile` is usually the place for variables meant for the whole login session. However, since you are using XFCE, `.xsessionrc` is the "source of truth" for the GUI.

**Pro-Tip for `chezmoi`:** If you want maximum portability across servers (where you might not have XFCE), you can put the `SSH_AUTH_SOCK` export in a shared snippet that both `.xsessionrc` and `.bashrc` call.

---

### Summary of the Variable Flow

1. **Login:** `.xsessionrc` sets `SSH_AUTH_SOCK` to the GPG socket.
    
2. **Inheritance:** XFCE launches. Every app launched (including Obsidian) now "owns" that variable.
    
3. **Terminal:** You open `xfce4-terminal`. It inherits the variable.
    
4. **Verification:** Running `echo $SSH_AUTH_SOCK` in any terminal should point to something like `/run/user/1000/gnupg/S.gpg-agent.ssh`.
    

### Important Conflict Check

If your Obsidian sync ever fails, check your `~/.bashrc` for these lines and **remove or comment them out**:

- `eval $(ssh-agent -s)`
    
- `exec ssh-agent bash`
    

These commands create a "private" agent that only the terminal knows about, which effectively "hides" the YubiKey/GPG agent from that specific terminal window.

Does your current `.bashrc` have any existing `ssh-agent` or `keychain` logic that we need to clean up?