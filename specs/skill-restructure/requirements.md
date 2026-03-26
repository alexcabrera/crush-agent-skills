# Requirements

## Rough Idea

restructure agent-skills repository for spec compliance and script-based transitions

---

## Clarification Q&A

### Q1: What problems are we solving?
**A:** 
1. "choo-choo" name is too clever, not discoverable by agents
2. Skills are tightly coupled with relative path references
3. "design" skill name conflicts with visual design interpretation
4. Scripts are buried inside skills, not easily reusable
5. No formal mechanism for skill-to-skill transitions
6. Single "test" skill doesn't handle CLI vs Web testing differences
7. No autonomous execution mode for hands-off development cycles
8. No bootstrap mechanism for system dependencies

### Q2: What is the primary purpose of this repository?
**A:** 
Central repository for maintaining agent skills. Skills will be symlinked or referenced from actual project locations. Users will never copy someone else's skills directly - they will always adapt to their system.

### Q3: What naming should replace "design"?
**A:** `elaborate` - captures iterative clarification + architecture process

### Q4: How should skill transitions work?
**A:**
Each skill has a `scripts/next` executable that:
- Outputs the next skill to invoke
- Provides context/arguments
- Returns exit codes indicating workflow state (continue/complete/blocked)

### Q5: What directory structure is spec-compliant?
**A:**
Skills at repository root for symlink compatibility:
```
~/Code/agent-skills/           # symlink to ~/.config/crush/skills/
├── elaborate/SKILL.md
├── plan/SKILL.md
├── execute/SKILL.md           # Autonomous virtuous cycle
├── implement/SKILL.md         # Single ticket implementation
├── ...
```

### Q6: Meta-skill or next scripts only?
**A:** No meta-skill. Each skill is independently invocable. `next` scripts handle handoffs.

### Q7: Where should shared resources live?
**A:** `shared/` directory at repo root with relative path references (`../shared/STATE-structure.md`). Pattern used by Microsoft EF Core and others.

### Q8: Backward compatibility?
**A:** Clean break. Kill choo-choo name. Update symlink at `~/.config/crush/skills/choo-choo-skills` → point to restructured repo root.

### Q9: How should testing work?
**A:** Split into two skills:
- `test-cli` - CLI/TUI testing via tmux
- `test-web` - Web testing via agent-browser (Vercel)

Skill detection in `execute` determines which to use based on project type.

### Q10: How should autonomous execution work?
**A:** Rename current `execute` → `implement` (single ticket). New `execute` skill runs autonomous virtuous cycle:
- Pick ready ticket
- Research if needed
- Implement (TDD)
- Test (cli or web based on detection)
- Verify against acceptance criteria
- Fix gaps → loop
- Refactor if needed
- Integrate full suite
- Report progress

### Q11: How should skills handle system dependencies?
**A:** Each skill has a `scripts/bootstrap` executable that:
- Checks for required tools
- Installs missing dependencies with user consent
- Idempotent (safe to run multiple times)
- Outputs what was installed or what's missing

---

## Summary

Restructure the agent-skills repository to:
1. Use repo-root directory layout for symlink compatibility
2. Rename skills for agent discoverability (flatten, rename "design" → "elaborate")
3. Add `next` scripts to each skill for executable-based transitions
4. Add `bootstrap` scripts to each skill for dependency installation
5. Split testing into `test-cli` (tmux) and `test-web` (agent-browser)
6. Rename single-ticket skill to `implement`, add `execute` for autonomous cycle
7. Use `shared/` directory for cross-skill resources
8. Update crush symlink configuration

---

## Open Questions

*All resolved.*

---

## Decisions Made

| Decision | Rationale |
|----------|-----------|
| Skills at repo root | Symlink entire repo to `~/.config/crush/skills/` |
| `elaborate` naming | Avoids visual design confusion, captures process |
| `next` scripts | Decoupled, discoverable, executable transitions |
| `bootstrap` scripts | Self-sufficient skills, easy first-time setup |
| `shared/` directory | DRY, used in wild, easy migration path |
| Clean break | Personal repo, no backward compatibility needed |
| `test-cli` + `test-web` | Different tools (tmux vs agent-browser), different patterns |
| `execute` = autonomous | "Execute the plan" semantics for hands-off mode |
| `implement` = single ticket | Clear scope, one ticket at a time |
| agent-browser over Playwright | Purpose-built for AI agents, ref-based, no Node.js required |
