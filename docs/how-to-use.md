# How to Use the Task Sync System

## The Core Idea

**Two separate systems for two different needs:**
- **Coding tasks** → bd (git-backed, in this project)
- **Non-coding tasks** → Sunsama/Notion (your existing workflow)

Everything happens automatically. You just create tasks and commit code.

## Daily Workflow

### 1. Start Your Day (Automatic)

When you open Claude Code in this project:

```
⚡ 1 coding tasks ready to work on (2 total). Run 'bd ready' to see them.
```

The startup hook shows you what's ready. **No action needed.**

---

### 2. See What to Work On

```bash
bd ready
```

Shows unblocked coding tasks. That's it.

**Example output:**
```
task-sync-system-abc [P0] [bug] open [security urgent] - Fix authentication
task-sync-system-xyz [P1] [feature] open [api] - Add rate limiting
```

---

### 3. Capture Tasks as You Think of Them

**The most important habit:**

```bash
bd create "Fix broken login button"
```

**That's it.** 2 seconds. Never forget a task again.

**With details:**
```bash
bd create "Add rate limiting to API" \
  -d "Prevent abuse of public endpoints" \
  -p 0 \
  -l feature,api,security
```

Options:
- `-d` = Description
- `-p` = Priority (0=highest, 4=lowest)
- `-l` = Labels (comma-separated)

---

### 4. Update Task Status (As You Work)

```bash
# Start working on a task
bd update task-sync-system-abc --status in_progress

# Check task details
bd show task-sync-system-abc

# Close when done
bd close task-sync-system-abc
```

---

### 5. Commit Your Code (Automatic Git Integration)

```bash
git commit -m "Fix authentication bug"
```

**The pre-commit hook automatically:**
1. Flushes bd changes to JSONL
2. Stages `.beads/beads.left.jsonl`
3. Includes it in your commit

**No `git add` needed. No extra steps. Just commit.**

---

### 6. Push to GitHub

```bash
git push
```

Your bd tasks are now version-controlled with your code.

---

## Common Commands

### Creating Tasks

```bash
bd create "Task name"                          # Simple
bd create "Task" -p 0 -l bug,urgent           # With priority & labels
bd create "Task" -d "Details here"            # With description
bd create "Task" -t feature                   # With type (bug/feature/task/epic)
bd create "Task" --assignee yourname          # Assign to someone
```

### Viewing Tasks

```bash
bd ready                      # Unblocked tasks (what to work on now)
bd list                       # All tasks
bd list --status open        # Filter by status
bd list --priority 0         # High priority only
bd list --label urgent       # Filter by label
bd show task-sync-system-abc # Task details
```

### Managing Tasks

```bash
bd update task-sync-system-abc --status in_progress
bd update task-sync-system-abc --priority 0
bd update task-sync-system-abc --assignee alice
bd close task-sync-system-abc
bd close task-sync-system-abc --reason "Fixed in PR #42"
bd reopen task-sync-system-abc
```

### Dependencies (Advanced)

```bash
# Task B blocks Task A (A can't start until B is done)
bd dep add task-B task-A --type blocks

# View dependency tree
bd dep tree task-sync-system-abc

# Remove dependency
bd dep remove task-B task-A

# Check for circular dependencies
bd dep cycles
```

### Git Workflow

```bash
# Your normal workflow - bd is automatic
git commit -m "message"      # bd files auto-staged!
git push                     # bd syncs to GitHub

# Pull changes from others
git pull                     # bd auto-imports latest tasks

# View what's in the commit
git show HEAD --name-only    # You'll see .beads/beads.left.jsonl
```

---

## Non-Coding Tasks (Notion/Sunsama)

### In Sunsama (Daily Planning)

1. Time block your day
2. Pull tasks from Notion, Gmail, GitHub
3. Work on tasks
4. Complete → Syncs back automatically

### In Claude Code (Query Notion)

```bash
# Load Notion tasks into current session
/sync-notion

# Or ask Claude directly
"What are my Notion tasks?"
```

Claude uses Notion MCP to search/list tasks on demand.

---

## What Happens Automatically

### ✅ On Session Start
- Notion sync runs (if 1Password available)
- bd checks for tasks in current project
- Shows you what's ready

### ✅ When You Create/Update bd Tasks
- Auto-exports to JSONL after 5 seconds
- Survives crashes, restarts, everything

### ✅ When You Git Commit
- Pre-commit hook flushes bd changes
- Auto-stages `.beads/beads.left.jsonl`
- Includes bd tasks in commit (no manual add!)

### ✅ When You Git Pull
- Post-merge hook imports latest bd tasks
- Your local database updates automatically

### ✅ When You Git Push
- Tasks sync to GitHub
- Other collaborators get your tasks on pull

---

## Minimal Mental Model

**You only need to remember 2 commands:**

1. **When you think of a task:**
   ```bash
   bd create "task name"
   ```

2. **When you want to see work:**
   ```bash
   bd ready
   ```

**Everything else is automatic or shown when you need it.**

---

## Troubleshooting

### "bd: command not found"

**Fix:**
```bash
# Install bd
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash
```

### "No tasks shown on startup"

**Cause:** Not in a directory with bd initialized

**Fix:**
```bash
cd ~/projects/task-sync-system  # Or your project
bd list  # Verify bd works
```

### "Git commit didn't include bd file"

**Cause:** Pre-commit hook not installed or outdated

**Fix:**
```bash
cd ~/projects/task-sync-system
bd hooks install  # Reinstall hooks
```

### "Notion sync failed"

**Cause:** 1Password token not available

**Fix:**
```bash
# Test token access
op read "op://Private/Notion MCP API Credentials/credential"

# If that fails, check 1Password is running
# Otherwise, run sync manually
/sync-notion
```

### "Too many test tasks"

**Fix:**
```bash
# Close test tasks
bd list --label test | grep task-sync | while read id _; do bd close $id; done

# Or delete them
bd delete task-sync-system-abc
```

---

## Tips & Best Practices

### 1. Use Labels Consistently

```bash
bd create "Task" -l bug,security,urgent
bd create "Task" -l feature,api
bd create "Task" -l docs,website
```

Later, filter easily:
```bash
bd list --label security
bd list --label urgent
```

### 2. Set Priorities

- **P0** = Blocking/Critical (do now)
- **P1** = High (this week)
- **P2** = Medium (this month)
- **P3** = Low (someday)
- **P4** = Nice to have

```bash
bd create "Fix production bug" -p 0
bd create "Add new feature" -p 1
bd create "Refactor old code" -p 2
```

### 3. Use Dependencies for Complex Work

```bash
# Can't deploy until tests pass
bd dep add task-write-tests task-deploy --type blocks

# Now bd ready won't show deploy until tests are done
```

### 4. Close Tasks with Reasons

```bash
bd close task-abc --reason "Fixed in commit abc123"
bd close task-xyz --reason "Duplicate of task-abc"
bd close task-old --reason "No longer needed"
```

Helps with project retrospectives later.

### 5. Review Weekly

```bash
# See what you accomplished
bd list --status closed

# See what's stale
bd stale

# Clean up old closed tasks (optional)
bd cleanup --older-than 30d
```

---

## Integration with Other Tools

### GitHub Issues (Optional)

Create a GitHub Action to auto-create public issues from bd:

```yaml
# .github/workflows/sync-to-issues.yml
on:
  push:
    paths: ['.beads/beads.left.jsonl']
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Create GitHub Issues
        run: |
          # Parse JSONL and create issues
          gh issue create --title "..." --body "..."
```

### VS Code / Cursor

bd works in any directory. Just `cd` to your project:

```bash
cd ~/projects/task-sync-system
bd ready
```

### CI/CD

bd tasks are just JSONL files. Query them in CI:

```bash
# Check for P0 tasks before deploy
P0_COUNT=$(bd list --priority 0 --json | jq '. | length')
if [ "$P0_COUNT" -gt 0 ]; then
  echo "❌ Cannot deploy: $P0_COUNT critical tasks open"
  exit 1
fi
```

---

## Quick Reference Card

```bash
# CREATE
bd create "Task name" -p 0 -l tags

# VIEW
bd ready                    # What to work on now
bd list                     # All tasks
bd show task-id             # Task details

# UPDATE
bd update task-id --status in_progress
bd close task-id

# GIT
git commit -m "message"     # bd auto-staged!
git push                    # bd syncs to GitHub

# DEPENDENCIES
bd dep add blocker blocked --type blocks
bd dep tree task-id

# NOTION
/sync-notion                # Load Notion tasks
```

---

## Architecture Summary

```
┌─────────────────────────────────────┐
│ CODING TASKS (Git-Backed)           │
├─────────────────────────────────────┤
│ bd create → .beads/beads.left.jsonl │
│ git commit (auto-stages bd file)    │
│ git push → GitHub                   │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ NON-CODING TASKS (Notion/Sunsama)   │
├─────────────────────────────────────┤
│ Sunsama ↔ Notion (native)           │
│ Claude → Notion MCP (on demand)     │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ UNIFIED VIEW (Claude Code)          │
├─────────────────────────────────────┤
│ Startup: Shows bd + Notion          │
│ bd ready: Coding work               │
│ /sync-notion: Non-coding work       │
└─────────────────────────────────────┘
```

---

## Next Steps

1. **Start using it**: `bd create "Try the system"`
2. **Build the habit**: Create tasks as you think of them
3. **Review weekly**: `bd list --status closed` (see progress!)
4. **Customize**: Add labels, priorities that match your workflow

**The system gets better the more you use it.**

---

**Created**: 2025-01-18
**Project**: https://github.com/miqcie/task-sync-system
**Architecture**: [simplified-architecture.md](simplified-architecture.md)
**Status**: Ready to use ✓
