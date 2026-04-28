---
layout: templater
title: date
description: Fires a formatted date and time notice when the template runs. Good as a daily note opener or just to know what time it is without leaving Obsidian.
category: Date & Time
tags: [date, time, notice, greeting]
date: 2024-01-01
---

Paste this at the top of any template. When Templater runs it, a notice pops up in the corner with the current time, day name, date, and week number — then the script returns nothing, leaving the note clean.

```javascript
<%* 
new Notice(tp.date.now("[it's ]h:mma[ on a 👉]dddd[👈\n] Do MMMM Y[\n]Wo[ week of the year, Q]Q-Y")); return ""; 
%>
```

## How It Works

`tp.date.now()` takes a moment.js format string. The square brackets `[ ]` are literal text — everything outside them is a format token. So `[it's ]h:mma` outputs `it's 9:34am`, and `dddd` outputs the full day name.

`new Notice()` sends the formatted string to Obsidian's notification system. It disappears after a few seconds automatically.

`return ""` makes sure the script leaves no trace in the note body — without it, Templater would insert `undefined` where the block was.

## Format Tokens Used

- `h:mma` → 12-hour time with am/pm, e.g. `9:34am`
- `dddd` → full day name, e.g. `Wednesday`
- `Do` → day of month with ordinal, e.g. `3rd`
- `MMMM Y` → full month name + year, e.g. `January 2025`
- `Wo` → week number with ordinal, e.g. `1st`
- `QQ-Y` → quarter + year, e.g. `Q1-2025`

## Customising

Swap out any token to change the output. The `\n` inside the string is a line break — use it to control how the notice stacks. If you want a 24-hour clock, replace `h:mma` with `HH:mm`.