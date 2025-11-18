# Task Management Architecture

## System Overview

Event-driven, multi-context task capture with automatic routing and git-backed coding tasks.

## Architecture Diagram

```mermaid
graph TB
    subgraph "CAPTURE LAYER - Zero Friction Documentation"
        A1[Claude Code CLI<br/>bd create task]
        A2[Notion UI<br/>Add task + #code tag]
        A3[Sunsama<br/>Time blocking]
    end

    subgraph "SYNC LAYER - Automatic Routing"
        B1[bd SQLite<br/>Local cache]
        B2[.beads/issues.jsonl<br/>Git-committed]
        B3[Notion Webhook<br/>Real-time events]
        B4[Webhook Listener<br/>Bun.sh service]
        B5[Sunsama Sync<br/>Native integration]
    end

    subgraph "GIT LAYER - Version Control"
        C1[Git Commit]
        C2[GitHub Action<br/>On JSONL change]
        C3[GitHub Issues<br/>Build in public]
    end

    subgraph "VIEW LAYER - Context-Appropriate Access"
        D1[Claude Code<br/>bd ready + Notion MCP]
        D2[Notion<br/>All tasks unified]
        D3[Sunsama<br/>Daily planning]
        D4[GitHub<br/>Public visibility]
    end

    %% Capture flows
    A1 -->|Creates| B1
    A2 -->|Triggers| B3
    A3 -->|Syncs to| A2

    %% Sync flows
    B1 -->|Auto-export<br/>5s debounce| B2
    B3 -->|POST request| B4
    B4 -->|If #code tag| B1
    B5 -->|Bidirectional| A2

    %% Git flows
    B2 -->|Committed| C1
    C1 -->|Triggers| C2
    C2 -->|Creates/updates| C3

    %% View flows
    B1 -.->|bd ready| D1
    A2 -.->|Notion MCP| D1
    A2 -.->|View all| D2
    A2 -.->|Pull tasks| D3
    C3 -.->|Browse| D4

    %% Styling
    classDef capture fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef sync fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef git fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef view fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px

    class A1,A2,A3 capture
    class B1,B2,B3,B4,B5 sync
    class C1,C2,C3 git
    class D1,D2,D3,D4 view
```

## Data Flow by Task Type

### Coding Tasks (Git-Backed)

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant bd as bd SQLite
    participant JSONL as .beads/issues.jsonl
    participant Git
    participant GH as GitHub Issues

    User->>Claude: "bd create 'Fix auth bug'"
    Claude->>bd: Create issue (instant)
    bd-->>Claude: bd-a1b2 created

    Note over bd,JSONL: 5-second debounce
    bd->>JSONL: Auto-export

    User->>Git: git add .beads/issues.jsonl
    User->>Git: git commit & push
    Git->>GH: GitHub Action triggers
    GH->>GH: Create public issue
```

### Non-Coding Tasks (Notion-First)

```mermaid
sequenceDiagram
    participant User
    participant Sunsama
    participant Notion as Notion DB
    participant Webhook as Webhook Listener
    participant bd as bd SQLite

    User->>Sunsama: Add task "Email team"
    Sunsama->>Notion: Native sync
    Notion->>Webhook: POST event
    Webhook->>Webhook: Check tags

    alt Has #code tag
        Webhook->>bd: Create coding task
        bd->>bd: Git sync flow
    else No #code tag
        Webhook-->>Notion: Stays in Notion
    end
```

## Key Architectural Decisions

### 1. Tag-Based Routing
- **`#code` tag** → Routes to bd (git-backed)
- **No tag** → Stays in Notion/Sunsama
- **Solves**: Automatic categorization without manual sorting

### 2. Event-Driven Sync
- **Notion webhooks** → Real-time (sub-500ms)
- **bd auto-export** → 5-second debounce (configurable)
- **Solves**: No polling, no startup delays

### 3. Capture Anywhere
- **Claude**: `bd create` (CLI-native)
- **Notion**: Add task (visual interface)
- **Sunsama**: Time blocking (ADHD-friendly)
- **Solves**: "I forget to document" → Zero friction capture

### 4. Git as Source of Truth (Coding Only)
- **bd issues** → Version controlled in `.beads/issues.jsonl`
- **Notion tasks** → Stay in Notion (not git)
- **Solves**: Coding tasks have full git history

## Token Efficiency

| Operation | Tokens | Method |
|-----------|--------|--------|
| Create task in Claude | ~50 | `bd create` (local, no LLM) |
| View ready tasks | ~200 | `bd ready` (local query) |
| Sync to Notion | ~0 | Webhook (background) |
| View all tasks | ~3K | Notion MCP (on demand) |

**Total per session**: ~3,500 tokens (vs 60K-300K for full MCP sync)

## Implementation Status

- [x] Phase 0: Architecture design
- [x] Phase 1a: Project setup with git
- [x] Phase 1b: bd initialization with hooks
- [ ] Phase 1c: Test bd workflow
- [ ] Phase 2: Notion webhook listener
- [ ] Phase 3: GitHub Action for public issues
- [ ] Phase 4: Claude Code startup integration

---

**Created**: 2025-01-18
**Architecture**: Event-driven, multi-context capture
**Primary Goal**: Eliminate "forgetting to document tasks"
