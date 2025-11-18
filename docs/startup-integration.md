# Claude Code Startup Integration

## What It Does

When Claude Code starts, the startup hook automatically:

1. **Syncs Notion tasks** (if 1Password token available)
2. **Checks for bd tasks** in current project
3. **Shows you what's ready** to work on

## Startup Messages

### If you're in a project with bd

```
⚡ 1 coding tasks ready to work on (2 total). Run 'bd ready' to see them.
```

### If you have Notion tasks

```
📋 56 active tasks synced from Notion. Run /sync-notion to load them into /todos.
```

### If bd has tasks but none are ready

```
📦 5 coding tasks tracked (all blocked or in progress). Run 'bd list' to see them.
```

## How It Works

The hook lives at: `~/.claude/hooks/auto-sync-notion.sh`

**On session start:**
1. Checks if HOOK_MATCHER=startup
2. Tries to sync Notion (skips if 1Password unavailable)
3. Checks for bd database in current directory or parents
4. Outputs JSON messages for Claude to display

## Commands After Startup

### View coding tasks

```bash
bd ready          # See unblocked coding work
bd list           # See all coding tasks
bd show bd-abc    # See task details
```

### View non-coding tasks

```bash
/sync-notion      # Load Notion tasks into /todos
```

### Combined workflow

```bash
# See everything
bd ready && echo "---" && /sync-notion
```

## Troubleshooting

### "No bd tasks shown"

**Cause**: Not in a directory with bd initialized

**Fix**:
```bash
cd ~/projects/task-sync-system  # Or your project
bd list  # Verify bd works
```

### "Notion sync failed"

**Cause**: 1Password token not available

**Fix**:
```bash
# Test token access
op read "op://Private/Notion MCP API Credentials/credential"

# If that works, run sync manually
/sync-notion
```

### "Hook not running"

**Cause**: Hook file permissions or location

**Fix**:
```bash
# Check hook exists and is executable
ls -la ~/.claude/hooks/auto-sync-notion.sh

# Make executable if needed
chmod +x ~/.claude/hooks/auto-sync-notion.sh
```

## Customization

### Change bd message threshold

Edit `~/.claude/hooks/auto-sync-notion.sh`:

```bash
# Only show if more than 5 tasks
if [ "$BD_COUNT" -gt 5 ]; then
```

### Disable bd check

Comment out lines 63-86 in the hook:

```bash
# # Check for bd (beads) tasks...
# BD_INFO=$(bd info --json 2>/dev/null || echo '{"issue_count":0}')
# ...
```

### Disable Notion sync

Comment out lines 20-60 in the hook.

## Performance

| Operation | Time | Impact |
|-----------|------|--------|
| Notion sync | 2-10s | Runs in background |
| bd check | <100ms | Near instant |
| Total startup | 2-10s | Only on fresh startup |

## Integration with Other Projects

The bd check works **per-project**:

```bash
# Project A
cd ~/projects/task-sync-system
# Startup shows: "⚡ 1 coding tasks ready..."

# Project B (no bd)
cd ~/projects/other-project
# Startup shows: (no bd message)

# Project C (bd but no tasks)
cd ~/projects/empty-bd-project
# Startup shows: (no bd message)
```

This is intentional - bd tasks are scoped to the project you're working in.

---

**Created**: 2025-01-18
**Hook Location**: `~/.claude/hooks/auto-sync-notion.sh`
**Status**: Active, tested
