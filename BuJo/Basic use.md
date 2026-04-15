
Properties (formerly known as YAML front-matter) are the secret sauce that turns Obsidian from a simple text editor into a powerful database. In a Bullet Journal, Properties act as the "metadata" for your day—data that stays hidden but remains searchable and trackable.

Here is how to optimize them for your BuJo.

---

## 1. Essential BuJo Properties

When you create your **Daily Note template**, add these properties at the very top. This allows you to track trends over time without cluttering your actual journal entries.

|**Property Name**|**Type**|**Purpose**|
|---|---|---|
|**date**|Date|Sorts your notes chronologically in queries.|
|**mood**|Text/List|Track your emotional state (e.g., "Energetic," "Reflective").|
|**sleep**|Number|Log hours of sleep for health tracking.|
|**tags**|Multiselect|Label the day (e.g., `#work`, `#travel`, `#rest`).|
|**completed**|Checkbox|A quick "Done" toggle for your daily review.|

---

## 2. Why Use Properties Instead of Just Text?

While you _could_ just write "Mood: Happy" in the body of your note, using Properties allows for **automated reporting**:

- **Heatmaps:** You can use the _Heatmap Calendar_ plugin to look at your "mood" property and see a color-coded map of your year.
    
- **Dataview Summaries:** You can create a "Weekly Review" note that automatically pulls the `sleep` or `mood` data from the last seven days into a table:
    
    - _Example:_ `TABLE sleep, mood FROM "Daily Notes" WHERE date > date(today) - dur(7 days)`
        
- **Filtering:** You can instantly find every day you tagged as `#high-stress` to see if there’s a pattern with your workload.
    

---

## 3. Best Practices for BuJo Properties

To keep your journaling friction-free, follow these rules:

### Use "List" types for Tags

Instead of one long string, set your "tags" property to a **List**. This lets you click on individual tags to find related "Collections" or other daily logs.

### Automate with Templater

Don't type these manually every day. In your template, use a dynamic command so the date fills itself in:

YAML

```
---
date: <% tp.date.now("YYYY-MM-DD") %>
mood: 
sleep: 
tags: [daily-log]
---
```

### Keep it Lean

The "Digital BuJo Trap" is adding 20 properties (water intake, steps, weather, focus score). **If you don't plan to graph it or search for it, don't track it.** Start with 3 properties and only add more if you find yourself actually using the data.

---

## 4. Visualizing Properties

Obsidians's **Properties UI** (introduced in late 2023 and refined in 2026) makes this look like a clean form at the top of your note. You don't have to look at the raw code; you just click and type into the boxes.

If the Properties section feels too "techy" and ruins the vibe of your journal, you can go to **Settings > Editor > Properties in document** and set it to **Source** (to hide the UI) or **Hidden** (to keep it in the background).

Do you see yourself using these properties more for "health and habit tracking," or are you looking to use them to organize your project notes?