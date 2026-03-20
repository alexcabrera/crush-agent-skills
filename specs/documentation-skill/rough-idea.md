# Rough Idea

Create a documentation skill that runs at the end of the choo-choo workflow after all tickets are verified complete, but before reporting completion to the user.

## User's Initial Requirements

**Scope:** All documentation types
- API documentation
- CLI usage (help text, command reference)
- Architecture/decision records
- User guides/tutorials
- README updates

**Trigger:** After verify passes, before final completion report

**Output Structure:**
- Top-level `README.md` - Everything a human needs to use the software
- `docs/` directory - Power users and developers
- `README.md` in subdirectories as needed

**Philosophy:** READMEs are for users (complete usage guide), docs are for power users and devs
