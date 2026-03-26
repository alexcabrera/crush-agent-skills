# App Detection Reference

## Detection Strategy

The execute skill uses these patterns to detect application type:

### Web Applications

Indicators:
- `package.json` with `next`, `vite`, `react`, `vue`, `svelte`, `astro`
- `index.html` or `app.html`
- `go.mod` with `net/http`
- Explicit `type: web` in `specs/*/design.md`

### CLI/TUI Applications

Indicators:
- `cmd/` directory (Go)
- `Cargo.toml` with `[[bin]]` target
- Main entry in Python with `argparse`/`click`
- `package.json` without web frameworks
- Explicit `type: cli` in `specs/*/design.md`

### Detection Function

```bash
detect_app_type() {
    local project_root="${PROJECT_ROOT:-$(pwd)}
    
    # Check for explicit marker
    if [[ -f "specs/${SPEC_NAME:-}/design.md" ]]; then
        if grep -q "type:\s*web\|type:\s*api" specs/*/design.md 2>/dev/null; then
            echo "web"
            return
        fi
    fi
    
    # Check for web frameworks in package.json
    if [[ -f "package.json" ]]; then
        if grep -qE "next|vite| react| vue| svelte| astro" package.json 2>/dev/null; then
            echo "web"
            return
        fi
    fi
    
    # Check for index.html or app.html
    if [[ -f "index.html" ]] || [[ -f "app.html" ]]; then
        echo "web"
        return
    fi
    
    # Check
Go web server
    if [[ -f "go.mod" ]]; then
        if grep -q "net/http" go.mod 2>/dev/null; then
            echo "web"
            return
        fi
    fi
    
    # Default to CLI
    echo "cli"
}
```
