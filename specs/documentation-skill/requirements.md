# Requirements Clarification

## Q1: Change Detection

When the documentation skill runs, how should it determine what needs documentation?

**Decision:** Combination approach (no explicit input):
- Git diff based - Compare to last documented state
- Ticket scope - Document based on completed tickets
- Full scan - Catch undocumented legacy or drift

## Q2: Documentation Granularity

Should documentation happen:
- **Per ticket** - After each verified ticket (incremental)
- **Per epic** - After all tickets in an epic are verified
- **Per workflow** - Once at the very end, after all epics done

**Decision:** Per epic - runs after all tickets in an epic are verified, before moving to next epic

## Q3: Documentation Scope (Revised)

**Decision:** Single-file documentation - root README.md only

The skill updates the project's root README.md with comprehensive, single-file documentation that represents the complete state of the project:
- Project overview and purpose
- Installation and quick start
- Usage guide (full feature walkthrough)
- Deep reference section
- API documentation (at the end)

No docs/ directory, no separate files. README.md is the single source of truth.

## Q4: README Structure

**Decision:** Comprehensive single-file documentation about what the project does.

Structure:
1. **Project name & tagline** - What it is in one line
2. **Overview** - What it does, why it exists
3. **Installation** - How to get it
4. **Quick start** - Minimal example to get running
5. **Usage guide** - Full feature walkthrough with examples
6. **Deep reference** - Detailed behavior, options, edge cases
7. **API documentation** - At the end for developers/integrators

## Q5: Handling Existing READMEs

When a README.md already exists:
- **Full overwrite** - Replace entirely with fresh documentation
- **Preserve custom sections** - Keep any user-added sections, update standard ones
- **Merge** - Update existing content, add missing sections, remove obsolete ones

**Decision:** Always write from scratch - regenerate fresh with current understanding of the application

## Q6: Information Sources

Where should the skill gather information to write the README:
- **Code analysis** - Read source files, understand structure and behavior
- **specs/design.md** - Use existing design docs as context
- **Tests** - Infer usage patterns from test cases
- **Package files** - package.json, Cargo.toml, etc. for dependencies/metadata
- **All of the above** - Comprehensive scan

**Decision:** All of the above - comprehensive analysis from code, specs, tests, and package metadata

## Q7: API Documentation Detail Level

For the API documentation section at the end:
- **Full reference** - Every public function/type with signatures and descriptions
- **Key interfaces only** - Main entry points and important types
- **Examples-driven** - Show common patterns, link to code for details
- **Depends on project type** - Library = full, app = minimal/none

**Decision:** Depends on project type:
- Library projects → Full API reference (public functions, types, signatures)
- Application projects → Minimal or no API docs (focus on usage guide instead)

## Q8: Ready for Design?

Requirements captured:
1. ✅ Change detection: git diff + ticket scope + full scan
2. ✅ Granularity: Per epic
3. ✅ Scope: Single README.md, comprehensive documentation
4. ✅ Structure: 7-section format
5. ✅ Existing READMEs: Always write from scratch
6. ✅ Sources: Code, specs, tests, package files
7. ✅ API detail: Varies by project type

**Question:** Should I proceed to design.md, or are there other aspects to clarify?
