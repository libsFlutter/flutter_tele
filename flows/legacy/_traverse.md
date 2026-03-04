# Traversal State

> Persistent recursion stack for tree traversal. AI reads this to know where it is and what to do next.

## Mode

- **BFS** (no comment): Breadth-first, analyze all domains systematically
- **DFS** (with comment): Depth-first, focus deeply on specific topic

## Source Path

lib/ and android/src/main/kotlin/org/telon/tele/flutter_tele/

## Focus (DFS only)

[none]

## Existing Flows Index

| Flow Path | Type | Topics | Key Decisions |
|-----------|------|--------|---------------|
| (none - first run) | - | - | - |

## Algorithm

```
RECURSIVE-UNDERSTAND(node):
    1. ENTER: Push node to stack, set phase = ENTERING
    2. EXPLORE: Read code, form understanding, set phase = EXPLORING
    3. SPAWN: Identify children (deeper concepts), set phase = SPAWNING
    4. RECURSE: For each child -> RECURSIVE-UNDERSTAND(child)
    5. SYNTHESIZE: Combine children insights, set phase = SYNTHESIZING
    6. EXIT: Pop from stack, bubble up summary, set phase = EXITING
```

## Current Stack

> Read top-to-bottom = root-to-current. Last item = where AI is now.

```
/ (root)                           SPAWNING
└── call-management                EXITING
```

## Stack Operations Log

| # | Operation | Node | Phase | Result |
|---|-----------|------|-------|--------|
| 1 | PUSH | / (root) | ENTERING | Stack initialized |
| 2 | UPDATE | / (root) | EXPLORING | _root.md created |
| 3 | UPDATE | / (root) | SPAWNING | 5 children identified |
| 4 | PUSH | call-management | ENTERING | Recursing into first child |
| 5 | UPDATE | call-management | EXPLORING | Analyzed TeleCall model |
| 6 | UPDATE | call-management | SYNTHESIZING | Synthesized understanding |
| 7 | UPDATE | call-management | EXITING | Ready to generate SDD |

## Current Position

- **Node**: call-management
- **Phase**: EXITING
- **Depth**: 1
- **Path**: lib/src/call.dart, android/.../TeleService.kt

## Pending Children

> Children identified but not yet explored (LIFO - last added explored first)

```
1. dialer
2. event-streaming
3. android-telecom-integration
4. endpoint
```

## Visited Nodes

> Completed nodes with their summaries

| Node Path | Summary | Flow Created |
|-----------|---------|--------------|
| call-management | TeleCall model: 40+ fields Dart, 10 fields Kotlin, duration calculation, URI parsing | PENDING |

## Next Action

1. Generate SDD flow for call-management
2. Create flows/sdd-call-model/ directory
3. Generate 01-requirements.md and 02-specifications.md
4. Update _traverse.md with flow creation
5. Pop from stack, bubble up to root

---

## Phase Definitions

### ENTERING
- Just arrived at this node
- Create _node.md file
- Read relevant source files
- Form initial hypothesis

### EXPLORING
- Deep analysis of this node's scope
- Validate/refine hypothesis
- Identify what belongs here vs. children

### SPAWNING
- Identify child concepts that need deeper exploration
- Add children to Pending stack
- Children are LOGICAL concepts, not filesystem paths

### SYNTHESIZING
- All children completed (or no children)
- Combine insights from children
- Update this node's _node.md with full understanding

### EXITING
- Pop from stack
- Bubble up summary to parent
- Mark as visited

---

## Resume Protocol

When `/legacy` starts:
1. Read _traverse.md
2. Find current position (top of stack)
3. Check phase
4. Continue from that phase

If interrupted mid-phase:
- Re-enter same phase (idempotent operations)

---

*Updated by /legacy recursive traversal*
