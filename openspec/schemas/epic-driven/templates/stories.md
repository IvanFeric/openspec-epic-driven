## Story Index

Keep story IDs and change names stable after a child change has been created.
Document order breaks ties between stories with the same priority.
`/opsx-epic-sync` checks a box only when that story's child change is archived.

- [ ] E1-S1
- [ ] E1-S2
- [ ] E2-S1

## Stage 1

### Story E1-S1: <Short outcome-oriented title>

- **Change**: `<unique-kebab-case-change-name>`
- **Store**: `<registered-store-id>` (omit this field when the child uses the epic's OpenSpec root)
- **Priority**: P1
- **Depends on**: none
- **Outcome**: <Independent value delivered by this story>
- **Acceptance signals**:
  - <Observable behavior or verification signal>
- **Likely capabilities**:
  - <Existing or proposed OpenSpec capability path, or "discovery required">
- **Parallel safe**: yes | no — <reason, likely overlap, or coordination boundary>

### Story E1-S2: <Short outcome-oriented title>

- **Change**: `<unique-kebab-case-change-name>`
- **Priority**: P1
- **Depends on**: E1-S1
- **Outcome**: <Independent value delivered by this story>
- **Acceptance signals**:
  - <Observable behavior or verification signal>
- **Likely capabilities**:
  - <Capability path>
- **Parallel safe**: no — depends on E1-S1

## Stage 2

### Story E2-S1: <Short outcome-oriented title>

- **Change**: `<unique-kebab-case-change-name>`
- **Priority**: P2
- **Depends on**: E1-S1, E1-S2
- **Outcome**: <Independent value delivered by this story>
- **Acceptance signals**:
  - <Observable behavior or verification signal>
- **Likely capabilities**:
  - <Capability path>
- **Parallel safe**: yes | no — <reason>
