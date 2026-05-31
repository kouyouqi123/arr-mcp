# Development Guidelines

This document contains critical information about working with this codebase. Follow these guidelines precisely.

## Core Development Rules

1. Package Management
   - ONLY use uv, NEVER pip
   - Installation: `uv add package`
   - Running tools: `uv run tool`
   - Upgrading: `uv add --dev package --upgrade-package package`
   - FORBIDDEN: `uv pip install`, `@latest` syntax

2. Code Quality
   - Type hints required for all code
   - Public APIs must have docstrings
   - Functions must be focused and small
   - Follow existing patterns exactly
   - Line length: 88 chars maximum

3. Testing Requirements
   - Framework: `uv run pytest`
   - Async testing: use anyio, not asyncio
   - Coverage: test edge cases and errors
   - New features require tests
   - Bug fixes require regression tests

4. Code Style
    - PEP 8 naming (snake_case for functions/variables)
    - Class names in PascalCase
    - Constants in UPPER_SNAKE_CASE
    - Document with docstrings
    - Use f-strings for formatting

- For commits fixing bugs or adding features based on user reports add:
  ```bash
  git commit --trailer "Reported-by:<name>"
  ```
  Where `<name>` is the name of the user.

- For commits related to a Github issue, add
  ```bash
  git commit --trailer "Github-Issue:#<number>"
  ```
- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Development Philosophy

- **Simplicity**: Write simple, straightforward code
- **Readability**: Make code easy to understand
- **Performance**: Consider performance without sacrificing readability
- **Maintainability**: Write code that's easy to update
- **Testability**: Ensure code is testable
- **Reusability**: Create reusable components and functions
- **Less Code = Less Debt**: Minimize code footprint

## Coding Best Practices

- **Early Returns**: Use to avoid nested conditions
- **Descriptive Names**: Use clear variable/function names (prefix handlers with "handle")
- **Constants Over Functions**: Use constants where possible
- **DRY Code**: Don't repeat yourself
- **Functional Style**: Prefer functional, immutable approaches when not verbose
- **Minimal Changes**: Only modify code related to the task at hand
- **Function Ordering**: Define composing functions before their components
- **TODO Comments**: Mark issues in existing code with "TODO:" prefix
- **Simplicity**: Prioritize simplicity and readability over clever solutions
- **Build Iteratively** Start with minimal functionality and verify it works before adding complexity
- **Run Tests**: Test your code frequently with realistic inputs and validate outputs
- **Build Test Environments**: Create testing environments for components that are difficult to validate directly
- **Functional Code**: Use functional and stateless approaches where they improve clarity
- **Clean logic**: Keep core logic clean and push implementation details to the edges
- **File Organsiation**: Balance file organization with simplicity - use an appropriate number of files for the project scale

## System Architecture

arr-mcp is an MCP server that provides natural language management of a home media server stack (Plex, Sonarr, Radarr, SABnzbd, etc.) via Podman or Docker.

### Deployment model

arr-mcp runs as a container alongside the media stack. It communicates with the container runtime via a bind-mounted Unix socket (`/run/user/1000/podman/podman.sock` for rootless Podman, `/var/run/docker.sock` for Docker). The host mounts `/opt/stacks` and `/media-server` into the container to allow filesystem and compose file access.

```
Claude (MCP client)
      │  HTTP + Bearer auth
      ▼
 arr-mcp container
      │  Unix socket
      ▼
 Podman/Docker runtime
      │
      ▼
 Media stack containers (plex, sonarr, radarr, ...)
```

### Planned: host-side helper agent (see issues #12, #13)

The current architecture cannot run `podman-compose` or `systemctl` commands because those binaries are not available inside the container. The planned solution is a small host-side helper process running as the media user (UID 1000), exposed to arr-mcp over a dedicated Unix socket. This will enable:

- Full stack lifecycle management via `podman-compose`
- Quadlet/systemd service management (`systemctl --user`)
- Reloading changed compose files without host access

### Target environment

- **OS**: Debian/Ubuntu
- **Runtime**: Rootless Podman under a dedicated `media` service account (UID 1000)
- **Socket**: `/run/user/1000/podman/podman.sock`
- **Stacks**: `/opt/stacks/<stack-name>/compose.yaml`
- **Media**: `/media-server/`

## Core Components

- `src/arr_mcp/server.py`: Starlette ASGI app, API key auth middleware, entry point
- `src/arr_mcp/config.py`: Pydantic settings loaded from environment / `.env`
- `src/arr_mcp/runtime/detector.py`: Auto-detects Podman or Docker socket at startup
- `src/arr_mcp/runtime/client.py`: Async HTTP client over the container runtime socket
- `src/arr_mcp/tools/containers.py`: Container lifecycle tools (list, start, stop, restart, remove, logs, stats)
- `src/arr_mcp/tools/stacks.py`: Stack management tools (up, down, pull, restart, compose read/write/validate)
- `src/arr_mcp/tools/filesystem.py`: Filesystem tools scoped to allowed paths (disk usage, directory list, read, write)
- `src/arr_mcp/tools/logs.py`: Log reading and searching tools

## Pull Requests

- Create a detailed message of what changed. Focus on the high level description of
  the problem it tries to solve, and how it is solved. Don't go into the specifics of the
  code unless it adds clarity.

- Always add `ArthurClune` as reviewer.

- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Python Tools

## Code Formatting

1. Ruff
   - Format: `uv run ruff format .`
   - Check: `uv run ruff check .`
   - Fix: `uv run ruff check . --fix`
   - Critical issues:
     - Line length (88 chars)
     - Import sorting (I001)
     - Unused imports
   - Line wrapping:
     - Strings: use parentheses
     - Function calls: multi-line with proper indent
     - Imports: split into multiple lines

2. Type Checking
   - Tool: `uv run pyright`
   - Requirements:
     - Explicit None checks for Optional
     - Type narrowing for strings
     - Version warnings can be ignored if checks pass

3. Pre-commit
   - Config: `.pre-commit-config.yaml`
   - Runs: on git commit
   - Tools: Prettier (YAML/JSON), Ruff (Python)
   - Ruff updates:
     - Check PyPI versions
     - Update config rev
     - Commit config first

## Error Resolution

1. CI Failures
   - Fix order:
     1. Formatting
     2. Type errors
     3. Linting
   - Type errors:
     - Get full line context
     - Check Optional types
     - Add type narrowing
     - Verify function signatures

2. Common Issues
   - Line length:
     - Break strings with parentheses
     - Multi-line function calls
     - Split imports
   - Types:
     - Add None checks
     - Narrow string types
     - Match existing patterns

3. Best Practices
   - Check git status before commits
   - Run formatters before type checks
   - Keep changes minimal
   - Follow existing patterns
   - Document public APIs
   - Test thoroughly