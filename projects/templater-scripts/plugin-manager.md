---
layout: templater
title: Plugin Manager
description: Registers three commands directly into Obsidian's command palette — enable a disabled plugin, disable an enabled plugin, and open any settings tab. All three use a fuzzy suggester so you never have to scroll through the settings panel again.
category: Utility
tags: [plugins, commands, settings, brat]
date: 2024-01-01
---

Run this once on startup. It registers three commands into the palette permanently for that session. After that, trigger them via `Cmd/Ctrl + P` like any other command — no need to re-run the template.

```javascript
<%* 
const { plugins, setting, commands, workspace } = app;

// ── ENABLE PLUGIN ──────────────────────────────────────────
if (!commands.commands["brat-enable-plugin"]) {
  commands.addCommand({
    id: "brat-enable-plugin",
    name: "🟢 Enable a Disabled Plugin",
    callback: async () => {
      const manifests = Object.values(plugins.manifests);
      const enabled = Object.values(plugins.plugins).map(p => p.manifest);
      const disabledList = manifests
        .filter(manifest => !enabled.some(p => p.id === manifest.id))
        .map(m => ({ display: `${m.name} (${m.id})`, info: m.id }));

      if (disabledList.length === 0) {
        new Notice("✅ All plugins are already enabled!");
        return;
      }

      const selected = await tp.system.suggester(
        disabledList.map(p => p.display),
        disabledList
      );
      if (selected) {
        await plugins.enablePlugin(selected.info);
        new Notice(`✅ Enabled ${selected.display}`);
      }
    }
  });
}

// ── DISABLE PLUGIN ─────────────────────────────────────────
if (!commands.commands["brat-disable-plugin"]) {
  commands.addCommand({
    id: "brat-disable-plugin",
    name: "🔴 Disable an Enabled Plugin",
    callback: async () => {
      const manifests = Object.values(plugins.manifests);
      const enabled = Object.values(plugins.plugins).map(p => p.manifest);
      const enabledList = manifests
        .filter(m => enabled.some(p => p.id === m.id))
        .map(m => ({ display: `${m.name} (${m.id})`, info: m.id }));

      if (enabledList.length === 0) {
        new Notice("⚠️ No enabled plugins to disable.");
        return;
      }

      const selected = await tp.system.suggester(
        enabledList.map(p => p.display),
        enabledList
      );
      if (selected) {
        await plugins.disablePluginAndSave(selected.info);
        new Notice(`🛑 Disabled ${selected.display}`);
      }
    }
  });
}

// ── OPEN SETTINGS TAB ──────────────────────────────────────
if (!commands.commands["brat-open-settings"]) {
  commands.addCommand({
    id: "brat-open-settings",
    name: "⚙️ Open Any Settings Tab",
    callback: async () => {
      const pluginTabs = Object.values(setting.pluginTabs)
        .map(t => ({ display: `Plugin: ${t.name}`, info: t.id }));
      const coreTabs = Object.values(setting.settingTabs)
        .map(t => ({ display: `Core: ${t.name}`, info: t.id }));

      const settingsTabs = [...coreTabs, ...pluginTabs];
      const selected = await tp.system.suggester(
        settingsTabs.map(t => t.display),
        settingsTabs
      );
      if (selected) {
        setting.open();
        setting.openTabById(selected.info);
      }
    }
  });
}
%>
```

## The Three Commands

**🟢 Enable a Disabled Plugin** — pulls every installed-but-disabled plugin into a fuzzy suggester. Pick one, it enables immediately.

**🔴 Disable an Enabled Plugin** — same flow in reverse. Lists everything currently running, pick what to turn off. Uses `disablePluginAndSave` so the state persists across restarts.

**⚙️ Open Any Settings Tab** — lists both core tabs and plugin tabs in one suggester. Faster than navigating the settings sidebar, especially with 20+ plugins installed.

## How the Guard Works

Each command registration is wrapped in `if (!commands.commands["id"])` — so running the template twice in a session won't duplicate the commands in the palette.

## Notes

- Commands are session-scoped. They disappear when Obsidian closes — re-run the template on next startup or add it to your startup template.
- Requires Templater plugin. The `tp.system.suggester` call is a Templater API method, not native Obsidian.