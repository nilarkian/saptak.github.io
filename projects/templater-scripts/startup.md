---
layout: templater
title: Startup Plugin Enabler
description: Runs on vault open and sequentially enables a defined list of plugins with a delay between each — prevents Obsidian from choking when loading multiple heavy plugins at once.
category: Utility
tags: [startup, plugins, automation]
date: 2024-01-01
---

Drop this into a note you run on vault open. It works through a list of plugin IDs one by one, waiting 5 seconds between each enable — giving Obsidian time to settle before loading the next one. Comment out any plugin you don't want enabled on that session.

```javascript
<%* 

async function main() {
    const plugins = app.plugins.plugins;

    // List of plugins to enable.
    // Comment out any line to skip that plugin on this run.
    const pluginsToEnable = [
        //"note-toolbar",
        //"tasknotes",
        //"HCI",
        //"pill-view",
        //"obsidian-tasks-plugin",
        "breadcrumbs",
        "note-mover-shortcut",
        //"obsidian-auto-link-title",
        "callout-integrator",
        "next-toc",
    ];

    // Enable plugins one by one with a 5-second delay between each
    for (const plugin of pluginsToEnable) {
        await new Promise(resolve => setTimeout(resolve, 5000));
        await app.plugins.enablePlugin(plugin);
    }
}

// Run the script
await main();
%>
```

## Why the Delay

Obsidian loads plugins synchronously by default. Enabling several at once — especially heavier ones — can cause race conditions, missed loads, or UI freezes. The `5000ms` delay gives each plugin time to fully initialise before the next one starts.

## How to Customise

**Change the delay** — swap `5000` for any millisecond value. 3000 works fine for lighter plugins, 7000+ for anything that touches the file system heavily.

**Add a plugin** — find its ID in `.obsidian/plugins/` (it's the folder name), add it as a new string in `pluginsToEnable`.

**Skip a plugin for a session** — comment out its line with `//`. It stays in the list for next time.

**Remove a plugin permanently** — delete its line entirely.

## Finding Plugin IDs

Open `.obsidian/plugins/` in your file explorer. Each subfolder name is the plugin's ID — that's the exact string to use in the array.