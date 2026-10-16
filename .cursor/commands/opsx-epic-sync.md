---
name: "/opsx-epic-sync"
id: "opsx-epic-sync"
category: "Workflow"
description: "Update OpenSpec epic story checkboxes from archived child changes"
---

Use the `openspec-epic-management` skill in SYNC mode.

**Store selection:** If the user names a store or the epic lives in one, run
`openspec store list --json` and pass `--store <id>` on epic CLI calls
(`status`, `list`, `show`, `validate`, `archive`, `doctor`, `context`,
`schemas`, `view`). Query each child in that story's **Store** when set;
otherwise use the epic's selected root. Do not run `/opsx-sync`
(delta-spec merge) against the epic.

**Input**: The optional argument is the epic change name. Infer it only when
conversation context is unambiguous or exactly one active epic-driven change
exists in the selected root.

Check a Story Index box only when that story's child change is archived.
Uncheck boxes for children that are still planned, active, or implemented but
unarchived. Do not edit product code or other story fields.
