# Implementation Plan

## Progress
- [ ] Step 1: Create skill structure and SKILL.md
- [ ] Step 2: Implement analysis component
- [ ] Step 3: Implement classification component
- [ ] Step 4: Implement generation component with template
- [ ] Step 5: Implement change detection component
- [ ] Step 6: Integrate with choo-choo workflow

---

## Step 1: Create Skill Structure

**Objective:** Set up the document skill directory and SKILL.md

**Implementation:**
- Create `choo-choo-skills/document/` directory
- Create `SKILL.md` with frontmatter and skill documentation
- Add to main choo-choo-skills/SKILL.md skills table
- Add to AGENTS.md skills table

**Tests Required:**
- SKILL.md follows specification (name matches directory, valid frontmatter)
- Skill appears in choo-choo workflow documentation

**Integration:**
- Skill discoverable by Crush
- Listed in choo-choo meta skill

**Demo:**
- `ls choo-choo-skills/document/` shows SKILL.md
- Crush can reference the document skill

---

## Step 2: Implement Analysis Component

**Objective:** Gather information from code, specs, tests, and package files

**Implementation:**
- Define analysis process in SKILL.md
- Document how to scan project structure
- Document how to extract entry points and public interfaces
- Document how to parse tests for usage examples
- Document how to read package metadata

**Tests Required:**
- Analysis produces consistent project understanding
- Handles missing files gracefully

**Integration:**
- Analysis results feed into classification and generation

**Demo:**
- Given sample project, analysis extracts key information

---

## Step 3: Implement Classification Component

**Objective:** Determine project type (library vs application) and API doc level

**Implementation:**
- Document classification heuristics in SKILL.md or reference
- Define rules for library/application/CLI/service detection
- Define API documentation scaling rules

**Tests Required:**
- Library projects classified as library
- CLI projects classified as CLI
- Unknown types default to application

**Integration:**
- Classification determines README template emphasis

**Demo:**
- Sample library → full API docs
- Sample CLI → usage guide focus, minimal API

---

## Step 4: Implement Generation Component

**Objective:** Produce README.md from analysis and classification

**Implementation:**
- Define README template structure
- Document how to populate each section
- Document overwrite behavior (always from scratch)

**Tests Required:**
- Generated README has all 7 sections
- Section content is accurate to project state

**Integration:**
- Reads from analysis and classification components
- Writes to project root README.md

**Demo:**
- Given analyzed project, produces complete README.md

---

## Step 5: Implement Change Detection Component

**Objective:** Determine scope of documentation changes

**Implementation:**
- Document git diff analysis process
- Document ticket scope analysis
- Document full scan approach
- Define how to combine all three signals

**Tests Required:**
- Detects changes from git history
- Correlates with completed tickets
- Catches undocumented features via full scan

**Integration:**
- Feeds scope information to generation component

**Demo:**
- Changed files → focused documentation attention
- New features → added to README

---

## Step 6: Integrate with choo-choo Workflow

**Objective:** Connect document skill to workflow

**Implementation:**
- Update choo-choo workflow diagram to include document phase
- Update verify skill to trigger document after epic verified
- Update STATE.md structure if needed
- Document handoff from verify → document → user report

**Tests Required:**
- After epic verified, document skill runs
- README.md generated before user sees completion report

**Integration:**
- Part of standard choo-choo workflow

**Demo:**
- Complete epic → verify → document → README updated → report to user
