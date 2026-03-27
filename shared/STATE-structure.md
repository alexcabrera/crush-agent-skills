# STATE.md Structure

The STATE.md file is the control plane for the agent-skills workflow. It persists workflow state across context resets and enables context-recovery patterns.

## Location

```
.tickets/STATE.md
```

## Format

```yaml
---
phase: init | design | plan | validate | execution | verification | gap-closure | done
epic: E-001
focus: T-003
started: 2024-01-15T10:30:00Z
learnings:
  - "Pattern discovered: using context.cancel for graceful shutdown"
decisions:
  - "[2024-01-15] Chose SQLite over JSON for state persistence"
---
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `phase` | enum | yes | Current workflow phase |
| `epic` | string | no | Current epic ticket ID |
| `focus` | string | no | Current ticket being worked on |
| `started` | ISO 8601 | yes | When this workflow started |
| `learnings` | string[] | no | Patterns/insights discovered |
| `decisions` | string[] | no | Architecture decisions with dates |

## Phase Values

| Phase | Description |
|-------|-------------|
| `init` | Project initialized, no design yet |
| `design` | Active design/requirements clarification |
| `plan` | Creating ticket structure |
| `validate` | Verifying ticket plan |
| `execution` | Implementing tickets |
| `verification` | Verifying implementation |
| `gap-closure` | Fixing verification gaps |
| `done` | All work complete |

## Usage by Skills

### Reading STATE.md

All skills should read STATE.md at the start to understand current context:

```bash
# Parse YAML frontmatter
head -n 20 .tickets/STATE.md | sed -n '/^---$/,/^---$/p' | tail -n +2 | head -n -1
```

### Updating STATE.md

Skills update specific fields:

| Skill | Updates |
|-------|---------|
| `design` | `phase: design`, `decisions` |
| `plan` | `phase: plan`, `epic` |
| `validate` | `phase: validate` |
| `execute` | `phase: execution`, `focus` |
| `verify` | `phase: verification` |
| `close-gaps` | `phase: gap-closure`, `learnings` |

### Adding Learnings

When a pattern or insight is discovered:

```yaml
learnings:
  - "Retry with exponential backoff works better than fixed delay"
```

Also add to epic ticket via `tk add-note` for long-term reference.

### Recording Decisions

When making architecture decisions:

```yaml
decisions:
  - "[2024-01-15] Using parallel execution for independent tickets"
```

## Initialization

When starting a new project:

```yaml
---
phase: init
started: 2024-01-15T10:30:00Z
learnings: []
decisions: []
---
```

## Crash Recovery

If the agent crashes during execution:

1. Read STATE.md to determine last known state
2. Check `phase` to know which skill was active
3. Check `focus` to know which ticket was being processed
4. Prompt user to resume or restart from that point
