---
name: "/opsx-epic-next"
id: "opsx-epic-next"
category: "Workflow"
description: "Select or start the next dependency-ready story from an OpenSpec epic"
---

Use the `openspec-epic-management` skill in NEXT mode.

**Store selection:** If the user names a store (a store is a standalone OpenSpec
repo registered on this machine) or the epic lives in one, run
`openspec store list --json` to discover registered store ids, then pass
`--store <id>` on the commands that read or write specs and changes
(`new change`, `status`, `instructions`, `list`, `show`, `validate`, `archive`,
`doctor`, `context`, `schemas`, `view`). Once selected, treat `--store <id>` as
sticky for the epic. When starting a story, use that story's **Store** as the
sticky store for `openspec-propose` if set. Without a store, commands act on
the nearest local `openspec/` root.

**Input**: Accept an optional epic name, story ID, and the word `start`.
Without `start`, recommend work and print the standard `/opsx-propose` handoff
(including `--store` when the story has one) without writing files. With
`start`, plan exactly one selected child story using the installed
`openspec-propose` skill, then stop before implementation.
