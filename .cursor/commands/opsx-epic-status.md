---
name: "/opsx-epic-status"
id: "opsx-epic-status"
category: "Workflow"
description: "Show derived progress and available work for an OpenSpec epic"
---

Use the `openspec-epic-management` skill in STATUS mode.

**Store selection:** If the user names a store (a store is a standalone OpenSpec
repo registered on this machine) or the epic lives in one, run
`openspec store list --json` to discover registered store ids, then pass
`--store <id>` on the commands that read or write specs and changes
(`new change`, `status`, `instructions`, `list`, `show`, `validate`, `archive`,
`doctor`, `context`, `schemas`, `view`). Once selected, treat `--store <id>` as
sticky for the epic. Query each child in that story's **Store** when set.
Without a store, commands act on the nearest local `openspec/` root.

**Input**: The optional argument is the epic change name. Infer it only when
conversation context is unambiguous or exactly one active epic-driven change
exists. Do not write files. Treat Story Index checkboxes as a projection:
OpenSpec archive state is source of truth; report checkbox drift and recommend
`/opsx-epic-sync`.
