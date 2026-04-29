---
layout: templater
title: Next-Action + Closer
description: Registers a close command that prompts for a next action before shutting the tab — so every note you leave has a task telling you exactly where to pick up.
category: Utility
tags: [tasks, next-action, close, workflow, commands]
date: 2025-04-28
---

Run once on startup. After that, trigger `❌add next-action & close` from `Cmd/Ctrl + P` whenever you're done with a note — it prompts, writes the task, and closes the tab in one move.

```javascript
<%* const { vault, workspace, commands } = app;
if (!commands.commands["❌add next-action & close"]) { commands.addCommand({ id: "❌add next-action & close", name: "❌add next-action & close", callback: async () => { const file = workspace.getActiveFile(); const orConditions = [ file.path.includes("3. resource/programs"), !file, !file.path.endsWith("md"), file.path.includes("blank page"), file.path.includes("calender/"), ];

    if (orConditions.some(Boolean)) {
      await app.commands.executeCommandById("workspace:close");
      return;
    }

    const nextAction = await tp.system.prompt("Enter next action");
    if (!nextAction || !nextAction.trim()) {
      await app.commands.executeCommandById("workspace:close");
      return;
    }

    const time = tp.date.now("YYYY-MM-DD");

    const content = await vault.read(file);
    const match = content.match(/^---\s*[\s\S]*?^---\s*/m);

    let updated;
    if (match) {
      const pos = match[0].length;
      updated =
        content.slice(0, pos) +
        `\n- [ ] #next-action  ${nextAction} ➕ ${time} \n` +
        content.slice(pos);
    } else {
      updated = `- [ ] #next-action ${time} ${nextAction}\n${content}`;
    }

    await vault.modify(file, updated);
    await app.commands.executeCommandById("workspace:close");
  }
});

} new Notice("❌loaded closer"); return ""; %>
```

## The Command

**❌ add next-action & close** — triggers a prompt, writes one task into the active note, then closes the tab. Three outcomes:

- **Excluded path** → skips the prompt entirely, closes immediately
- **Empty prompt / cancelled** → no task written, closes immediately
- **Valid input** → inserts `- [ ] #next-action [text] ➕ [date]` right after the frontmatter block, then closes

The task lands after the `---` fence — not at the top of the body, not at the bottom. It stays out of the way of your prose but inside the file where Tasks can find it.

## The Task Queries

Paste both blocks into your dashboard or daily note. Together they cover everything tagged `#next-action`.

**Query 1 — Scheduled, Due Today, or Overdue**

Surfaces tasks with a date attached that are due on or before today. Groups by date descending so the most overdue items sit at the top.

```tasks
happens on or before today
group by function reverse task.happens.format("%%YYYY-MM-DD%%Do MMMM YYYY")
hide created date
tags include next-action
hide tags
show tree
not done
sort by happens reverse
```

**Query 2 — Undated Tasks**

Surfaces everything with no due or scheduled date — tasks you parked without a deadline. Groups by creation date so oldest open threads surface first.

```tasks
no due date
no scheduled date
tags include next-action
group by function reverse task.created.format("%%YYYY-MM-DD%%Do MMMM YYYY")
hide created date
hide tags
show tree
not done
```

## Excluded Paths

The script skips the prompt and just closes if the active file matches any of these conditions:

| Condition | Why |
|---|---|
| `file.path.includes("3. resource/programs")` | Reference material — no actions live here |
| `!file` | No active file open |
| `!file.path.endsWith("md")` | Not a markdown note |
| `file.path.includes("blank page")` | Scratch space — intentionally disposable |
| `file.path.includes("calender/")` | Calendar notes use a different close flow |

These fire via `orConditions.some(Boolean)` — one match is enough to skip straight to close.

## How to Customise

**Add an excluded path** — push a new condition into the `orConditions` array:
```javascript
file.path.includes("your/folder/here"),
```

**Change the task format** — edit the template string inside the `if (match)` branch. The current format writes:
```
- [ ] #next-action  [text] ➕ YYYY-MM-DD
```
Swap `➕` for a different emoji, add a due date with `📅`, or drop the creation date entirely.

**Change the date format** — `tp.date.now("YYYY-MM-DD")` uses moment.js tokens. Replace with `"DD MMM YYYY"` for `28 Apr 2025` style.

**Skip the prompt on more file types** — add `file.extension !== "md"` as an orCondition if you work with canvas or other non-markdown files.

## Notes

- Command is session-scoped — re-run on startup or chain into your startup template.
- Requires Templater. The `tp.system.prompt` call is a Templater API method, not native Obsidian.
- Requires the Tasks plugin for the query blocks to render.
- `return ""` at the end prevents Templater from inserting anything into the note body.
- The guard `if (!commands.commands["❌add next-action & close"])` prevents duplicate palette entries if the template runs twice in a session.