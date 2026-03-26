# Change Detection Procedures

Procedures for determining what documentation needs updating based on code changes.

## Detection Strategy

Use three complementary approaches:

```
┌─────────────────────────────────────────────────────────────────┐
│                 CHANGE DETECTION STRATEGY                        │
│                                                                  │
│  1. GIT DIFF                                                    │
│     What files changed since last documentation?                │
│         ↓                                                        │
│  2. TICKET SCOPE                                                │
│     What tickets were completed in this epic?                   │
│         ↓                                                        │
│  3. FULL SCAN                                                   │
│     What's missing or drifted from current code?                │
│         ↓                                                        │
│  COMBINED SCOPE → Documentation Focus Areas                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Git Diff Analysis

### Last Documented State

Find the last point where documentation was updated:

```bash
# Find last commit that touched README.md
git log -1 --format="%H" -- README.md

# Or use a documentation tag if you tag releases
git describe --tags --abbrev=0
```

### Changed Files Analysis

```bash
# Files changed since last doc update
git diff --name-only <last-doc-commit> HEAD
```

**Categorize changes:**

| File Pattern | Documentation Impact |
|--------------|---------------------|
| `src/**/*.ts` | API docs, usage examples |
| `package.json` | Installation, dependencies |
| `README.md` | N/A (being regenerated) |
| `docs/**/*` | May inform structure |
| `tests/**/*` | Usage patterns, examples |
| `config/*` | Configuration section |

### Semantic Change Detection

Beyond file changes, understand what changed semantically:

**New Functions/Methods:**
```bash
# Find new exported functions (example for TypeScript)
git diff <last-doc-commit> HEAD -- '*.ts' | grep -E "^\+export (function|const|class)"
```

**Removed Functions:**
```bash
# Find removed exports
git diff <last-doc-commit> HEAD -- '*.ts' | grep -E "^\-export (function|const|class)"
```

**Changed Signatures:**
- Manual review of core modules
- Focus on public API changes

---

## 2. Ticket Scope Analysis

### Completed Tickets

Review what was done in this epic:

```bash
# List closed tickets in current epic
./scripts/ticket closed --limit=50
```

### Map Tickets to Documentation Areas

| Ticket Type | Documentation Focus |
|-------------|---------------------|
| New feature | Usage guide section |
| Bug fix | Deep reference (edge cases) |
| Refactor | Minimal (if behavior unchanged) |
| Breaking change | All sections affected |
| New CLI command | Usage guide, quick start |
| New API endpoint | API documentation |

### Extract Context from Tickets

For each completed ticket:
1. Read ticket description and acceptance criteria
2. Identify which README sections are affected
3. Note any new examples that should be documented
4. Flag breaking changes prominently

---

## 3. Full Scan for Drift

### Compare README to Code Reality

**Check for undocumented features:**
1. Parse exports from code
2. Compare to documented APIs
3. Flag any missing documentation

**Check for obsolete documentation:**
1. Parse README for mentioned features
2. Verify each exists in code
3. Flag any stale references

### Drift Detection Checklist

| Check | Method |
|-------|--------|
| All exports documented? | Compare exports to API docs |
| All CLI commands documented? | Compare --help to usage guide |
| Installation still works? | Test installation instructions |
| Examples still run? | Execute quick start example |
| Config options current? | Compare to actual config parsing |

---

## Combining Signals

### Priority Matrix

| Source | Signal Strength | Action |
|--------|-----------------|--------|
| Git diff: API file changed | High | Update API docs |
| Git diff: Config file changed | Medium | Update deep reference |
| Git diff: Test file changed | Low | Extract examples |
| Ticket: New feature | High | Add usage section |
| Ticket: Bug fix | Low | Update edge cases |
| Ticket: Breaking change | Critical | Update all sections |
| Full scan: Missing export | High | Add to API docs |
| Full scan: Obsolete doc | Medium | Remove or update |

### Scope Determination

**Full regeneration if:**
- Breaking changes introduced
- Major version bump
- Core API changed significantly
- README significantly outdated

**Targeted update if:**
- Minor feature additions
- Bug fixes only
- Small API additions
- Recent README (within last few commits)

---

## Output: Documentation Scope

Produce a structured scope for generation:

```
DocumentationScope {
  full_regen: boolean,
  sections: {
    overview: boolean,      // Update if: major changes, new features
    installation: boolean,  // Update if: package metadata changed
    quick_start: boolean,   // Update if: core flow changed
    usage_guide: [          // Sections to add/update
      { section: "feature-x", action: "add" },
      { section: "feature-y", action: "update" }
    ],
    deep_reference: boolean, // Update if: options/config changed
    api_docs: [             // APIs to add/update/remove
      { name: "newFunction", action: "add" },
      { name: "oldFunction", action: "remove" }
    ]
  },
  changes: {
    features_added: [...],
    features_removed: [...],
    breaking_changes: [...],
    config_changes: [...]
  }
}
```

---

## Fallback Behavior

When git history unavailable:
- Skip git diff analysis
- Rely on ticket scope + full scan

When no tickets available:
- Skip ticket scope analysis
- Rely on git diff + full scan

When both unavailable:
- Full scan only
- Document everything from current code state
