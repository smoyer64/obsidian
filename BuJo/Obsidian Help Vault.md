To view the Obsidian documentation offline, there isn't a single toggle or a "built-in" offline reader within the app itself, but because the official documentation is hosted as a public repository of Markdown files, you can easily turn it into an offline vault.

### Option 1: Download the Official Help Vault (Recommended)

The most reliable way to access the documentation offline is to download the source files directly from Obsidian's GitHub. Since the documentation is literally an Obsidian vault, it will look and behave exactly like the official site once you open it.

1. **Download:** Visit the [obsidian-help GitHub repository](https://github.com/obsidianmd/obsidian-help).
    
2. **Get the Files:** Click the green **Code** button and select **Download ZIP**.
    
3. **Extract:** Unzip the folder on your computer.
    
4. **Open in Obsidian:**
    
    - Open the Obsidian Vault Switcher (the vault icon in the bottom left).
        
    - Click **Open folder as vault**.
        
    - Select the `en` folder (or your preferred language) inside the extracted directory.
        
5. **Offline Access:** You now have a complete, local, searchable copy of the documentation that works without an internet connection.
    

### Option 2: Use the "Obsidian Web Clipper"

If you only need specific sections of the documentation (or want to keep them updated in your main vault), you can use the official **Obsidian Web Clipper** (available for Chrome and Firefox).

- While online, navigate to the help pages you use most.
    
- Clip them to your vault as Markdown.
    
- These files are then stored locally and are accessible even when you are offline.
    

### Option 3: Community Plugins

While there isn't a specific plugin dedicated solely to "Offline Help," there are tools that help manage documentation:

- **DevDocs Plugin:** Some users use the DevDocs community plugin to pull in various documentation sets, though it is primarily geared toward programming languages.
    
- **GitHub Link / Git Plugin:** If you use the **Obsidian Git** plugin, you can clone the `obsidian-help` repository mentioned above into a folder within your vault. This allows you to "pull" updates whenever you have internet, keeping your offline documentation current.
    

**Pro Tip:** If you are a developer looking for the **API Documentation** (docs.obsidian.md) specifically, that is located in a separate repository called [obsidian-docs](https://www.google.com/search?q=https://github.com/obsidianmd/obsidian-docs) and can be downloaded/viewed offline using the same method as Option 1.