# Classification Procedures

Detailed heuristics for determining project type and documentation scale.

## Project Type Detection

### Decision Tree

```
Is there a bin/ field or executable entry?
├── Yes → CLI Application
└── No
    └── Is there a library export (index, lib, __init__)?
        ├── Yes
        │   └── Are there HTTP handlers or server setup?
        │       ├── Yes → Service with Library
        │       └── No → Library
        └── No
            └── Is there a main entry (main.*, app.*, server.*)?
                ├── Yes
                │   └── Are there HTTP handlers?
                │       ├── Yes → Service
                │       └── No → Application
                └── No → Library (default)
```

### Classification Signals

| Signal | Type | Confidence |
|--------|------|------------|
| `package.json` bin field | CLI | High |
| `main()` function in main.go/py/rs | Application | High |
| HTTP listener/server setup | Service | High |
| `exports` with functions/classes | Library | High |
| `index.js/ts` exporting from submodules | Library | Medium |
| `__init__.py` without main block | Library | Medium |
| `setup.py` or `pyproject.toml` without scripts | Library | Medium |
| `Cargo.toml` with `[lib]` only | Library | High |

### Language-Specific Patterns

#### JavaScript/TypeScript

**CLI:**
```json
// package.json
{
  "bin": {
    "my-cli": "./bin/cli.js"
  }
}
```
```javascript
// bin/cli.js
#!/usr/bin/env node
const { program } = require('commander');
program.parse();
```

**Library:**
```json
// package.json
{
  "main": "dist/index.js",
  "types": "dist/index.d.ts"
}
```
```typescript
// src/index.ts
export { someFunction } from './some-function';
```

**Service:**
```javascript
// server.js
const express = require('express');
const app = express();
app.listen(3000);
```

#### Python

**CLI:**
```toml
# pyproject.toml
[project.scripts]
my-cli = "my_package.cli:main"
```
```python
# my_package/cli.py
import argparse
def main():
    parser = argparse.ArgumentParser()
```

**Library:**
```python
# my_package/__init__.py
from .core import SomeClass
__all__ = ['SomeClass']
```

**Service:**
```python
# app.py
from fastapi import FastAPI
app = FastAPI()
uvicorn.run(app)
```

#### Go

**CLI:**
```go
// main.go
package main
import "flag"
func main() {
    flag.Parse()
    // ...
}
```

**Library:**
```go
// somepackage/somefile.go
package somepackage
func SomeFunction() { }
```

**Service:**
```go
// main.go
package main
import "net/http"
func main() {
    http.ListenAndServe(":8080", nil)
}
```

#### Rust

**CLI:**
```toml
# Cargo.toml
[[bin]]
name = "my-cli"
path = "src/main.rs"
```
```rust
// src/main.rs
use clap::Parser;
fn main() {
    let args = Args::parse();
}
```

**Library:**
```toml
# Cargo.toml
[lib]
path = "src/lib.rs"
```
```rust
// src/lib.rs
pub fn some_function() { }
```

**Service:**
```rust
// src/main.rs
use actix_web::{App, HttpServer};
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| App::new())
        .bind("127.0.0.1:8080")?
        .run()
        .await
}
```

## API Documentation Scale

Based on project type, determine how much API documentation to include:

| Project Type | API Docs Level | Reasoning |
|--------------|----------------|-----------|
| Library | Full | Consumers need complete API reference |
| CLI | None | Users interact via commands, not code |
| Application | Minimal | Document public API if extensible |
| Service | Minimal | Document endpoints, not internal code |
| Service+Library | Full | Both consumers and extenders |

### Full API Documentation Includes

- All public functions/methods
- Function signatures with types
- Parameter descriptions
- Return types and values
- Error conditions
- Usage examples

### Minimal API Documentation Includes

- Main entry points only
- Extension points (hooks, plugins)
- Configuration interfaces

### No API Documentation

- Omit section entirely
- Focus on usage guide and commands

## Edge Cases

### Monorepo with Multiple Packages

**Detection:** Multiple package.json, Cargo.toml, or pyproject.toml files

**Handling:**
- Document each package in its own README.md
- Root README documents overall structure

### Mixed CLI + Library

**Detection:** Both bin field and main exports

**Handling:**
- Classify as CLI
- Include usage guide for CLI
- Include API documentation for library usage

### Hybrid Service + Library

**Detection:** HTTP handlers + library exports

**Handling:**
- Classify as Service
- Include API docs for library consumers
- Include endpoint documentation for service users
