# choo-choo Design

> "I choo-choo-choose you" - Ralph Wiggum's Valentine card to Lisa

## Overview

choo-choo is a terminal-based orchestration layer that wraps the Crush coding agent, providing a structured workflow from design through autonomous execution. It combines Ralph Loop's fresh-context-per-session pattern with goal-backward verification, using the `tk` CLI for ticket management.

**Important: Skills are independent of choo-choo.** The 6 skills (`design`, `plan`, `validate`, `execute`, `verify`, `close-gaps`) are general-purpose Crush skills installed in `~/.config/crush/skills/`. They can be used:
- Directly in interactive Crush sessions
- Via `crush run "use the design skill to help me..."`
- Orchestrated by choo-choo for autonomous workflows

choo-choo adds visualization, coordination, and parallelization - but provides no unique capabilities. A user could accomplish the same workflow manually in Crush using the skills directly.

The system guides users through: **design → plan → validate → execute → verify → close-gaps**, with human gates at key decision points and full visibility into agent activity via a rich TUI built with bubbletea.

## Detailed Requirements

### Functional Requirements

**Design Phase**
- FR1: Interactive chat interface for requirements clarification
- FR2: Two-pane layout (chat + artifact preview)
- FR3: Invoke Crush with `design` skill for AI-assisted design
- FR4: Output: requirements.md, design.md in `specs/` directory

**Plan Phase**
- FR5: Non-interactive execution of `plan` skill
- FR6: Generate ticket tree with hierarchy (epic → stories → tasks)
- FR7: Establish dependencies between tickets
- FR8: Output: Ticket files in `.tickets/` directory

**Validate Phase**
- FR9: Run `validate` skill to check dependency integrity
- FR10: Verify ticket completeness (descriptions, acceptance criteria)
- FR11: Validate environment (skills exist, tk CLI available)
- FR12: Output deterministic execution order

**Execute Phase**
- FR13: Parallel execution of independent tickets (by level)
- FR14: Kanban visualization (TODO → DOING → DONE)
- FR15: Tabbed ticket popups (details, log, diff)
- FR16: Sequential fallback via `--sequential` flag

**Verify Phase**
- FR17: Per-ticket verification after execution
- FR18: Verification report generation
- FR19: Integration with acceptance criteria from tickets

**Close-gaps Phase**
- FR20: Automatic fix loop with max retry limit (3)
- FR21: Escalate to STATE.md if max retries exceeded

**State Management**
- FR22: STATE.md for session handoff across phases
- FR23: Phase tracking (design → plan → validate → execute → verify → close-gaps)
- FR24: Learnings logged to epic ticket via `tk add-note`

### Non-Functional Requirements

- NFR1: TUI built with Charm bubbletea v2 ecosystem
- NFR2: All AI work delegated to Crush (choo-choo is orchestration only)
- NFR3: Responsive UI during background execution
- NFR4: Crash recovery via STATE.md persistence
- NFR5: Work with any Crush-supported LLM provider

### Constraints

- C1: Skills installed in `~/.config/crush/skills/` (symlinked from `~/Code/agent-skills/choo-choo-skills/`)
- C2: Ticket management via `tk` CLI (included in project)
- C3: SQLite database at `~/.crush/crush.db` (read-only access)
- C4: Must not interfere with interactive Crush sessions

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        choo-choo TUI                             │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    bubbletea Application                     ││
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐││
│  │  │   Chat UI    │  │   Kanban     │  │    Ticket Popup      │││
│  │  │  (textarea)  │  │  (3 columns) │  │ (tabs: details/log/  │││
│  │  │  (viewport)  │  │              │  │       diff)          │││
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘││
│  │                    │                           │              ││
│  │                    ▼                           ▼              ││
│  │              ┌─────────────────────────────────────────┐     ││
│  │              │          Artifact Preview               │     ││
│  │              │    (viewport with glamour rendering)    │     ││
│  │              └─────────────────────────────────────────┘     ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Orchestrator (Go)                         ││
│  │  - Phase management                                          ││
│  │  - Crush process spawning                                    ││
│  │  - Parallel execution coordination                           ││
│  │  - STATE.md read/write                                       ││
│  │  - tk CLI integration                                        ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        ┌──────────┐    ┌──────────┐    ┌──────────────┐
        │  Crush   │    │  tk CLI  │    │   SQLite     │
        │ (headless│    │ (ticket  │    │  (read-only) │
        │   mode)  │    │  system) │    │ ~/.crush/db  │
        └──────────┘    └──────────┘    └──────────────┘
              │
              ▼
        ┌──────────────────────────────────────────┐
        │           Skills (6 total)               │
        │  design, plan, validate, execute,        │
        │  verify, close-gaps                      │
        └──────────────────────────────────────────┘
```

### Key Architectural Decisions

1. **TUI as orchestration layer**: choo-choo doesn't do AI work - it coordinates Crush invocations and visualizes progress
2. **STATE.md as control plane**: Single file for phase tracking, enables crash recovery and fresh-context-per-session
3. **Parallel execution by level**: validate outputs dependency levels, choo-choo executes each level in parallel
4. **Full Crush delegation**: All LLM interactions go through `crush run`, leveraging existing skills and tooling

## Components and Interfaces

### Orchestrator

**Purpose:** Core coordination logic for the entire application

**Responsibilities:**
- Manage application phases (init → design → plan → validate → execute → verify → close-gaps → done)
- Spawn and monitor Crush processes
- Coordinate parallel execution
- Read/write STATE.md
- Execute tk CLI commands

**Interface:**
```go
type Orchestrator struct {
    projectDir   string
    state        *State
    crushPath    string
    tkPath       string
}

func (o *Orchestrator) Init() error
func (o *Orchestrator) RunDesign(ctx context.Context, prompt string) (<-chan StreamEvent, error)
func (o *Orchestrator) RunPlan(ctx context.Context) (<-chan StreamEvent, error)
func (o *Orchestrator) RunValidate(ctx context.Context) (*ValidationResult, error)
func (o *Orchestrator) RunExecute(ctx context.Context, parallel bool) (<-chan TicketEvent, error)
func (o *Orchestrator) GetPhase() Phase
func (o *Orchestrator) GetTickets() ([]Ticket, error)
func (o *Orchestrator) ReadArtifact(path string) (string, error)
```

**Dependencies:**
- Crush binary (via `exec.Command`)
- tk CLI (bundled or project-local)
- STATE.md file

---

### State Manager

**Purpose:** Manage STATE.md for session persistence and handoff

**Responsibilities:**
- Parse STATE.md with YAML frontmatter
- Update phase, focus ticket, learnings, decisions
- Provide atomic read/write operations

**Interface:**
```go
type State struct {
    Phase       Phase     `yaml:"phase"`
    Epic        string    `yaml:"epic"`
    Focus       string    `yaml:"focus"`
    Started     time.Time `yaml:"started"`
    Learnings   []string  `yaml:"learnings"`
    Decisions   []string  `yaml:"decisions"`
}

type Phase string

const (
    PhaseInit       Phase = "init"
    PhaseDesign     Phase = "design"
    PhasePlan       Phase = "plan"
    PhaseValidate   Phase = "validate"
    PhaseExecute    Phase = "execution"
    PhaseVerify     Phase = "verification"
    PhaseCloseGaps  Phase = "gap-closure"
    PhaseDone       Phase = "done"
)

func LoadState(path string) (*State, error)
func (s *State) Save(path string) error
func (s *State) SetPhase(p Phase)
func (s *State) SetFocus(ticketID string)
func (s *State) AddLearning(l string)
func (s *State) AddDecision(d string)
```

---

### Crush Runner

**Purpose:** Execute Crush in headless mode and capture output

**Responsibilities:**
- Build Crush command with appropriate flags
- Stream stdout/stderr to channels
- Handle process lifecycle (start, wait, kill)
- Parse streaming output for progress updates

**Interface:**
```go
type CrushRunner struct {
    crushPath string
    workDir   string
    sessionID string
}

type StreamEvent struct {
    Type    EventType
    Content string
    Time    time.Time
}

func (r *CrushRunner) Run(ctx context.Context, prompt string, opts RunOptions) (<-chan StreamEvent, error)
func (r *CrushRunner) RunWithSession(ctx context.Context, sessionID, prompt string) (<-chan StreamEvent, error)
func (r *CrushRunner) ContinueSession(sessionID string)

type RunOptions struct {
    Quiet   bool
    Yolo    bool
    Model   string
}
```

---

### Ticket Manager

**Purpose:** Interface with tk CLI for ticket operations

**Responsibilities:**
- Execute tk commands
- Parse ticket YAML frontmatter
- Query ticket hierarchy and dependencies
- Update ticket status

**Interface:**
```go
type Ticket struct {
    ID          string
    Type        TicketType // epic, story, task
    Title       string
    Description string
    Status      Status // todo, doing, done, blocked
    Parent      string
    Dependencies []string
    Accepts     []string // acceptance criteria
    Notes       []string
}

type TicketManager struct {
    tkPath     string
    ticketsDir string
}

func (m *TicketManager) List() ([]Ticket, error)
func (m *TicketManager) Get(id string) (*Ticket, error)
func (m *TicketManager) GetChildren(parentID string) ([]Ticket, error)
func (m *TicketManager) GetDependencies(id string) ([]Ticket, error)
func (m *TicketManager) SetStatus(id string, status Status) error
func (m *TicketManager) AddNote(id, note string) error
func (m *TicketManager) GetExecutionOrder() ([]string, error) // from validate output
```

---

### TUI Model

**Purpose:** bubbletea application model

**Responsibilities:**
- Render all UI components
- Handle keyboard input
- Route messages to appropriate components
- Manage focus state

**Interface:**
```go
type Model struct {
    phase       Phase
    orchestrator *Orchestrator
    
    // Components
    chat        ChatModel
    kanban      KanbanModel
    preview     PreviewModel
    popup       *PopupModel
    
    // State
    width       int
    height      int
    focus       FocusArea
    loading     bool
    errors      []string
}

type FocusArea int

const (
    FocusChat FocusArea = iota
    FocusKanban
    FocusPreview
    FocusPopup
)

func InitialModel() Model
func (m Model) Init() tea.Cmd
func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd)
func (m Model) View() string
```

---

### Chat Model

**Purpose:** Interactive chat interface for design phase

**Interface:**
```go
type ChatModel struct {
    viewport    viewport.Model
    textarea    textarea.Model
    messages    []ChatMessage
    spinner     spinner.Model
}

type ChatMessage struct {
    Role      Role // user, assistant
    Content   string
    Timestamp time.Time
}

func (m ChatModel) Update(msg tea.Msg) (ChatModel, tea.Cmd)
func (m ChatModel) View() string
func (m *ChatModel) AddMessage(role Role, content string)
func (m *ChatModel) SetStreaming(streaming bool)
```

---

### Kanban Model

**Purpose:** Visualize ticket execution progress

**Interface:**
```go
type KanbanModel struct {
    columns     [3][]Ticket // TODO, DOING, DONE
    cursorCol   int
    cursorRow   int
    viewport    viewport.Model
}

func (m KanbanModel) Update(msg tea.Msg) (KanbanModel, tea.Cmd)
func (m KanbanModel) View() string
func (m *KanbanModel) SetTickets(tickets []Ticket)
func (m *KanbanModel) MoveTicket(id string, from, to Status)
func (m *KanbanModel) SelectedTicket() *Ticket
```

---

### Popup Model

**Purpose:** Detailed ticket view with tabs

**Interface:**
```go
type PopupModel struct {
    open        bool
    ticket      *Ticket
    activeTab   Tab
    tabs        []TabContent
}

type Tab int

const (
    TabDetails Tab = iota
    TabLog
    TabDiff
)

type TabContent struct {
    title   string
    content string
    viewport viewport.Model
}

func (m PopupModel) Update(msg tea.Msg) (PopupModel, tea.Cmd)
func (m PopupModel) View() string
func (m *PopupModel) Open(ticket *Ticket)
func (m *PopupModel) Close()
func (m *PopupModel) SetLog(log string)
func (m *PopupModel) SetDiff(diff string)
```

## Data Models

### STATE.md

```yaml
---
phase: design | plan | validate | execution | verification | gap-closure | done
epic: E-001
focus: T-003
started: 2024-01-15T10:30:00Z
learnings:
  - "Pattern: Using context.cancel for graceful shutdown"
decisions:
  - "[2024-01-15] Using parallel execution for independent tickets"
---
```

**Validation Rules:**
- `phase` must be one of the defined phases
- `epic` must reference existing epic ticket
- `focus` must reference existing ticket (any type)
- `started` must be valid ISO 8601 timestamp

### Ticket File (via tk CLI)

```yaml
---
id: T-003
type: task
title: Implement State Manager
status: todo | doing | done | blocked
parent: S-001
depends_on:
  - T-001
  - T-002
accepts:
  - "State can be loaded from STATE.md"
  - "State can be saved atomically"
  - "Invalid YAML returns clear error"
created: 2024-01-15T10:30:00Z
updated: 2024-01-15T14:22:00Z
---

## Description

Implement the State struct and methods for managing STATE.md...

## Notes

- [2024-01-15] Pattern: using yaml.v3 for parsing
```

### Validation Result

```yaml
valid: true
execution_plan:
  parallel: true
  levels:
    - level: 0
      tickets: [T-001, T-002, T-003]
    - level: 1
      tickets: [T-004, T-005]
    - level: 2
      tickets: [T-006]
checks:
  - name: dependency_cycle
    passed: true
  - name: missing_dependencies
    passed: true
  - name: ticket_completeness
    passed: true
    warnings: ["T-003 has no acceptance criteria"]
  - name: environment
    passed: true
```

## Error Handling

### Error Categories

| Category | Example | User Message | Recovery |
|----------|---------|--------------|----------|
| Crush Error | LLM rate limit | "AI request failed. Retrying in 30s..." | Auto-retry with backoff |
| Crush Error | Invalid prompt | "Failed to process request" | Show error, allow retry |
| Validation Error | Dependency cycle | "Cannot execute: tickets have circular dependencies" | Block execution, show cycle |
| Validation Error | Missing skill | "Skill 'execute' not found in ~/.config/crush/skills/" | Block execution, show install instructions |
| Process Error | Crush crashed | "Agent process terminated unexpectedly" | Log to STATE.md, allow restart |
| File Error | STATE.md corrupted | "Could not read state file" | Prompt to restore or reinit |
| Max Retries | Ticket stuck | "Ticket T-003 failed after 3 attempts" | Mark blocked, continue others, log to STATE.md |

### Recovery Strategies

1. **Crush process crash**: Log error, mark ticket as blocked, continue with remaining tickets
2. **Network timeout**: Retry with exponential backoff (3 attempts)
3. **STATE.md corruption**: Attempt backup restore, or prompt user to reinitialize
4. **Max retries exceeded**: Log to STATE.md, mark ticket blocked, continue parallel group

## Acceptance Criteria

### AC1: Project Initialization
- **Given:** Empty directory
- **When:** User runs `choo-choo`
- **Then:** `.tickets/`, `specs/` directories created, STATE.md initialized with `phase: init`

### AC2: Design Phase Chat
- **Given:** Initialized project in init or design phase
- **When:** User types a message and presses Enter
- **Then:** Message sent to Crush, response streamed back, artifacts appear in preview pane

### AC3: Design Gate
- **Given:** Design phase complete (requirements.md, design.md exist)
- **When:** User approves design
- **Then:** Phase transitions to plan, `plan` skill invoked

### AC4: Plan Generation
- **Given:** design.md exists
- **When:** Plan phase starts
- **Then:** Tickets created in `.tickets/`, hierarchy and dependencies established

### AC5: Validation Gate
- **Given:** Tickets exist
- **When:** Validate phase runs
- **Then:** Validation report shown, execution plan generated, user can approve or modify

### AC6: Parallel Execution
- **Given:** Approved execution plan with parallel levels
- **When:** Execute phase starts
- **Then:** Level 0 tickets execute concurrently, level 1 waits for level 0, etc.

### AC7: Sequential Fallback
- **Given:** `--sequential` flag provided
- **When:** Execute phase starts
- **Then:** Tickets execute one at a time in order, no parallelism

### AC8: Kanban Visualization
- **Given:** Execution in progress
- **When:** User views kanban
- **Then:** Tickets shown in correct columns, move from TODO → DOING → DONE as they complete

### AC9: Ticket Popup Details
- **Given:** Ticket in DOING column
- **When:** User presses Enter on ticket
- **Then:** Popup opens with tabs for Details, Log, Diff

### AC10: Verify-Fix Loop
- **Given:** Ticket execution complete
- **When:** Verification finds gaps
- **Then:** close-gaps skill runs, loops back to verify, max 3 attempts

### AC11: Max Retries Escalation
- **Given:** Ticket failed verification 3 times
- **When:** Max retries exceeded
- **Then:** Ticket marked blocked, learning logged to STATE.md, execution continues with remaining tickets

### AC12: Crash Recovery
- **Given:** choo-choo crashed during execution
- **When:** User restarts choo-choo
- **Then:** STATE.md read, phase restored, user prompted to continue from last known state

## Testing Strategy

### Unit Tests

- **State Manager**: Load/save STATE.md, YAML parsing edge cases
- **Ticket Manager**: Parse tk output, build dependency graph
- **Orchestrator**: Phase transitions, Crush command building
- **Crush Runner**: Process lifecycle, output streaming

### Integration Tests

- **End-to-end design phase**: Mock Crush responses, verify artifacts created
- **Ticket generation**: Run plan skill (mocked), verify ticket structure
- **Validation cycle**: Generate tickets with cycles, verify detection
- **Parallel execution**: Simulate multiple Crush processes, verify coordination

### Edge Cases to Test

- Empty project directory
- STATE.md with invalid YAML
- Crush process killed mid-execution
- Network timeout during LLM call
- Circular dependencies in tickets
- Ticket with no acceptance criteria
- All tickets blocked scenario
- Concurrent file access (STATE.md)

### Manual Testing Scenarios

1. Full happy path: init → design → plan → validate → execute → verify → done
2. Crash and recovery: kill choo-choo during execution, restart
3. Sequential fallback: run with `--sequential`, verify no parallelism
4. Popup interaction: open ticket popup, switch tabs, close
5. Resize handling: shrink terminal, verify layout adapts

## Appendices

### A. Technology Choices

| Decision | Options Considered | Chosen | Rationale |
|----------|-------------------|--------|-----------|
| TUI Framework | bubbletea, tview, termui | bubbletea | Charm ecosystem, Crush uses it, v2 improvements |
| State Storage | SQLite, JSON, YAML file | YAML file (STATE.md) | Human-readable, git-friendly, simple |
| Ticket System | Custom, tk CLI, GitHub Issues | tk CLI | Already exists, file-based, has dependencies |
| AI Backend | Direct API, Crush, Claude Code | Crush | Existing skills, tool ecosystem, session management |
| Process Management | Go exec, goroutines, tmux | Go exec + goroutines | Native Go, good control, parallel support |
| Styling | lipgloss, termenv | lipgloss | Charm standard, pairs with bubbletea |

### B. Research Findings Summary

**Crush Headless Capabilities:**
- `crush run --quiet --yolo "prompt"` for non-interactive execution
- Session persistence via `--continue` and `--session`
- Skills auto-loaded from `~/.config/crush/skills/`
- SQLite DB at `~/.crush/crush.db` safe for read-only access
- No external event API (polling required for real-time updates)

**Ralph TUI Patterns:**
- Loop: SELECT TASK → BUILD PROMPT → EXECUTE → DETECT COMPLETION → NEXT
- Uses prd.json for task tracking
- Headless mode available
- Real-time TUI monitoring

**bubbletea Architecture:**
- "Smart model, dumb components" pattern
- Single Update() loop with message routing
- pubsub via program.Send() for background events
- lipgloss for layout (JoinHorizontal, JoinVertical)

### C. Alternative Approaches

**Alternative 1: TUI as separate viewer (Option C from Q9)**
- Description: Run Crush separately, choo-choo reads DB
- Pros: Full interactive Crush experience
- Cons: Two terminals required, choo-choo is passive during design
- Why not chosen: User wanted integrated experience

**Alternative 2: Embedded AI (no Crush)**
- Description: choo-choo calls LLM APIs directly
- Pros: No external dependency
- Cons: Reimplements Crush functionality, loses skill ecosystem
- Why not chosen: Crush provides mature tooling and skills

**Alternative 3: Sequential only (no parallel)**
- Description: Execute tickets one at a time
- Pros: Simpler implementation, clean git history
- Cons: Slower for independent tickets
- Why not chosen: Can always use `--sequential` if needed, parallel is opt-in

### D. Skill Contracts (Independent of choo-choo)

Each skill is a standalone Crush capability with defined inputs and outputs. They can be invoked directly in Crush without choo-choo.

#### `design` Skill
**Purpose:** Transform rough ideas into structured requirements and design documents

**Invoked when:** User describes a feature, problem, or rough concept

**Expected state:**
- `specs/` directory exists (created if not)
- Optional: existing `requirements.md` to refine

**Outputs:**
- `specs/requirements.md` - Clarified requirements with Q&A
- `specs/design.md` - Technical design document

**Handoff hint:** "Ready for planning. Invoke the `plan` skill to generate tickets."

---

#### `plan` Skill
**Purpose:** Decompose design into hierarchical ticket structure

**Invoked when:** `specs/design.md` exists and user wants to break it into work items

**Expected state:**
- `specs/design.md` exists
- `.tickets/` directory (created if not)

**Outputs:**
- Ticket files in `.tickets/` with hierarchy (epic → stories → tasks)
- Dependencies established between tickets
- STATE.md updated with `phase: plan`, `epic: E-XXX`

**Handoff hint:** "Tickets created. Invoke the `validate` skill to verify the plan is executable."

---

#### `validate` Skill
**Purpose:** Verify ticket structure is sound and produce execution plan

**Invoked when:** Tickets exist and user wants to verify before execution

**Expected state:**
- `.tickets/` populated with tickets
- STATE.md with `epic` set

**Checks:**
- No dependency cycles
- All dependencies exist
- Tickets have descriptions
- Environment ready (skills, tk CLI)

**Outputs:**
- Validation report (printed to user)
- STATE.md updated with `phase: validate`
- Execution order (can be queried via `tk` or read from validation output)

**Handoff hint:** "Plan validated. Ready for execution. Invoke the `execute` skill to begin."

---

#### `execute` Skill
**Purpose:** Implement a single ticket

**Invoked when:** User wants to implement a specific ticket (or choo-choo iterates through tickets)

**Expected state:**
- Ticket file exists in `.tickets/`
- All dependencies completed (status: done)
- STATE.md with `focus` set to ticket ID

**Outputs:**
- Code changes in project
- Tests written and passing
- Ticket status updated to `done`
- STATE.md `focus` cleared

**Handoff hint:** "Ticket complete. Invoke the `verify` skill to confirm implementation matches intent."

---

#### `verify` Skill
**Purpose:** Confirm implementation matches ticket acceptance criteria

**Invoked when:** A ticket has been executed and needs verification

**Expected state:**
- Ticket with `accepts` field (acceptance criteria)
- Implementation code exists
- Tests exist

**Outputs:**
- Verification report (pass/fail per criterion)
- If gaps found: list of discrepancies
- STATE.md updated with `phase: verify`

**Handoff hint:** 
- On pass: "Verification passed. Ready for next ticket or done."
- On fail: "Gaps found. Invoke the `close-gaps` skill to fix."

---

#### `close-gaps` Skill
**Purpose:** Fix discrepancies between implementation and acceptance criteria

**Invoked when:** Verification found gaps

**Expected state:**
- Verification report with specific gaps
- Ticket ID in STATE.md `focus`

**Outputs:**
- Code fixes
- Updated tests
- Gap resolution notes

**Handoff hint:** "Gaps addressed. Invoke the `verify` skill to re-check."

---

#### Skill Independence Summary

| Skill | Can use without choo-choo? | choo-choo adds |
|-------|---------------------------|----------------|
| `design` | ✓ Interactive Crush chat | Two-pane UI, artifact preview |
| `plan` | ✓ `crush run "use plan skill"` | Progress visualization |
| `validate` | ✓ `crush run "use validate skill"` | Gate UI, report display |
| `execute` | ✓ Per-ticket in Crush | Kanban, parallel, popup details |
| `verify` | ✓ `crush run "verify ticket T-001"` | Status in kanban |
| `close-gaps` | ✓ `crush run "close gaps on T-001"` | Retry loop, escalation |

### E. Glossary

| Term | Definition |
|------|------------|
| Phase | Current stage in the workflow (design, plan, validate, execute, verify, close-gaps) |
| Gate | Human approval point between phases |
| Level | Group of tickets with no interdependencies, can execute in parallel |
| STATE.md | Session persistence file with phase, focus, learnings, decisions |
| tk CLI | Bash-based ticket management system |
| Skill | Agent capability defined in SKILL.md, loaded by Crush |
| Gap | Difference between expected and actual implementation behavior |
| Crush | Charm's terminal-based AI coding agent |
| bubbletea | Charm's TUI framework (Elm Architecture) |
