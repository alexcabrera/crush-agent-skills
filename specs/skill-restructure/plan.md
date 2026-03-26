# Implementation Plan - Verified

## Summary
- **10 skills** to create (8 migrated, 2 new)
- **15 reference files** to migrate
- **3 script files** to migrate
- **1 shared file** to migrate
- **All files accounted for**

---

## Progress
- [ ] Step 1: Create new directory structure
- [ ] Step 2: Create shared/ directory with templates
- [ ] Step 3: Create test-cli skill (migrate from test)
- [ ] Step 4: Create test-web skill (new)
- [ ] Step 5: Migrate elaborate skill (from design)
- [ ] Step 6: Migrate plan skill
- [ ] Step 7: Migrate implement skill (rename from execute)
- [ ] Step 8: Create execute skill (new autonomous cycle)
- [ ] Step 9: Migrate verify skill
- [ ] Step 10: Migrate close-gaps skill
- [ ] Step 11: Migrate document skill
- [ ] Step 12: Migrate ui-design skill
- [ ] Step 13: Update AGENTS.md and README.md
- [ ] Step 14: Remove old choo-choo-skills directory
- [ ] Step 15: Update crush symlink
- [ ] Step 16: Validate skill discovery and bootstrap

---

## File Mappings (Verified)

### Skills (SKILL.md)

| Source | Destination | Notes |
|--------|-------------|-------|
| `choo-choo-skills/SKILL.md` | *(delete)* | Meta skill, not needed |
| `choo-choo-skills/design/SKILL.md` | `elaborate/SKILL.md` | Rename skill |
| `choo-choo-skills/plan/SKILL.md` | `plan/SKILL.md` | Update references |
| `choo-choo-skills/execute/SKILL.md` | `implement/SKILL.md` | Rename skill |
| *(new)* | `execute/SKILL.md` | New autonomous cycle skill |
| `choo-choo-skills/verify/SKILL.md` | `verify/SKILL.md` | Update references |
| `choo-choo-skills/close-gaps/SKILL.md` | `close-gaps/SKILL.md` | Update references |
| `choo-choo-skills/document/SKILL.md` | `document/SKILL.md` | Update references |
| `choo-choo-skills/test/SKILL.md` | `test-cli/SKILL.md` | Rename skill |
| *(new)* | `test-web/SKILL.md` | New web testing skill |
| `choo-choo-skills/ui-design/SKILL.md` | `ui-design/SKILL.md` | Update references |
| `choo-choo-skills/validate/SKILL.md` | *(delete or merge into plan)* | Optional, may not be needed |

### Scripts

| Source | Destination |
|--------|-------------|
| `choo-choo-skills/plan/scripts/ticket` | `plan/scripts/ticket` |
| `choo-choo-skills/test/scripts/test` | `test-cli/scripts/test` |
| `choo-choo-skills/design/scripts/init-spec` | `elaborate/scripts/init-spec` |
| *(new)* | `*/scripts/next` (all skills) |
| *(new)* | `*/scripts/bootstrap` (all skills) |

### References

| Source | Destination |
|--------|-------------|
| `choo-choo-skills/design/references/*.md` | `elaborate/references/` |
| `choo-choo-skills/plan/references/*.md` | `plan/references/` |
| `choo-choo-skills/document/references/*.md` | `document/references/` |
| `choo-choo-skills/test/references/*.md` | `test-cli/references/` |
| `choo-choo-skills/ui-design/references/*.md` | `ui-design/references/` |

### Shared Files

| Source | Destination |
|--------|-------------|
| `choo-choo-skills/shared/STATE-structure.md` | `shared/STATE-structure.md` |
| *(new)* | `shared/workflow.md` |
| *(new)* | `shared/bootstrap-template` |

---

## Validation Checkpoint

Before proceeding to implementation, verify:

1. [ ] All source files exist
2. [ ] No destination files exist (no overwrites)
3. [ ] Plan covers all files
4. [ ] New skill requirements are defined

---

## Step 1: Create New Directory Structure

**Objective:** Create all skill directories with required subdirectories

```bash
mkdir -p elaborate/{scripts,references}
mkdir -p plan/{scripts,references}
mkdir -p execute/{scripts,references}
mkdir -p implement/{scripts,references}
mkdir -p verify/{scripts,references}
mkdir -p close-gaps/{scripts,references}
mkdir -p document/{scripts,references}
mkdir -p test-cli/{scripts,references}
mkdir -p test-web/{scripts,references}
mkdir -p ui-design/{scripts,references}
mkdir -p shared
```

**Validation:**
```bash
ls -d elaborate plan execute implement verify close-gaps document test-cli test-web ui-design shared
```

---

## Step 2: Create shared/ Directory

**Objective:** Create shared resources

1. Copy STATE-structure.md
2. Create workflow.md
3. Create bootstrap-template

**Validation:**
```bash
ls shared/
```

---

## Step 3: Create test-cli Skill

**Objective:** Migrate test skill to test-cli with tmux focus

1. Copy `choo-choo-skills/test/SKILL.md` → `test-cli/SKILL.md`
2. Copy `choo-choo-skills/test/references/` → `test-cli/references/`
3. Copy `choo-choo-skills/test/scripts/test` → `test-cli/scripts/test`
4. Update SKILL.md:
   - Change `name: test` → `name: test-cli`
   - Update description to mention CLI/TUI focus
   - Update references to other skills
5. Create `test-cli/scripts/next`
6. Create `test-cli/scripts/bootstrap`

**Validation:**
```bash
./test-cli/scripts/test help
./test-cli/scripts/bootstrap --check
```

---

## Step 4: Create test-web Skill (New)

**Objective:** Create new skill for web testing with agent-browser

1. Create `test-web/SKILL.md`:
   - `name: test-web`
   - Description focused on web applications
   - References agent-browser
   - Same test patterns as test-cli but for web
2. Create `test-web/references/web-testing-patterns.md`
3. Create `test-web/scripts/browser` (wrapper for agent-browser)
4. Create `test-web/scripts/next`
5. Create `test-web/scripts/bootstrap`:
   - Check for agent-browser
   - Check for Chrome/Chromium
   - Install instructions

**Validation:**
```bash
./test-web/scripts/bootstrap --check
```

---

## Step 5: Migrate elaborate Skill

**Objective:** Migrate design → elaborate

1. Copy `choo-choo-skills/design/SKILL.md` → `elaborate/SKILL.md`
2. Copy `choo-choo-skills/design/references/` → `elaborate/references/`
3. Copy `choo-choo-skills/design/scripts/init-spec` → `elaborate/scripts/init-spec`
4. Update SKILL.md:
   - `name: design` → `name: elaborate`
   - Update description
   - Replace `[choo-choo-skills](../)` → references to `../shared/`
   - Replace `[ui-design](../ui-design/)` → `[ui-design](../ui-design/)`
   - Add handoff section referencing `scripts/next`
5. Create `elaborate/scripts/next`
6. Create `elaborate/scripts/bootstrap`

**Validation:**
```bash
grep "name: elaborate" elaborate/SKILL.md
./elaborate/scripts/init-spec --help
./elaborate/scripts/bootstrap --check
```

---

## Step 6: Migrate plan Skill

**Objective:** Migrate plan skill with ticket script

1. Copy `choo-choo-skills/plan/SKILL.md` → `plan/SKILL.md`
2. Copy `choo-choo-skills/plan/references/` → `plan/references/`
3. Copy `choo-choo-skills/plan/scripts/ticket` → `plan/scripts/ticket`
4. Update SKILL.md:
   - `author: choo-choo-skills` → remove or update
   - Replace `[choo-choo-skills](../)` → `../shared/`
   - Replace `[design](../design/)` → `[elaborate](../elaborate/)`
   - Replace `[test](../test/)` → `[test-cli](../test-cli/)` and `[test-web](../test-web/)`
5. Create `plan/scripts/next`
6. Create `plan/scripts/bootstrap`

**Validation:**
```bash
./plan/scripts/ticket help
./plan/scripts/bootstrap --check
```

---

## Step 7: Migrate implement Skill

**Objective:** Rename execute → implement

1. Copy `choo-choo-skills/execute/SKILL.md` → `implement/SKILL.md`
2. Update SKILL.md:
   - `name: execute` → `name: implement`
   - Update description to "single ticket implementation"
   - Replace references to other skills
   - Note that this is for single ticket, not the full cycle
3. Create `implement/scripts/next`
4. Create `implement/scripts/bootstrap`

**Validation:**
```bash
grep "name: implement" implement/SKILL.md
./implement/scripts/bootstrap --check
```

---

## Step 8: Create execute Skill (New)

**Objective:** Create autonomous virtuous cycle skill

1. Create `execute/SKILL.md`:
   - `name: execute`
   - Description: "Execute a plan autonomously. Runs the full virtuous cycle..."
   - Document the cycle: pick → research → implement → test → verify → fix → repeat
   - Document app type detection (CLI vs Web)
   - Document escalation conditions
   - Document output files (STATUS.md, STATE.md)
2. Create `execute/references/`:
   - `app-detection.md` - How to detect CLI vs Web
   - `virtuous-cycle.md` - Detailed cycle documentation
3. Create `execute/scripts/next`
4. Create `execute/scripts/bootstrap`

**Validation:**
```bash
grep "name: execute" execute/SKILL.md
./execute/scripts/bootstrap --check
```

---

## Step 9: Migrate verify Skill

**Objective:** Migrate verify skill

1. Copy `choo-choo-skills/verify/SKILL.md` → `verify/SKILL.md`
2. Update references
3. Create `verify/scripts/next`
4. Create `verify/scripts/bootstrap`

**Validation:**
```bash
./verify/scripts/bootstrap --check
```

---

## Step 10: Migrate close-gaps Skill

**Objective:** Migrate close-gaps skill

1. Copy `choo-choo-skills/close-gaps/SKILL.md` → `close-gaps/SKILL.md`
2. Update references
3. Create `close-gaps/scripts/next`
4. Create `close-gaps/scripts/bootstrap`

---

## Step 11: Migrate document Skill

**Objective:** Migrate document skill with references

1. Copy `choo-choo-skills/document/SKILL.md` → `document/SKILL.md`
2. Copy `choo-choo-skills/document/references/` → `document/references/`
3. Update references
4. Create `document/scripts/next`
5. Create `document/scripts/bootstrap`

---

## Step 12: Migrate ui-design Skill

**Objective:** Migrate ui-design skill

1. Copy `choo-choo-skills/ui-design/SKILL.md` → `ui-design/SKILL.md`
2. Copy `choo-choo-skills/ui-design/references/` → `ui-design/references/`
3. Update references to `../elaborate/`
4. Create `ui-design/scripts/next`
5. Create `ui-design/scripts/bootstrap`

---

## Step 13: Update AGENTS.md and README.md

**Objective:** Update repository documentation

1. Rewrite AGENTS.md:
   - Document new skill structure
   - Explain symlink usage
   - List all 10 skills with descriptions
   - Document bootstrap workflow
   - Document next script protocol
2. Update README.md:
   - Repository purpose
   - Skill list table
   - Installation instructions
   - Quick start guide

---

## Step 14: Remove Old Directory

**Objective:** Clean up

```bash
rm -rf choo-choo-skills/
```

**Verify:**
```bash
ls choo-choo-skills 2>/dev/null || echo "Removed successfully"
```

---

## Step 15: Update Crush Symlink

**Objective:** Update symlink configuration

```bash
rm ~/.config/crush/skills/choo-choo-skills
ln -s ~/Code/agent-skills ~/.config/crush/skills/agent-skills
```

**Verify:**
```bash
ls -la ~/.config/crush/skills/
```

---

## Step 16: Validate Skill Discovery and Bootstrap

**Objective:** Final validation

1. Start Crush in agent-skills repo
2. Verify all 10 skills appear
3. Test bootstrap scripts:

```bash
for skill in elaborate plan execute implement verify close-gaps document test-cli test-web ui-design; do
    echo "Testing $skill..."
    ./$skill/scripts/bootstrap --check || echo "  Missing dependencies"
done
```

**Success Criteria:**
- [ ] All 10 skills discovered by Crush
- [ ] All SKILL.md frontmatter valid
- [ ] All bootstrap scripts executable
- [ ] All next scripts executable
- [ ] No validation errors

---

## Post-Implementation Verification

After all steps complete:

1. Test full workflow:
   - Start with `elaborate` skill
   - Run through to `plan`
   - Hand off to `execute`
   - Verify virtuous cycle works

2. Test app detection:
   - Create mock CLI project → verify detects CLI
   - Create mock Web project → verify detects Web

3. Test bootstrap:
   - Fresh environment → run bootstrap → verify tools installed
