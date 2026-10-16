---
name: "/opsx-epic-propose"
id: "opsx-epic-propose"
category: "Workflow"
description: "Propose a new OpenSpec epic - create it and generate vision, roadmap, and stories in one step"
---

Use the `openspec-epic-management` skill in PROPOSE mode.

**Planning boundary**: This workflow creates epic planning artifacts only
(`vision.md`, `roadmap.md`, `stories.md`). Do not edit product code. Do not
propose, apply, or implement a child story in the same turn.

**Store selection:** If the user names a store (a store is a standalone OpenSpec
repo registered on this machine) or the epic lives in one, run
`openspec store list --json` to discover registered store ids, then pass
`--store <id>` on the commands that read or write specs and changes
(`new change`, `status`, `instructions`, `list`, `show`, `validate`, `archive`,
`doctor`, `context`, `schemas`, `view`). Once selected, treat `--store <id>` as
sticky for the epic. Every unscoped example of those commands is shorthand:
append the flag. Without a store, commands act on the nearest local `openspec/`
root.

When stories should land in other registered stores, record each story's
**Store** field during `stories.md` creation. Child uniqueness is
`(store, change name)`.

**Input**: The argument is the epic change name (kebab-case), OR a description
of the initiative. Always create the change with `--schema epic-driven`.

After all three artifacts exist, stop and prompt `/opsx-epic-status` then
`/opsx-epic-next`. Incremental planning remains `/opsx-new --schema epic-driven`
plus `/opsx-continue`.
