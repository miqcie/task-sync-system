# Task Sync System

> Event-driven task management for ADHD workflows with git-backed coding tasks

## Problem

**"I forget to document tasks, so I can't remember what I wanted to work on."**

Context switching between tools (Claude Code, Notion, Sunsama) makes task capture high-friction. Forgetting to document = lost work.

## Solution

**Capture tasks wherever you are, with zero friction:**

```bash
# In Claude Code CLI
bd create "Fix authentication bug" -l code,urgent

# In Notion
Add task with #code tag → Auto-routes to bd

# In Sunsama
Add task → Syncs to Notion → Webhook routes if #code
```

**The system handles routing automatically.**

## Architecture

See [docs/architecture.md](docs/architecture.md) for full architectural diagrams.

### Key Components

1. **bd (beads)** - Git-backed issue tracker for coding tasks
2. **Notion Webhooks** - Real-time event sync (future)
3. **Sunsama** - Central hub for daily planning
4. **GitHub Actions** - Auto-create public issues (future)

### Data Flow

```
Coding tasks:     Claude → bd → .beads/issues.jsonl → Git → GitHub Issues
Non-coding tasks: Sunsama → Notion (stays there)
Hybrid tasks:     Notion (#code tag) → Webhook → bd
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
│   ├── beads.db         # SQLite cache (gitignored)
│   └── issues.jsonl     # Source of truth (committed)
├── .github/
│   └── workflows/
│       └── sync-to-issues.yml  # Auto-create GitHub Issues
├── docs/
│   ├── architecture.md  # Mermaid diagrams
│   └── webhook-security.md  # Security considerations
├── src/
│   └── webhook-listener.js  # Notion webhook handler
└── README.md
```

## Implementation Phases

- [x] **Phase 1**: Project setup + bd initialization
- [ ] **Phase 2**: Notion webhook listener (Bun.sh)
- [ ] **Phase 3**: GitHub Action for public issues
- [ ] **Phase 4**: Claude Code integration

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
# bd auto-exports to .beads/issues.jsonl after 5 seconds
git add .beads/issues.jsonl
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
| Notion webhook | ~0 | Background |
| Notion MCP query | ~3K | On demand |

**Total**: ~3,500 tokens/session vs 60K-300K for vanilla MCP sync.

## Why This Architecture?

1. **ADHD-friendly**: Capture in any tool, no context switching
2. **Git-backed**: Coding tasks version-controlled
3. **Build in public**: Auto-sync to GitHub Issues
4. **Token-efficient**: 20x better than naive MCP
5. **Event-driven**: Real-time sync, no polling

## Contributing

This is a personal project, but ideas welcome! See [docs/architecture.md](docs/architecture.md) for design decisions.

## License

MIT

---

**Author**: Chris McConnell
**Created**: 2025-01-18
**Status**: Phase 1 complete, Phase 2 in progress
