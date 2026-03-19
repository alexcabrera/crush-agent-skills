# Terminal UI Best Practices

Comprehensive guide for designing Command-Line Interfaces (CLI) and Text-Based User Interfaces (TUI). Focuses on interaction patterns, usability heuristics, and human factors rather than implementation details.

---

## Core Principles

| Principle | Description |
|-----------|-------------|
| **Consistency** | Predictable naming, flags, and behavior across commands |
| **Feedback** | Immediate, clear, actionable responses to user actions |
| **Composability** | CLI tools that work well with pipes and scripts |
| **Accessibility** | Usable by people with disabilities |
| **Discoverability** | Easy to learn and explore without documentation |

---

## Command Structure

### Naming Conventions

| Pattern | Example | Use When |
|---------|---------|----------|
| Verb-noun | `docker run`, `git commit` | Action-oriented tools |
| Noun-verb | `kubectl get pods` | Resource-oriented tools |
| Hierarchical | `git remote add` | Complex tool with subcategories |

**Guidelines:**
- Use descriptive, unambiguous names reflecting action and object
- Avoid cryptic abbreviations unless widely adopted (`-v` for verbose)
- Keep hierarchies shallow (2-3 levels max)

### Flag Conventions

| Flag | Purpose |
|------|---------|
| `-h`, `--help` | Display usage information |
| `-v`, `--verbose` | Increase output detail |
| `-q`, `--quiet` | Suppress non-essential output |
| `--version` | Display version information |
| `--dry-run` | Show what would happen without executing |
| `--no-color` | Disable colored output |
| `-y`, `--yes` | Skip confirmation prompts |

**Guidelines:**
- Support both short and long forms for common flags
- Place frequently used flags early in help output
- Use `--no-X` pattern for boolean flags that default to true

### Arguments

| Type | Example | Description |
|------|---------|-------------|
| Positional | `git add <file>` | Order matters, no flag needed |
| Required | `docker run <image>` | Must be provided |
| Optional | `git log [path]` | Can be omitted |
| Variadic | `git add <files>...` | Accepts multiple values |

---

## User Feedback

### Progress Indicators

| Type | Use Case |
|------|----------|
| Spinner | Unknown duration, active state |
| Progress bar | Known progress (percentage) |
| Counter | Items processed (e.g., "3/10 files") |
| Status line | Ongoing operation description |

**Example patterns:**
```
⠋ Installing dependencies...
[████████░░░░░░░░] 53% Building bundles
Copying files: 45/120
```

### Success Confirmation

Brief, affirmative messages for completed operations:

```
✓ Created 3 files
✓ Pushed 5 commits to origin/main
Deployment complete: https://app.example.com
```

### Verbosity Levels

| Level | Flag | Output |
|-------|------|--------|
| Quiet | `-q` | Errors only |
| Default | (none) | Essential information |
| Verbose | `-v` | Additional details |
| Debug | `-vv` or `--debug` | Full diagnostics |

---

## Error Handling

### Error Message Structure

```
<program>: <context>: <error>

<explanation>
<suggestion>
```

**Example:**
```
git: push: rejected (non-fast-forward)

The remote has commits you don't have locally.
Run 'git pull' to merge remote changes before pushing.
```

### Error Principles

| Principle | Guidelines |
|-----------|------------|
| **Visibility** | Display errors prominently, use color/formatting |
| **Clarity** | Plain language, no jargon or raw stack traces |
| **Constructive** | Suggest specific recovery steps |
| **Prevention** | Validate input early, preserve user input |

### Common Error Patterns

| Error Type | Bad Example | Good Example |
|------------|-------------|--------------|
| File not found | `Error: ENOENT` | `Error: File 'config.yaml' not found. Create it with 'init --config'` |
| Permission denied | `EACCES` | `Error: Permission denied writing to '/usr/bin'. Try with sudo or use '--prefix ~/.local'` |
| Invalid input | `Invalid argument` | `Error: Port must be 1-65535, got 'abc'. Use a number like '3000'` |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Misuse of command |
| 64-69 | Custom errors (reserved) |
| 126 | Command not executable |
| 127 | Command not found |
| 130 | Interrupted (Ctrl+C) |

---

## TUI Navigation

### Input Models

| Model | Description | Examples |
|-------|-------------|----------|
| Modal | Different modes for different tasks | vim (normal/insert) |
| Menu-driven | Arrow keys to navigate options | lazygit, htop |
| Command-driven | Type commands directly | tmux (prefix mode) |
| Hybrid | Combination of above | lazygit |

### Keyboard Shortcuts

**Standard bindings:**
| Key | Action |
|-----|--------|
| `q`, `Esc` | Quit / Exit / Cancel |
| `?` | Help |
| `Enter` | Confirm / Select |
| `Tab`, `Shift+Tab` | Navigate forward/backward |
| `↑` `↓` `←` `→` | Navigate / Scroll |
| `/` | Search |
| `Ctrl+C` | Cancel / Interrupt |
| `Ctrl+D` | Exit / End of input |

**Mnemonic shortcuts:**
- Use first letter of action when possible (`j` for jump, `f` for find)
- Group related actions (navigation: `h/j/k/l` or `j/k` for up/down)
- Avoid destructive actions on single keys

### Visual Hierarchy

```
┌─────────────────────────────────────────────────┐
│ Title / Header                        [status]  │  ← Top bar
├─────────────────────────────────────────────────┤
│                                                 │
│  Primary content area                           │  ← Main region
│  (list, editor, data display)                   │
│                                                 │
├─────────────────────────────────────────────────┤
│  Secondary content / details                    │  ← Detail pane
├─────────────────────────────────────────────────┤
│  [a]ction  [b]ack  [?]help  [q]uit              │  ← Footer/hints
└─────────────────────────────────────────────────┘
```

---

## Accessibility

### Color and Contrast

| Requirement | Guideline |
|-------------|-----------|
| Contrast ratio | Minimum 4.5:1 for text (WCAG AA) |
| Color blindness | Don't rely solely on color; use symbols |
| Dark/light support | Respect terminal theme or provide options |

**Color usage:**
- Red: Errors, destructive actions
- Yellow/Orange: Warnings
- Green: Success, safe actions
- Blue: Information, links
- Cyan: Highlights

**Fallback indicators:**
```
✓ Success    (green checkmark)
✗ Error      (red X)
⚠ Warning    (yellow triangle)
ℹ Info       (blue info circle)
```

### Screen Reader Compatibility

| Practice | Description |
|----------|-------------|
| Semantic structure | Use consistent formatting patterns |
| ASCII only fallback | Avoid box-drawing for essential info |
| Plain text mode | `--no-color` or `NO_COLOR=1` |
| Structured output | JSON/YAML for programmatic consumption |

### Alternative Input

- All actions achievable via keyboard (no mouse required)
- Configuration via environment variables
- Non-interactive mode with flags (`--yes`, `--no-input`)

---

## Help and Documentation

### Help Structure

**Short help (`-h`):**
```
Usage: myapp <command> [options]

Commands:
  build    Build the project
  deploy   Deploy to environment
  logs     View application logs

Run 'myapp <command> --help' for command details.
```

**Long help (`--help`):**
```
MYAPP(1)                        User Commands                       MYAPP(1)

NAME
    myapp - Application deployment tool

SYNOPSIS
    myapp <command> [options]

DESCRIPTION
    myapp manages the full lifecycle of application deployment...

COMMANDS
    build [options]
        Build the project...

OPTIONS
    -v, --verbose
        Increase output verbosity...

EXAMPLES
    myapp build --production
    myapp deploy --env staging

SEE ALSO
    Full documentation: https://docs.example.com/myapp
```

### Contextual Help

| Type | When to Use |
|------|-------------|
| Inline hints | Show available actions in TUI footer |
| Tooltips | Display on hover/focus (TUI) |
| Wizards | Guide first-time users through setup |
| Examples | Include common usage patterns in help |

---

## Interaction Patterns

### Confirmations

**Destructive actions:**
```
⚠ This will delete 5 files. Continue? [y/N]
```

**Bulk operations:**
```
This affects 127 files. Continue? [y/N/a/s]
  y = yes  N = no  a = yes to all  s = show files
```

### Prompts

**Required input:**
```
Enter your name: _
```

**With default:**
```
Port [3000]: _
```

**With validation:**
```
Email (must be @company.com): foo@_   ← Invalid, show error
```

### Selection

**Single select:**
```
Choose environment:
  ❯ production
    staging
    development
```

**Multi-select:**
```
Select features: (space to select, enter to confirm)
  ◉ authentication
  ◯ database
  ◉ caching
```

---

## Real-World Examples

### Git (CLI)

**Strengths:**
- Hierarchical subcommands with consistent patterns
- Detailed error messages with suggestions
- Layered help (usage → man pages → web docs)
- Progress feedback during operations

**Patterns:**
```
git <command> [options] [--] [paths]
git status              # Show state
git add -p              # Interactive staging
git commit --amend      # Modify last commit
```

### Docker (CLI)

**Strengths:**
- Verb-noun command structure
- Progress bars for pulls/builds
- Clear error messages with context

**Patterns:**
```
docker <verb> <noun> [options]
docker run -d -p 80:80 nginx
docker build -t myapp .
```

### LazyGit (TUI)

**Strengths:**
- Multi-pane layout showing context
- Keyboard-driven with hints
- Visual feedback for all actions
- Confirmation for destructive operations

**Layout:**
```
┌─Status────────┬─Files───────────┐
│ On branch...  │ M file1.txt     │
│               │ A file2.txt     │
├───────────────┴─────────────────┤
│ [c]ommit [P]ush [p]ull [q]uit   │
└─────────────────────────────────┘
```

### htop (TUI)

**Strengths:**
- Color-coded metrics for quick scanning
- Keyboard shortcuts for all actions
- Real-time updates
- Customizable display

---

## Design Checklist

Use this checklist when designing terminal interfaces:

### Command Structure
- [ ] Consistent naming convention
- [ ] Logical subcommand hierarchy
- [ ] Standard flags (`-h`, `-v`, `--version`)
- [ ] Short and long flag forms
- [ ] Clear positional vs optional arguments

### User Feedback
- [ ] Progress indicators for long operations
- [ ] Success confirmations for important actions
- [ ] Verbosity levels supported
- [ ] Color used strategically (with fallback)

### Error Handling
- [ ] Visible, prominent error display
- [ ] Plain language explanations
- [ ] Specific recovery suggestions
- [ ] Input validation with clear messages
- [ ] Appropriate exit codes

### Navigation (TUI)
- [ ] Keyboard-only operation possible
- [ ] Consistent shortcuts across screens
- [ ] Clear mode/context indicators
- [ ] Always-visible navigation hints

### Accessibility
- [ ] High contrast support
- [ ] Color-blind friendly (symbols + color)
- [ ] Screen reader compatible output mode
- [ ] Non-interactive mode available

### Documentation
- [ ] Concise `-h` output
- [ ] Detailed `--help` output
- [ ] Examples in help text
- [ ] Contextual hints in TUI

---

## References

- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [Microsoft Command-Line Syntax](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/command-line-syntax)
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [12 Factor CLI Apps](https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46)
- [Heroku CLI Design](https://devcenter.heroku.com/articles/cli-design)
