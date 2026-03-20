---
name: document
description: Generates comprehensive README.md documentation for projects by analyzing code, specs, tests, and package metadata. Runs after each epic is verified. Produces single-file documentation representing the complete state and capabilities of the project. Use when an epic completes and documentation needs updating.
license: MIT
compatibility: Requires bash, git.
metadata:
  version: "1.0.0"
  author: choo-choo-skills
---

# Document Skill

Generate comprehensive, single-file documentation for projects. This skill analyzes code, specs, tests, and package metadata to produce a complete README.md that represents the current state and capabilities of the project.

Part of the [choo-choo-skills](../) collection.

## When to Use This Skill

| Scenario | Use |
|----------|-----|
| Epic completed, all tickets verified | This skill |
| README needs updating after implementation | This skill |
| Project documentation is stale | This skill |

### Invocation Triggers

Activate this skill when:
- All tickets in an epic have been verified
- Before reporting epic completion to user
- Documentation needs to reflect current code state

---

## Output

A single `README.md` in the project root with comprehensive documentation:

```markdown
# {Project Name}

{Tagline}

## Overview
## Installation
## Quick Start
## Usage Guide
## Deep Reference
## API Documentation
```

---

## The Document Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    DOCUMENT WORKFLOW                             │
│                                                                  │
│  1. ANALYZE project (code, specs, tests, package)              │
│         ↓                                                        │
│  2. CLASSIFY project type (library vs application)              │
│         ↓                                                        │
│  3. DETECT changes (git diff + tickets + full scan)            │
│         ↓                                                        │
│  4. GENERATE README.md from scratch                            │
│         ↓                                                        │
│  5. REPORT completion to user                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Analyze Project

Gather information from all available sources:

**Source Files:**
- Scan project structure (files, directories)
- Identify entry points (main file, exports)
- Extract public interfaces (functions, types, classes)

**Specs:**
- Read `specs/design.md` if present
- Extract context and design decisions

**Tests:**
- Parse test files for usage examples
- Infer usage patterns from test cases

**Package Metadata:**
- Read `package.json`, `Cargo.toml`, `pyproject.toml`, etc.
- Extract name, version, dependencies

---

## Step 2: Classify Project Type

Determine the project type to scale documentation appropriately:

| Project Type | Signals | API Docs Level |
|--------------|---------|----------------|
| CLI | `bin/` in package.json, main entry with command parsing | Minimal/None |
| Application | main.go, main.py, index.js as entry | Minimal/None |
| Library | exports in package.json, `[lib]` in Cargo.toml | Full |
| Service | HTTP handlers, server listeners | Minimal |

**Default:** Application (if unclear)

---

## Step 3: Detect Changes

Determine what needs documentation attention:

1. **Git diff** - Compare to last documented state
2. **Ticket scope** - Review completed tickets from epic
3. **Full scan** - Catch undocumented features or drift

Combine all three signals to understand what changed and what needs updating.

---

## Step 4: Generate README

**Always write from scratch** - regenerate fresh with current understanding.

### README Structure

```markdown
# {Project Name}

{One-line tagline describing what it does}

## Overview

{What it does, why it exists, key features}

## Installation

{How to install - platform specific if needed}

## Quick Start

{Minimal example to get running - copy-paste ready}

## Usage Guide

{Full feature walkthrough with examples}
{Organized by feature area or command}

## Deep Reference

{Detailed behavior, options, edge cases}
{Configuration options}
{Environment variables}

## API Documentation

{For libraries: Full API reference with signatures}
{For applications: Minimal or omitted}
```

### Section Guidelines

**Overview:**
- 2-3 paragraphs maximum
- Explain the "why" not just "what"
- List 3-5 key features

**Installation:**
- All supported platforms
- Prerequisites clearly stated
- Version requirements if applicable

**Quick Start:**
- Single example that demonstrates core value
- Should work in under 5 minutes
- No complex setup required

**Usage Guide:**
- Examples for each major feature
- Code blocks with expected output
- Common patterns and workflows

**Deep Reference:**
- All options/flags documented
- Edge cases and gotchas
- Performance considerations

**API Documentation (libraries only):**
- All public functions/types
- Signatures with parameter types
- Return types and error conditions
- Usage examples for each

---

## Step 5: Report Completion

After README.md is generated:
- Confirm documentation reflects current state
- Report to user that epic is complete
- README is ready for human review

---

## Integration with choo-choo Workflow

```
execute → verify → [all tickets verified?]
                          ↓ yes
                    document skill
                          ↓
                    README.md updated
                          ↓
                    Report epic completion
```

Position: After verify, before user notification.

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| No source files | Error - cannot document empty project |
| No package metadata | Infer from directory name, proceed |
| No specs/design.md | Proceed with code analysis only |
| No tests | Skip usage patterns from tests |
| Ambiguous project type | Default to "application" |
| Git not available | Skip git diff, use other signals |

---

## Package Metadata Sources

| Language | Files |
|----------|-------|
| JavaScript/TypeScript | package.json |
| Rust | Cargo.toml |
| Python | pyproject.toml, setup.py |
| Go | go.mod |
| Ruby | Gemfile, *.gemspec |

---

## File References

- [Analysis Procedures](references/analysis-procedures.md) - Detailed analysis methodology
- [Classification Procedures](references/classification-procedures.md) - Project type detection
- [README Templates](references/readme-templates.md) - Templates and examples
- [Change Detection](references/change-detection.md) - Change scope determination
- [Design Spec](../../specs/documentation-skill/design.md) - Full design specification
