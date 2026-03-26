# README Templates

Templates and examples for generating comprehensive README.md documentation.

## Standard Template

```markdown
# {Project Name}

{One-line tagline}

## Overview

{2-3 paragraphs explaining what it is, why it exists, key features}

## Installation

{Installation instructions for all supported platforms}

## Quick Start

{Minimal working example - copy-paste ready}

## Usage Guide

{Comprehensive walkthrough of all features}

## Deep Reference

{Detailed options, configuration, edge cases}

## API Documentation

{For libraries: Full API reference}
{For applications: Omitted or minimal}
```

---

## Section Details

### Project Name & Tagline

**Pattern:**
```markdown
# my-project

A brief description of what this does in one line.
```

**Guidelines:**
- Use the package/module name
- Tagline should answer "what does it do?"
- No period at end of tagline
- Keep under 80 characters

### Overview

**Pattern:**
```markdown
## Overview

{Project Name} is a {category} that {primary purpose}. It {key differentiator}.

**Key Features:**
- Feature 1: {brief description}
- Feature 2: {brief description}
- Feature 3: {brief description}

{Additional context about why this exists or who it's for}
```

**Guidelines:**
- 2-3 paragraphs maximum
- First paragraph: what + why
- Second paragraph: key features as bullet list
- Third paragraph (optional): context, audience, philosophy

### Installation

**For Node.js packages:**
```markdown
## Installation

```bash
npm install my-package
# or
yarn add my-package
# or
pnpm add my-package
```

**Requirements:**
- Node.js 18 or higher
- npm, yarn, or pnpm
```

**For Python packages:**
```markdown
## Installation

```bash
pip install my-package
# or
pipx install my-package  # for CLI tools
```

**Requirements:**
- Python 3.10 or higher
```

**For Go packages:**
```markdown
## Installation

```bash
go install github.com/user/my-package@latest
```

Or clone and build:

```bash
git clone https://github.com/user/my-package.git
cd my-package
go build -o my-package ./cmd/my-package
```
```

**For Rust packages:**
```markdown
## Installation

```bash
cargo install my-package
```

Or add to Cargo.toml:

```toml
[dependencies]
my-package = "1.0"
```
```

### Quick Start

**Pattern:**
```markdown
## Quick Start

{1-2 sentences about what this example demonstrates}

```{language}
{minimal working code}
```

{Expected output if applicable}

```text
{output}
```
```

**Guidelines:**
- Should work in under 5 minutes
- No complex setup
- Demonstrates core value immediately
- Include expected output

**Example (Library):**
```markdown
## Quick Start

Create your first item:

```python
from my_package import Client

client = Client(api_key="your-key")
item = client.items.create(name="My Item")
print(item.id)
```

Output:
```text
item_abc123
```
```

**Example (CLI):**
```markdown
## Quick Start

Initialize a new project:

```bash
my-cli init my-project
cd my-project
my-cli dev
```

Your project is now running at http://localhost:3000
```

### Usage Guide

**Pattern:**
```markdown
## Usage Guide

### {Feature Area 1}

{Description}

```{language}
{code example}
```

### {Feature Area 2}

{Description}

```{language}
{code example}
```

### {Feature Area N}
...
```

**Guidelines:**
- Organize by feature, not by code structure
- Each section has description + example
- Show common patterns
- Include edge cases that come up often

**Example (Library):**
```markdown
## Usage Guide

### Creating a Client

```python
from my_package import Client

client = Client(
    api_key="your-api-key",
    timeout=30,  # optional
    retries=3    # optional
)
```

### Working with Items

Create an item:

```python
item = client.items.create(
    name="My Item",
    tags=["important", "review"]
)
```

List items:

```python
items = client.items.list(limit=10)
for item in items:
    print(item.name)
```

Update an item:

```python
item = client.items.update(
    item_id="item_abc123",
    name="Updated Name"
)
```
```

**Example (CLI):**
```markdown
## Usage Guide

### Basic Commands

Initialize a new project:

```bash
my-cli init [project-name]
```

Options:
- `--template` - Use a template (default: default)
- `--no-git` - Skip git initialization

Start development server:

```bash
my-cli dev
```

Options:
- `--port` - Port number (default: 3000)
- `--host` - Host to bind (default: localhost)

Build for production:

```bash
my-cli build
```
```

### Deep Reference

**Pattern:**
```markdown
## Deep Reference

### Configuration

{Configuration file or environment variables}

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| option1 | string | "value" | What it does |
| option2 | number | 100 | What it does |

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| API_KEY | Yes | Your API key |
| DEBUG | No | Enable debug mode |

### {Advanced Topic 1}

{Detailed explanation}

### {Advanced Topic 2}

{Detailed explanation}

### Error Handling

{Common errors and solutions}

| Error | Cause | Solution |
|-------|-------|----------|
| E001 | Missing API key | Set API_KEY environment variable |
| E002 | Rate limited | Wait and retry with backoff |
```

**Guidelines:**
- Cover all options/flags/variables
- Document edge cases
- Include troubleshooting table
- Note performance considerations

### API Documentation

**For Libraries Only**

**Pattern:**
```markdown
## API Documentation

### Client

The main client for interacting with the API.

```typescript
const client = new Client(options: ClientOptions)
```

**Options:**
| Option | Type | Required | Description |
|--------|------|----------|-------------|
| apiKey | string | Yes | Your API key |
| timeout | number | No | Request timeout in ms (default: 30000) |

---

### items.create()

Create a new item.

```typescript
client.items.create(params: CreateItemParams): Promise<Item>
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| name | string | Yes | Item name |
| tags | string[] | No | Tags for the item |

**Returns:** `Promise<Item>`

**Example:**
```typescript
const item = await client.items.create({
  name: "My Item",
  tags: ["important"]
});
```

---
{Continue for all public APIs}
```

**Guidelines:**
- Document every public function/method
- Include type signatures
- Document all parameters
- Show return types
- Provide examples
- Document error conditions

---

## Complete Examples

### CLI Application README

```markdown
# my-cli

A command-line tool for managing projects.

## Overview

my-cli is a project management CLI that simplifies creating, developing, and
deploying projects. It provides a unified interface for common development
tasks.

**Key Features:**
- Project scaffolding with templates
- Built-in development server
- One-command deployment
- Plugin system for extensibility

## Installation

```bash
npm install -g my-cli
```

**Requirements:**
- Node.js 18 or higher

## Quick Start

Create a new project:

```bash
my-cli init my-project
cd my-project
my-cli dev
```

Your project is now running at http://localhost:3000

## Usage Guide

### Creating Projects

Initialize from a template:

```bash
my-cli init [name] --template [template]
```

Available templates:
- `default` - Basic project structure
- `typescript` - TypeScript with strict mode
- `library` - NPM library template

### Development

Start the development server:

```bash
my-cli dev --port 3001
```

### Building

Build for production:

```bash
my-cli build
```

### Deployment

Deploy to the platform:

```bash
my-cli deploy --env production
```

## Deep Reference

### Configuration

Create `my-cli.config.js` in your project root:

```javascript
module.exports = {
  port: 3000,
  buildDir: 'dist',
  plugins: []
};
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| MY_CLI_TOKEN | Authentication token for deployment |
| MY_CLI_DEBUG | Enable debug logging |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 130 | Interrupted (Ctrl+C) |
```

### Library README

```markdown
# my-library

A type-safe HTTP client with automatic retries.

## Overview

my-library is an HTTP client library that provides type-safe requests with
automatic retries, timeout handling, and request/response interception.

**Key Features:**
- Full TypeScript support with inferred types
- Automatic retry with exponential backoff
- Request/response interceptors
- Built-in telemetry

## Installation

```bash
npm install my-library
```

**Requirements:**
- Node.js 18 or higher
- TypeScript 5.0+ (for TypeScript projects)

## Quick Start

```typescript
import { Client } from 'my-library';

const client = new Client({ baseUrl: 'https://api.example.com' });

const response = await client.get('/users/{id}', {
  params: { id: '123' }
});

console.log(response.data.name);
```

## Usage Guide

### Creating a Client

```typescript
import { Client } from 'my-library';

const client = new Client({
  baseUrl: 'https://api.example.com',
  timeout: 30000,
  retries: 3
});
```

### Making Requests

GET request:

```typescript
const user = await client.get('/users/{id}', {
  params: { id: '123' }
});
```

POST request:

```typescript
const created = await client.post('/users', {
  body: { name: 'Alice', email: 'alice@example.com' }
});
```

### Error Handling

```typescript
import { ClientError } from 'my-library';

try {
  const user = await client.get('/users/{id}', { params: { id: '123' } });
} catch (error) {
  if (error instanceof ClientError) {
    console.log(error.status);    // HTTP status code
    console.log(error.code);      // Error code
    console.log(error.message);   // Human-readable message
  }
}
```

## Deep Reference

### Client Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| baseUrl | string | - | Base URL for all requests |
| timeout | number | 30000 | Request timeout in ms |
| retries | number | 3 | Number of retry attempts |
| headers | object | {} | Default headers |

### Error Codes

| Code | Description |
|------|-------------|
| E001 | Network error |
| E002 | Timeout |
| E003 | Invalid response |

## API Documentation

### Class: Client

```typescript
class Client {
  constructor(options: ClientOptions)
  get<T>(path: string, options?: RequestOptions): Promise<Response<T>>
  post<T>(path: string, options?: RequestOptions): Promise<Response<T>>
  put<T>(path: string, options?: RequestOptions): Promise<Response<T>>
  delete<T>(path: string, options?: RequestOptions): Promise<Response<T>>
}
```

### Interface: ClientOptions

```typescript
interface ClientOptions {
  baseUrl: string;
  timeout?: number;
  retries?: number;
  headers?: Record<string, string>;
}
```

### Interface: RequestOptions

```typescript
interface RequestOptions {
  params?: Record<string, string>;
  query?: Record<string, string | number | boolean>;
  body?: unknown;
  headers?: Record<string, string>;
}
```

### Interface: Response

```typescript
interface Response<T> {
  data: T;
  status: number;
  headers: Record<string, string>;
}
```
```
