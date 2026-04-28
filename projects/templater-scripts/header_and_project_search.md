---
layout: templater
title: Heading and Project Search
description: Registers two navigation commands into the palette — jump to any note tagged as a project, and jump to any H1 heading across the entire vault. Both use fuzzy search.
category: Navigation
tags: [navigation, search, headings, projects, commands]
date: 2024-01-01
---

Run once to register both commands. After that they live in the palette for the session — use `Cmd/Ctrl + P` to trigger either one. No need to open a file first.

```javascript
<%* 
const { vault, metadataCache, workspace, commands } = app;

// ── JUMP TO PROJECT ────────────────────────────────────────
if (!commands.commands["templater-jump-to-project"]) {
  commands.addCommand({
    id: "templater-jump-to-project",
    name: "🗂️ Jump to Project (is-project)",
    callback: async () => {
      const files = vault.getMarkdownFiles();
      const projects = [];

      for (const file of files) {
        const cache = metadataCache.getFileCache(file);
        const fm = cache?.frontmatter;

        if (fm && Object.prototype.hasOwnProperty.call(fm, "is-project")) {
          projects.push({
            display: file.basename,
            path: file.path
          });
        }
      }

      if (!projects.length) {
        new Notice("No project pages found.");
        return;
      }

      const selected = await tp.system.suggester(
        projects.map(p => p.display),
        projects
      );

      if (!selected) return;
      await workspace.openLinkText(selected.path, "/", false);
    }
  });
}

// ── JUMP TO ANY HEADING ────────────────────────────────────
if (!commands.commands["templater-heading-jump"]) {
  commands.addCommand({
    id: "templater-heading-jump",
    name: "🪄 Jump to Any Heading (first H1 only)",
    callback: async () => {
      const files = vault.getMarkdownFiles();
      const headingItems = [];

      for (const file of files) {
        const headings = metadataCache.getFileCache(file)?.headings ?? [];
        const firstH1 = headings.find(h => h.level === 1);

        if (firstH1 && !firstH1.heading.includes("🌞")) {
          headingItems.push({
            display: `${firstH1.heading}\n`,
            filePath: file.path,
            heading: firstH1.heading.trim()
          });
        }
      }

      if (!headingItems.length) {
        new Notice("⚠️ No H1 headings found.");
        return;
      }

      const selected = await tp.system.suggester(
        headingItems.map(h => h.display),
        headingItems
      );

      if (!selected) return;

      const target = `${selected.filePath}#${selected.heading}`;
      await workspace.openLinkText(target, "/", false);
    }
  });
}

return "";
%>
```

## The Two Commands

**🗂️ Jump to Project** — scans every markdown file in the vault for a frontmatter key called `is-project`. The value doesn't matter — just having the key is enough. All matching files appear in the suggester by filename.

**🪄 Jump to Any Heading** — pulls the first H1 from every file in the vault into one searchable list. Skips any heading containing 🌞 (useful for excluding daily notes or dashboard headers). Navigates directly to the heading, not just the file.

## Setting Up Project Detection

Add `is-project:` to the frontmatter of any note you want to appear in the Jump to Project suggester:

```yaml
---
is-project:
title: My Project
---
```

The value can be anything — empty, `true`, a date. The script only checks for key existence.

## Filtering Headings

The `🌞` filter on the heading jump is hardcoded. To change what gets excluded, replace `"🌞"` with any string that appears in headings you want to skip — for example `"Daily"` or `"Inbox"`.

## Notes

- Both commands are session-scoped — re-run on startup or chain into your startup template.
- Requires Templater. Uses `tp.system.suggester` for the fuzzy picker.
- `return ""` at the end prevents Templater from inserting anything into the note body.