---
name: "/opsx-epic-replan"
id: "opsx-epic-replan"
category: "Workflow"
description: "Revise future OpenSpec epic stages and planned stories from new learning"
---

Use the `openspec-epic-management` skill in REPLAN mode.

**Store selection:** If the user names a store (a store is a standalone OpenSpec
repo registered on this machine) or the epic lives in one, run
`openspec store list --json` to discover registered store ids, then pass
`--store <id>` on the commands that read or write specs and changes
(`new change`, `status`, `instructions`, `list`, `show`, `validate`, `archive`,
`doctor`, `context`, `schemas`, `view`). Once selected, treat `--store <id>` as
sticky for the epic. **Store** on a story may change only while that story is
still planned. Without a store, commands act on the nearest local `openspec/`
root.

**Input**: The argument should identify the epic and explain the new information
or desired roadmap change. Show proposed revisions first. Preserve completed
history and active child story scope, and require confirmation before writing.
Do not edit Story Index checkboxes here; use `/opsx-epic-sync` after archive.
