
This guide provides a step-by-step configuration for a 2026-standard Bullet Journal vault. By following this, you’ll move from "manual notes" to a system that handles your task migrations and data tracking automatically.

---

## 1. Vault Structure (The "Engine Room")

To keep things clean, use a "Pluralized & Flat" folder structure. This prevents the "nested folder fatigue" common in digital journals.

- **`00_Admin/`**: Put your `Templates` folder here.
- **`Daily/`**: All your daily logs go here.
- **`Archive/`**: Move old months here at the end of the year.
- **`Collections/`**: For project-specific notes (e.g., "Kitchen Reno," "Reading List").
    

---

## 2. Configuring the Core Plugins

Go to **Settings > Community Plugins** to install these. Here is exactly how to set them up:

### Plugin 1: Periodic Notes

- **Purpose**: Creates your Daily Log.    
- **Config**:
    - **Format**: `YYYY-MM-DD` (Standard for sorting).
    - **Folder**: Set to `Daily/`.
    - **Template**: Set to `00_Admin/Templates/Daily Template`.

### Plugin 2: Rollover Daily Todos (The "Migration" Plugin)

- **Purpose**: Automatically moves yesterday's unfinished tasks to today.
- **Config**:
    - **Automatic Rollover**: Toggle **ON**.
    - **Template Heading**: Set this to `## 📝 Rapid Log`. (The plugin will find this heading in your today's note and drop yesterday's tasks right under it).
    - **Remove empty todos**: Toggle **ON**.

### Plugin 3: Calendar

- **Purpose**: Visual navigation.
- **Config**:
    - Open the right sidebar and drag the Calendar to the top.
    - Enable **"Show week numbers"** to help with weekly reviews.

### Plugin 4: Tasks

- **Purpose**: Global search for every `- [ ]` in your vault.
- **Config**:
    - In settings, set the **Global task filter** to something like `#task` if you want to be specific, or leave it blank to track every single checkbox in your journal.

---

## 3. The "Dated Scheduling" Workflow

While **Rollover Daily Todos** handles "Today vs. Yesterday," the **Tasks** plugin handles the "Future."

If you have a task for next Tuesday, type it like this in today's log:

`- [ ] Appointment with Dentist 📅 2026-04-21`

Even though you wrote it on the 14th, the task will not show up in your "Today's Focus" until the 21st if you use a Dataview query.

---

## 4. Automation: The Master Dashboard

To see everything in one place, create a note called `00_Dashboard`. Paste this code inside to create an automatic "Master To-Do List":

Markdown

````
# ⚡ Master To-Do List

## 🔴 Overdue
```tasks
not done
due before today
````

## 🟢 Today's Focus

Code snippet

```
not done
due today
```

## 📅 Scheduled This Week

Code snippet

```
not done
due after today
due before in 1 week
```

```

---

## 5. Summary of Your New Daily Workflow
1.  **Open Obsidian**: It will automatically create today's note (via *Periodic Notes*).
2.  **Auto-Migrate**: *Rollover Daily Todos* instantly copies any unfinished tasks from yesterday into your new log.
3.  **Check Dashboard**: Look at your `00_Dashboard` to see any tasks you "Scheduled" weeks ago that are now due today.
4.  **Log**: Throughout the day, use the **Rapid Log** section for new tasks (`- [ ]`) and notes (`- /`).

Would you like me to generate a downloadable `.md` file for this specific Dashboard setup as well?
```