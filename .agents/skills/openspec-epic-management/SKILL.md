---
name: openspec-epic-management
description: Manage multi-change OpenSpec epics — propose vision/roadmap/stories together, derive progress, sync completion checkboxes, select dependency-ready stories, and replan future work. Use when the user mentions an OpenSpec epic, epic propose, epic sync, roadmap status, next story, available story, or epic replanning.
license: MIT
compatibility: Requires OpenSpec CLI and the epic-driven schema.
metadata:
  author: Ivan Ferić
  version: "1.0"
---

# OpenSpec Epic Management

Manage an epic-driven planning change while preserving normal spec-driven child
changes for implementation.

## Invariants

- The epic is an OpenSpec change whose schema is `epic-driven`.
- Every implementation story is a separate `spec-driven` OpenSpec change.
- Never edit product code while operating on the epic.
- Never run apply, spec-sync (`/opsx-sync` / `openspec-sync-specs`), or verify
  against the epic change.
- Omitting the schema's `apply` block is not a CLI lock. Even if generic CLI
  instructions say the epic is ready for implementation, stop: only child
  `spec-driven` changes may be implemented.
- `/opsx-epic-sync` is the only write that projects completion onto
  `stories.md` checkboxes. Do not check them by hand.
- A story is `done` only when its child change is archived in that story's
  OpenSpec root (the story's **Store**, or the epic's root when Store is omitted).
- Checkboxes are a display projection. Derived OpenSpec state wins when they
  disagree; report drift and recommend `/opsx-epic-sync`.
- Completed stories and stages are historical facts. Do not rewrite them.
- Active story scope changes belong in that child change via the normal
  `openspec-update-change` workflow.

## Store selection

If the user names a store (a store is a standalone OpenSpec repo registered on
this machine) or the epic lives in one, run `openspec store list --json` to
discover registered store ids, then pass `--store <id>` on the commands that
read or write specs and changes (`new change`, `status`, `instructions`,
`list`, `show`, `validate`, `archive`, `doctor`, `context`, `schemas`,
`view`). Once selected for the epic, treat `--store <id>` as sticky for epic
CLI calls. Every unscoped example of those commands below is shorthand: before
running it, append the flag. For example, run
`openspec status --change "<epic-name>" --json --store "<id>"`, not the
unscoped form. Other commands do not take the flag. Hints printed by commands
already carry the flag; keep it on follow-ups. Without a store, commands act on
the nearest local `openspec/` root.

Do not scan every registered store for epics unless the user asks. Resolve the
epic in the selected store or the nearest local root, same as other OpenSpec
skills.

### Per-story stores

A story may live in a different OpenSpec root than the epic. That is how one
initiative spans multiple repos.

- Optional field **Store**: a registered store id from `openspec store list --json`.
- Omit **Store** when the child change uses the same root as the epic.
- Child uniqueness is `(store-or-epic-root, Change name)`. Duplicate Change
  names are allowed only across different stores.
- When reading or writing a child (`list`, `status`, `new change`, propose,
  apply, archive), pass `--store "<story-store-id>"` if **Store** is set;
  otherwise use the epic's selected-root flags.
- If **Store** is set but that id is not registered, stop and report it. Do not
  guess a different store.
- NEXT-mode propose handoffs must include the story store when present, for
  example `/opsx-propose <child-change-name> --store <story-store-id>`.

## Select the epic

If the user names an epic, use it. Otherwise run:

```bash
openspec list --json
```

For each listed change in the selected root, inspect
`openspec status --change "<name>" --json` and filter by
`schemaName: epic-driven`; list output alone may not include the schema.
Auto-select only when exactly one exists or conversation context is unambiguous.
Otherwise ask the user.

Run:

```bash
openspec status --change "<epic-name>" --json
```

Confirm `schemaName` is `epic-driven`. Use `changeRoot`, `planningHome`,
`artifactPaths`, and `actionContext` from this output; do not assume paths.
Read every existing epic artifact from
`artifactPaths.<id>.existingOutputPaths`.

If planning is incomplete and the user did not ask for PROPOSE mode, report the
next ready artifact and recommend `/opsx-continue <epic-name>` or
`/opsx-epic-propose <epic-name>` to fill remaining artifacts. Do not infer
stories before `stories.md` exists.

## Parse stories

Read each `### Story <id>: <title>` block in `stories.md`.
Require these fields:

- Change
- Priority
- Depends on
- Outcome
- Acceptance signals
- Likely capabilities
- Parallel safe

Optional fields:

- Store (registered store id; default is the epic's OpenSpec root)

Parse the Story Index checkboxes when present:

```markdown
- [ ] <story-id>
- [x] <story-id>
```

A checked box means "projected done". It is not evidence of completion.

Require a stage heading for every story and a priority of P0, P1, P2, or P3.
Stop and report malformed entries, duplicate IDs, duplicate child change names
in the same store, unknown dependencies, unknown store ids, or dependency
cycles. Do not guess around an invalid graph.

## Derive status

For every story, resolve the child in **that story's** OpenSpec root:

1. If a matching archived change exists in that root, status is `done`.
2. Otherwise, if the active change does not exist in that root, status is
   `planned`.
3. Otherwise run:

   ```bash
   openspec status --change "<child-change>" --json
   ```

   with the story's `--store` when **Store** is set.

4. Classify the active child:
   - missing planning artifacts: `planning`
   - planning complete and no tracked task completed: `ready`
   - some tracked tasks complete: `in-progress`
   - all tracked tasks complete but still active: `implemented`

Use OpenSpec-reported paths. For a planning-complete child, query
`openspec instructions apply --change "<child-change>" --json` in that child's
root for task progress; this reads instructions, it does not authorize apply.
Do not request apply instructions for the epic. A child with no tracked tasks
is `ready`, not automatically `implemented`.

Resolve archives beneath the story root's reported changes directory. Match
either the exact child name or `YYYY-MM-DD-<complete-child-name>`, never an
arbitrary suffix that could belong to a different change. If both active and
archived matches exist, or multiple archives match, stop and report ambiguous
child identity rather than guessing which lifecycle the story represents.

## Determine the current stage

The current stage is the earliest stage containing any story that is not `done`.
A later stage is locked until every story in every earlier stage is `done`.

A planned story is `available` only when:

- it belongs to the current stage;
- every story in `Depends on` is `done`; and
- no active or archived child change already uses its Change name in that
  story's OpenSpec root.

Order available stories by priority (`P0`, `P1`, `P2`, `P3`), then by their
document order. Parallel-safe is advisory: report it, but do not treat it as a
dependency.

If the current stage has no available story, explain the exact blockers. Do not
select work from a later stage to bypass them.

## PROPOSE mode

Create the epic (if needed) and generate `vision.md`, `roadmap.md`, and
`stories.md` in one planning pass. This workflow authorizes planning only.
Do not edit product code. Do not start a child story or apply anything.

1. If no input, ask what initiative to plan. Derive a kebab-case epic name.
2. Resolve store selection for the **epic** as in Store selection.
3. Always use `--schema epic-driven`:

   ```bash
   openspec new change "<epic-name>" --schema epic-driven
   ```

   If a change with that name already exists, ask whether to fill missing
   artifacts or pick a new name.
4. Loop artifacts in schema order (`vision`, then `roadmap`, then `stories`).
   For each missing artifact, run `openspec instructions <id> --change "<epic-name>" --json`,
   read dependency files from disk, and write `resolvedOutputPath`. Re-run
   `openspec status --change "<epic-name>" --json` after each write.
5. When writing `stories.md`, if the initiative spans multiple OpenSpec roots,
   run `openspec store list --json` and set **Store** on each story that is not
   in the epic's root. Ask which stories belong to which store when that would
   change delivery ownership. Omit **Store** when every child shares the epic
   root.
6. Include a Story Index with an unchecked box per story ID. Do not check boxes
   during propose unless a child is already archived (unusual); prefer leaving
   them unchecked and letting SYNC project later.
7. Show `openspec status --change "<epic-name>"` and stop.

Prompt: artifacts are ready for review. Next:
`/opsx-epic-status <epic-name>`, then `/opsx-epic-next <epic-name>`.
Incremental alternative remains `/opsx-new` + `/opsx-continue`.

## SYNC mode

Project derived completion onto Story Index checkboxes. Do not implement, apply,
or run spec-sync.

1. Select the epic and parse stories.
2. Derive status for every story using that story's store/root.
3. Rewrite only checkbox markers in the Story Index:
   - `[x]` when derived status is `done`;
   - `[ ]` otherwise.
4. If the index is missing entries, append a line per story ID without changing
   titles, Change names, Store, or other body fields.
5. Do not check a box because tasks are complete but unarchived
   (`implemented`). Those stay unchecked until archive.
6. Summarize: boxes newly checked, newly unchecked, stores queried, and any
   implemented-but-unarchived warnings.

Do not invent a second progress model (no in-progress checkboxes).

## STATUS mode

Show:

- epic store / OpenSpec root;
- current stage and its exit criteria;
- counts for done, active, available, and blocked stories;
- each story's **Store** when set;
- each active child change and its OpenSpec planning/task progress (queried in
  that story's root);
- available stories in selection order;
- blocked stories with unmet dependency IDs;
- implemented-but-unarchived stories as an explicit warning;
- checkbox drift: any Story Index `[x]` whose child is not archived, or `[ ]`
  whose child is archived — recommend `/opsx-epic-sync <epic-name>`.

Do not write files in STATUS mode. Do not treat a checked box as `done`.

## NEXT mode

First derive status and available stories.

If the user specifies a story ID, require it to exist and be available before
planning it. Otherwise explain its blockers or existing child state. Never
substitute a different story silently.

If the user asks only what to work on, recommend the first available story and
show other parallel-safe candidates. Include this ready-to-run handoff
(include `--store` when the story or epic uses one):

```text
/opsx-propose <child-change-name> [--store <story-store-id>] from epic <epic-name>, story <story-id>:
<story title, outcome, acceptance signals, likely capabilities, store, and constraints>
```

If the user explicitly asks to start a story, announce the selected story and
invoke the installed `openspec-propose` skill with that handoff, using the
story's store as the propose workflow's sticky store. Explicitly require
`--schema spec-driven` when it creates the child, and confirm the child's
`schemaName` with `openspec status` afterward. If it is not `spec-driven`, stop
and report the mismatch. Do not recreate or abbreviate its planning workflow.

Require the child proposal to begin with:

```markdown
> Parent epic: `<epic-name>`  
> Roadmap story: `<story-id>`  
> OpenSpec store: `<story-store-id-or-epic-root>`
```

The proposal workflow is planning-only. Never carry an implementation request
through it. After proposal, specs, design, and tasks are ready, stop and wait for
a separate `/opsx-apply <child-change-name>` (with the story `--store` when set).

## REPLAN mode

Derive current state before proposing edits.

Replanning may:

- clarify the vision without contradicting delivered outcomes;
- edit, add, reorder, split, or remove only planned stories;
- assign or change **Store** only on planned stories;
- edit only future roadmap stages;
- append replacement work for completed or active behavior;
- add newly discovered dependencies without creating a cycle.

Replanning must not:

- rename or delete a story with an active or archived child change;
- change **Store** or Change name on a story whose child already exists;
- mutate completed stages;
- silently change active child scope;
- mark progress by editing checkboxes (use SYNC after archive instead).

Show proposed changes and their impact on available work. After user confirmation,
invoke `openspec-update-change` for the epic and pass these constraints. If an
active child needs revision, handle it separately with `openspec-update-change`
in that story's store.

After edits, re-read `stories.md`, validate IDs, names, stores, references, and
cycles, then show the newly available stories.

## Handoff after a story

- Plan epic: `/opsx-epic-propose <epic-name>`
- Plan story: `/opsx-propose <child-change>` (add `--store` when the story has one)
- Refine: `/opsx-update <child-change>`
- Implement: `/opsx-apply <child-change>`
- Verify when installed: `/opsx-verify <child-change>`
- Archive: `/opsx-archive <child-change>`
- Project checkboxes: `/opsx-epic-sync <epic-name>`
- Reassess epic: `/opsx-epic-status <epic-name>`

Archive the epic itself only after every included story is done or explicitly
removed while still planned, and the initiative success measures have been
reviewed.
