Since Obsidian notes are just a collection of Markdown files in a local folder, you have a few ways to keep them in sync depending on how much you want to spend and how much you like to tinker.

## 1. The Official Way: Obsidian Sync

This is the most seamless method. It’s a paid service ($4–$10/month) that provides end-to-end encrypted syncing managed directly within the app.

- **Pros:** Easiest setup, handles version history, and works perfectly on mobile (iOS/Android) where background syncing is often restricted.
    
- **How to do it:** Enable the **Sync** core plugin in Obsidian settings, log in, and create a "Remote Vault."
    

---

## 2. The Free (Tinkerer’s) Way: Syncthing

Since you're comfortable with terminal-based workflows and Linux environments, **Syncthing** is likely your best bet for a free, peer-to-peer solution.

- **Pros:** Privacy-focused (no cloud), fast, and highly configurable.
    
- **Setup:**
    
    1. Install Syncthing on your computer and the **Syncthing-Fork** app on Android (or Mobius/SyncTrain on iOS).
        
    2. Add your vault folder in Syncthing on your PC.
        
    3. Share that folder with your phone’s device ID.
        
    4. On your phone, point the Obsidian app to the folder Syncthing is managing.
        
- **Note:** Keep an eye on file conflicts if you edit the same note on both devices simultaneously.
    

---

## 3. The "Developer" Way: Obsidian Git

Since you already use Git for projects, you can treat your vault like a repository.

- **Setup:** Use the **Obsidian Git** community plugin.
    
- **Workflow:** It can be configured to auto-commit and pull/push on a timer or every time you open/close the app.
    
- **Mobile:** On Android, you can use an app like **MGit** or the plugin’s built-in (though sometimes experimental) support. On iOS, most people use **Working Copy** to bridge the Git repo to the file system.
    

---

## 4. Cloud Storage Bridge (Platform Dependent)

If you already use a cloud provider, you can "bridge" the connection:

- **iOS + Mac/Windows:** Use **iCloud**. Create your vault directly inside the iCloud/Obsidian folder.
    
- **Android + Windows/Linux:** Use an app like **OneSync** (for OneDrive) or **Autosync** (for Google Drive/Dropbox). These apps perform a "two-way sync" between the cloud and a local folder on your phone that Obsidian can see.
    

### Summary Comparison

|**Method**|**Cost**|**Effort**|**Best For**|
|---|---|---|---|
|**Obsidian Sync**|Paid|Low|Reliability & simplicity|
|**Syncthing**|Free|Medium|Privacy & local-only sync|
|**Git**|Free|High|Version control enthusiasts|
|**Cloud Bridge**|Free*|Medium|Using existing storage|

Which operating systems are you using on your computer and phone?