# Agent Skills Directory

This directory contains various agent skills that are being maintained and developed by the user.

## Agent Skills Specification

This repository includes a clone of the official [Agent Skills specification](https://agentskills.io) in `.read-only/agentskills/`. All skills in this directory must adhere to this specification.

## Key Requirements for Agents

When working in this directory, agents must ensure their work always follows the Agent Skills specification as defined in `.read-only/agentskills/docs/specification.mdx`. This includes:

### Skill Structure
- Each skill must be a directory containing a `SKILL.md` file with YAML frontmatter
- The `name` field in frontmatter must match the parent directory name
- Name must be 1-64 characters, lowercase letters/numbers/hyphens only, no leading/trailing hyphens
- Description field (required, 1-1024 chars) must describe both what the skill does and when to use it

### Optional Directories
- `scripts/` - Executable code that agents can run
- `references/` - Additional documentation loaded on demand
- `assets/` - Static resources like templates and data files

### Progressive Disclosure
- Keep `SKILL.md` under 500 lines and 5000 tokens when possible
- Move detailed reference material to separate files
- Use relative paths when referencing files within a skill

### Validation
Before creating or modifying skills, verify compliance with the specification. The specification defines the complete format requirements for all skill components.

### Reference
- Complete specification: `.read-only/agentskills/docs/specification.mdx`
- Project README: `.read-only/agentskills/README.md`
- Additional docs: `.read-only/agentskills/docs/`
