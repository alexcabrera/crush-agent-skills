# Analysis Procedures

Detailed procedures for gathering information from a project to generate documentation.

## Analysis Process

### 1. Scan Project Structure

```
project/
├── src/           → Source code location
├── tests/         → Test patterns
├── docs/          → Existing docs (for context)
├── specs/         → Design specs (for context)
└── package files  → Metadata
```

**Identify:**
- Root directory structure
- Source file locations
- Entry point candidates
- Test file locations
- Configuration files

### 2. Identify Entry Points

| Language | Entry Point Files |
|----------|-------------------|
| JavaScript | index.js, main.js, app.js, server.js |
| TypeScript | index.ts, main.ts, app.ts, server.ts |
| Python | __main__.py, main.py, app.py |
| Go | main.go |
| Rust | src/main.rs, src/lib.rs |
| Ruby | lib/*.rb, bin/* |

**Process:**
1. Check for `main` function/module
2. Check for `bin` field in package.json
3. Check for executable files in bin/
4. Look for HTTP server setup (for services)

### 3. Extract Public Interfaces

**For Libraries:**
- Exported functions with signatures
- Exported types/interfaces
- Public classes and methods
- Constants and configurations

**For Applications:**
- CLI commands and subcommands
- Configuration options
- Environment variables
- API endpoints (for services)

**Process:**
1. Parse import/export statements
2. Identify function signatures
3. Extract type definitions
4. Map command structure (for CLI)

### 4. Parse Tests for Examples

**What to extract:**
- Import statements (shows API surface)
- Function calls (shows usage patterns)
- Assertions (shows expected behavior)
- Setup/teardown (shows initialization)

**Example extraction:**
```python
# From test file
def test_user_creation():
    user = User(name="Alice", email="alice@example.com")
    assert user.name == "Alice"
```

**Becomes README example:**
```python
# Create a user
user = User(name="Alice", email="alice@example.com")
```

### 5. Read Package Metadata

#### JavaScript/TypeScript (package.json)

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "description": "Package description",
  "main": "dist/index.js",
  "bin": { "my-cli": "./bin/cli.js" },
  "scripts": { "start": "node server.js" },
  "dependencies": { ... },
  "peerDependencies": { ... }
}
```

Extract:
- name → Project name
- description → Tagline/overview
- bin → CLI commands
- scripts.start → Application entry
- dependencies → Installation requirements

#### Python (pyproject.toml)

```toml
[project]
name = "my-package"
version = "1.0.0"
description = "Package description"

[project.scripts]
my-cli = "my_package.cli:main"
```

Extract:
- name → Project name
- description → Tagline
- project.scripts → CLI commands

#### Go (go.mod)

```
module github.com/user/my-package

go 1.21

require (
    github.com/some/dependency v1.0.0
)
```

Extract:
- module → Project name (last part)
- require → Dependencies

#### Rust (Cargo.toml)

```toml
[package]
name = "my-package"
version = "1.0.0"
description = "Package description"

[[bin]]
name = "my-cli"
path = "src/main.rs"
```

Extract:
- name → Project name
- description → Tagline
- [[bin]] → CLI commands

### 6. Read Design Specs

If `specs/design.md` exists:

**Extract:**
- Overview section → README overview
- Requirements → Feature list
- Architecture diagram → May inform structure
- Key decisions → May inform deep reference

**Process:**
1. Read design.md
2. Extract non-technical descriptions
3. Adapt technical details for user audience
4. Use as context for accuracy

### 7. Synthesize Findings

Combine all sources into structured understanding:

```
ProjectAnalysis {
  name: from package or directory
  type: library | application | cli | service
  tagline: from description or inferred
  features: from code + specs
  dependencies: from package metadata
  examples: from tests
  entry_points: from structure analysis
}
```

This analysis feeds directly into classification and generation.
