# Task Sync System

> Simple, ADHD-friendly task management with git-backed coding tasks

## Problem

**"I forget to document tasks, so I can't remember what I wanted to work on."**

Context switching between tools (Claude Code, Notion, Sunsama) makes task capture high-friction. Forgetting to document = lost work.

## Solution

**Use the right tool for each job. No complex sync needed.**

```bash
# Coding tasks → bd (git-backed)
bd create "Fix authentication bug" -p 0 -l bug,security

# Non-coding tasks → Sunsama/Notion
Add in Sunsama → Syncs to Notion automatically

# View everything in Claude when needed
bd ready          # Coding tasks
/sync-notion      # Non-coding tasks
```

**Simple. Separate systems. Unified view when you need it.**

## Architecture

See [docs/simplified-architecture.md](docs/simplified-architecture.md) for the full explanation.

### Two Systems, Two Purposes

1. **bd (beads)** - Git-backed coding tasks (bugs, features, refactoring)
2. **Sunsama + Notion** - Non-coding tasks (planning, email, meetings)
3. **Notion MCP** - Query Notion from Claude when needed

### Data Flow

```
Coding tasks:     Claude → bd → .beads/beads.left.jsonl → Git → GitHub Issues
Non-coding tasks: Sunsama ↔ Notion (native integration)
Unified view:     bd ready + Notion MCP → Combined in Claude
```

## Quick Start

### 1. Create a Coding Task

```bash
cd ~/projects/task-sync-system
bd create "Add user authentication" -p 0 -l feature,code
```

### 2. View Ready Work

```bash
bd ready
# Shows tasks with no blockers
```

### 3. Commit to Git

```bash
git add .beads/issues.jsonl
git commit -m "Add authentication task"
git push
```

### 4. View in GitHub

GitHub Action auto-creates public issue (once configured).

## Project Structure

```
task-sync-system/
├── .beads/
│   ├── beads.db              # SQLite cache (gitignored)
│   └── beads.left.jsonl      # Source of truth (committed)
├── .github/
│   └── workflows/
│       └── sync-to-issues.yml  # Auto-create GitHub Issues (optional)
├── docs/
│   ├── simplified-architecture.md  # Current architecture
│   └── architecture.md            # Original (complex) design
└── README.md
```

## Implementation Status

- [x] **Phase 1**: bd setup + git hooks ✓
- [x] **Keep existing**: Notion sync (startup hook) ✓
- [ ] **Optional**: GitHub Action for public issues
- [x] **Simplified**: Removed webhook complexity ✓

## Commands

### bd (beads) Commands

```bash
bd create "Task name" -p 0-4 -l tags    # Create task
bd list                                  # List all tasks
bd list --status open                   # Filter by status
bd ready                                 # Show unblocked work
bd show bd-a1b2                         # Show details
bd update bd-a1b2 --status in_progress  # Update status
bd close bd-a1b2                        # Close task
bd dep add bd-456 bd-123 --type blocks  # Add dependency
bd dep tree bd-a1b2                     # Visualize dependencies
```

### Git Workflow

```bash
# bd auto-exports to .beads/beads.left.jsonl after 5 seconds
git add .beads/beads.left.jsonl
git commit -m "Update tasks"
git push

# Git hooks ensure immediate sync:
# - pre-commit: Flushes bd changes
# - post-merge: Imports latest JSONL
```

## Token Efficiency

| Operation | Tokens | Notes |
|-----------|--------|-------|
| `bd create` | ~50 | Local, no LLM |
| `bd ready` | ~200 | Local query |
| `/sync-notion` | ~3K | When you need it |
| Notion MCP query | ~2-5K | Ad-hoc queries |

**Total**: 3-8K tokens/session vs 60K-300K for complex sync.

## Why This Architecture?

1. **Simple**: Two separate systems, no complex sync
2. **ADHD-friendly**: Use the tool you're already in
3. **Git-backed**: Coding tasks version-controlled
4. **Token-efficient**: 20x better than naive MCP
5. **Maintainable**: No webhooks, no servers to run

## Contributing

This is a personal project, but ideas welcome! See [docs/architecture.md](docs/architecture.md) for design decisions.

## License

MIT

---

**Author**: Chris McConnell
**Created**: 2025-01-18
**Status**: Simplified architecture complete, ready to use
