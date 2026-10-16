# Epic-Driven OpenSpec

An [OpenSpec](https://github.com/Fission-AI/OpenSpec) community schema and companion
agent skill for planning a complex initiative as a staged roadmap and delivering
it through independently reviewable, story-sized changes.

**One epic plans the initiative; child changes deliver it.** The epic produces
`vision.md`, `roadmap.md`, and `stories.md`. Each selected story follows the
existing `spec-driven` workflow: proposal, delta specs, design, tasks,
implementation, verification, and archive.

## How it works

```text
Explore -> Epic planning -> Select a ready story -> Normal story workflow
                  ^                                        |
                  +------ Sync, reassess, replan <---------+

Epic artifacts:  vision -> roadmap -> stories
Child artifacts: proposal -> specs / design -> tasks
```

OpenSpec schemas describe artifact dependencies **within one change**. The
companion skill adds the thin cross-change layer: story dependencies, stage
gates, selection, progress, and replanning. OpenSpec remains the system of record;
there is no separate implementation engine or project-management database.

The epic schema deliberately has **no `apply` phase**. Product code changes only
through child changes, and their capability specs merge into `openspec/specs/`
through the normal archive workflow. Keep the epic active while its children
are delivered.

**This is a workflow boundary, not a CLI lock.** OpenSpec can return generic
apply instructions even for a schema without an `apply` block. Do not apply the
epic regardless of that output; the companion skill explicitly forbids it.

## Requirements

- OpenSpec CLI **1.10 or later**. The source design targets 1.10.0; schema commands
  are experimental, so pin your team's CLI and revalidate after upgrades.
- An initialized OpenSpec project with **`spec-driven` as its default schema**.
- The core OpenSpec workflows, including the installed `openspec-propose` skill
  for child planning and the normal apply/archive workflows.
- The `openspec-update-change` workflow for replanning.
- Optional `new` and `continue` workflows for incremental epic planning, and
  `verify` for child verification.
- An assistant that discovers `.agents/skills/` or `.github/skills/`.
- For cross-repository stories, a CLI with registered-store support
  (`openspec store list --json` and `--store`). Local-only use needs no stores.

## Repository layout

This is a copyable community bundle, not an OpenSpec core modification or an
automatically installed plugin.

```text
openspec-epic-driven/
|-- README.md
|-- LICENSE
|-- openspec/
|   `-- schemas/
|       `-- epic-driven/
|           |-- schema.yaml
|           `-- templates/
|               |-- vision.md
|               |-- roadmap.md
|               `-- stories.md
|-- .agents/skills/openspec-epic-management/SKILL.md
|-- .cursor/commands/
|   |-- opsx-epic-propose.md
|   |-- opsx-epic-status.md
|   |-- opsx-epic-next.md
|   |-- opsx-epic-sync.md
|   `-- opsx-epic-replan.md
|-- .github/
|   |-- skills/openspec-epic-management/SKILL.md
|   `-- prompts/
|       |-- opsx-epic-propose.prompt.md
|       |-- opsx-epic-status.prompt.md
|       |-- opsx-epic-next.prompt.md
|       |-- opsx-epic-sync.prompt.md
|       `-- opsx-epic-replan.prompt.md
`-- examples/notification-platform/proposal.md
```

The two `SKILL.md` files have identical contents. Cursor commands and Copilot
prompts are thin adapters that invoke the skill, not independent implementations.
The example proposal is illustrative, not an active change or another template.

## Install

1. Check `openspec --version`. Run `openspec init` in the target project if it
   does not already use OpenSpec.
2. Copy [`openspec/schemas/epic-driven/`](openspec/schemas/epic-driven/) into the
   target project's `openspec/schemas/epic-driven/`.
3. Install the companion skill and adapters for your assistant:

| Assistant | Copy into the target project |
|-----------|------------------------------|
| Portable skill consumers | `.agents/skills/openspec-epic-management/` |
| Cursor | The portable skill above and the five `.cursor/commands/opsx-epic-*.md` files |
| GitHub Copilot | `.github/skills/openspec-epic-management/` and the five `.github/prompts/opsx-epic-*.prompt.md` files for IDE use |

Copy only the named extension files; do not replace existing assistant
directories or unrelated workflows. Install both skill locations if different
assistants in your team need them, and keep their contents identical.

Retain the normal workflows. To enable incremental planning or optional workflows:

```sh
openspec config profile
openspec update
```

Retain the core workflows and select `propose`, `update`, and, as needed, `new`,
`continue`, and `verify`. Profile selection is machine-global; `openspec update`
refreshes generated integrations in the current project.

Keep this setting in the target project's `openspec/config.yaml`, preserving its
other context and rules:

```yaml
schema: spec-driven
```

Validate installation from the target project root:

```sh
openspec schema validate epic-driven
openspec schema which epic-driven
openspec schemas
```

`schema which` should resolve to the copied project bundle. Select `epic-driven`
explicitly only for epic changes; **do not make it the project default**.

The `/opsx-epic-*` examples below refer to the supplied adapters. Copilot IDE
integrations use the `.prompt.md` wrappers. Copilot CLI does not consume those
prompt files directly; invoke the installed `openspec-epic-management` skill by
name and specify PROPOSE, STATUS, NEXT, SYNC, or REPLAN mode instead.

## Usage

### Plan an epic

Explore the initiative, then propose all three epic artifacts in one planning
pass:

```text
/opsx-explore notification-platform
/opsx-epic-propose notification-platform
```

This creates an `epic-driven` change and writes vision, roadmap, and stories.
Review the artifacts before selecting implementation work. Proposal is
planning-only: it neither starts a child nor edits product code.

For incremental planning, use the optional built-in workflows instead:

```text
/opsx-new notification-platform --schema epic-driven
/opsx-continue notification-platform
/opsx-continue notification-platform
/opsx-continue notification-platform
```

Each continue invocation creates the next artifact: vision, then roadmap, then
stories.

### Select and plan one story

```text
/opsx-epic-status notification-platform
/opsx-epic-next notification-platform
/opsx-epic-next notification-platform E1-S1 start
```

Without `start`, NEXT recommends available work without writing files. With
`start`, it delegates exactly one available story to the installed
`openspec-propose` skill. The resulting child uses `spec-driven`, retains a
parent-epic/story reference in its proposal, and stops before implementation.

### Deliver, sync, and reassess

For a child named `establish-preferences`:

```text
/opsx-apply establish-preferences
/opsx-verify establish-preferences
/opsx-archive establish-preferences
/opsx-epic-sync notification-platform
/opsx-epic-status notification-platform
```

Use verify when installed. Archive is the completion signal: completed task
checkboxes alone do not complete a story. Epic sync updates the Story Index
checkboxes after looking up archived children. It is **not** `/opsx-sync`, which
merges delta specs and must not run against the epic.

When delivery changes future plans:

```text
/opsx-epic-replan notification-platform provider selection must precede templates
```

REPLAN shows the proposed changes and requires confirmation. It may edit planned
stories and future stages, but preserves active child scope and completed history.
Refine active child scope separately through `/opsx-update <child-change>`.

## Artifacts and story contract

| Artifact | Purpose |
|----------|---------|
| [`vision.md`](openspec/schemas/epic-driven/templates/vision.md) | Problem, consumers, measurable outcomes, scope, constraints, assumptions, and success measures |
| [`roadmap.md`](openspec/schemas/epic-driven/templates/roadmap.md) | Incremental, demonstrable stages with dependencies, exit criteria, and what each stage unlocks |
| [`stories.md`](openspec/schemas/epic-driven/templates/stories.md) | Story Index and ordered, dependency-aware candidates for child changes |

Each `### Story <id>: <title>` block belongs to a stage and has a stable ID,
unique kebab-case **Change** name, **Priority** (`P0` through `P3`), **Depends on**
IDs (or `none`), **Outcome**, **Acceptance signals**, **Likely capabilities**, and
**Parallel safe** note. Optional **Store** selects a registered OpenSpec root.
Design decisions and implementation task lists belong in the child, not the epic.

Duplicate IDs, duplicate child names within a root, unknown dependencies, unknown
store IDs, and dependency cycles stop selection rather than being guessed around.
Keep IDs, child names, and stores stable once a child exists.

## Derived status and stage gates

| State | Evidence in the child's OpenSpec root |
|-------|--------------------------------------|
| `planned` | No active or archived child |
| `planning` | Active child with missing planning artifacts |
| `ready` | Planning complete; no tracked tasks completed |
| `in-progress` | Some tracked tasks completed |
| `implemented` | All tasks complete, but the child is still active |
| `done` | Matching archived child, including date-prefixed archive directories |

The **current stage** is the earliest stage with an unfinished story. Later stages
stay locked until every story in every earlier stage is archived. A planned story
is **available** only in the current stage, with all dependencies done and no
existing child using its name in its root. Order available stories by priority,
then document order. Parallel safety is advisory, not an extra dependency.

STATUS is read-only: it reports progress, available work, blockers, unarchived
implementations, and checkbox drift. Only SYNC projects archive state onto the
Story Index: `[x]` for done, `[ ]` otherwise. Do not check story boxes by hand.
Stage exit criteria remain review criteria; story checkboxes are not a second
progress database.

## Multiple repositories and stores

Without a store selection, use the nearest local `openspec/` root. If the epic
lives in a registered store, discover it with `openspec store list --json` and
keep `--store <id>` on the epic's supported CLI calls.

Set a story's optional **Store** field only when it targets a different root;
otherwise it inherits the epic's root. Child identity is
`(store-or-epic-root, change name)`, so the same child name may appear in different
stores but not twice in one root. Unknown stores are a hard stop. Do not scan all
registered stores unless requested.

Propose, inspect, apply, verify, and archive each child in **its own** store,
passing that story's `--store` through the normal workflow:

```text
/opsx-propose email-provider-adapter --store delivery-service
/opsx-apply email-provider-adapter --store delivery-service
/opsx-archive email-provider-adapter --store delivery-service
```

The skill documents which CLI commands accept the store flag. Install the epic
schema in the root where epic planning occurs, and ensure every child root
provides the normal `spec-driven` workflow.

## Worked example

In Stage 1 of a notification platform:

```text
E1-S1 establish-preferences --+
                            +--> E1-S3 notification-api
E1-S2 email-provider-adapter-+
```

E1-S1 and E1-S2 have no dependencies and may be parallel-safe. NEXT recommends the
higher-priority candidate, breaking ties by document order. E1-S3 remains blocked
until both foundations are archived; then SYNC checks their index boxes and
E1-S3 becomes available.

See the extracted [child proposal excerpt](examples/notification-platform/proposal.md)
for the parent-epic, story-ID, and root traceability header.

## Validation and maintenance

After changing the schema, run `openspec schema validate epic-driven` and
`openspec schema which epic-driven`. For lifecycle checks, use a disposable
initialized project, not a real project's changes:

- A new epic has vision ready and roadmap/stories blocked; each completed
  artifact unlocks the next.
- PROPOSE generates all three artifacts; incremental CONTINUE generates one.
- The epic has no apply phase and epic workflows never edit product code.
- NEXT plans a normal `spec-driven` child and stops before implementation.
- Invalid story graphs stop selection; later stages cannot bypass blockers.
- Date-prefixed archives count as done in the correct story root.
- STATUS never writes; SYNC checks only archived children and clears stale boxes.
- REPLAN cannot rename, move, or delete active or archived stories.

Schema validation checks the artifact graph and templates, not the agent's
cross-change behavior. Exercise the skill scenarios separately with your
assistant. Keep both skill copies identical and adapter modes consistent.

## Boundaries

There is no epic-wide apply, cross-change graph in OpenSpec core, hand-maintained
progress queue, or Jira/GitLab synchronization. The skill orchestrates existing
workflows; it does not replace them or enforce behavior through executable hooks.
New learning changes future work or adds replacement stories, never silently
rewrites delivery history.

Archive the epic only after every included story is done (or explicitly removed
while still planned) and the initiative success measures have been reviewed.

## References

- [OpenSpec customization and community schemas](https://github.com/Fission-AI/OpenSpec/blob/main/docs/customization.md)
- [OpenSpec workflow profiles](https://openspec.dev/docs/profiles)
- [OPSX artifact workflow](https://openspec.dev/docs/opsx)
- [Supported assistant integrations](https://openspec.dev/docs/reference/supported-tools)
- [AIDE Spec Kit extension](https://github.com/mnriem/spec-kit-extensions/tree/main/aide), a design reference

## License

[MIT](LICENSE).
