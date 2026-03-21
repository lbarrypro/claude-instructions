# claude-instructions

> Operating instructions and behavioral guidelines for Claude AI sessions on this project.

## Overview

This repository contains `CLAUDE.md`, a comprehensive set of instructions that define how Claude should behave when working on projects. It covers session startup, coding best practices, security, testing strategy, CI/CD, documentation conventions, and more.

The instructions are designed to be dropped into any project root so Claude follows consistent, senior-engineer-level standards across all sessions.

## Tech Stack

This is a documentation-only repository. No runtime dependencies.

## Prerequisites

None. This repository contains only markdown files.

## Getting Started

```bash
# Clone
git clone https://github.com/lbarrypro/claude-instructions.git

# Use CLAUDE.md in your project
cp CLAUDE.md /path/to/your/project/CLAUDE.md
```

## Project Structure

```
CLAUDE.md          # Main Claude operating instructions
README.md          # This file
CHANGELOG.md       # Change history
.claudeignore      # Files Claude should not read
docs/              # Supporting documentation
tasks/             # Work-in-progress tracking
```

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Conventions](docs/CONVENTIONS.md)
- [Lessons Learned](docs/lessons.md)
- [Changelog](CHANGELOG.md)

## Contributing

Branch naming: `claude/<short-description>-<session-id>`
All changes via Pull Request — see Git Workflow in `CLAUDE.md`.

## License

MIT
