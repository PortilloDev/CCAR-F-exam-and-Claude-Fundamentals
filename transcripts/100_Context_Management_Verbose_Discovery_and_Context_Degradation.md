# Lecture 100: Context Management: Verbose Discovery & Context Degradation

**Section:** Context and Reliability  
**Lecture ID:** 58217283

---

## Transcript

### Codebase discovery pattern (exam)

```
1. Glob — list all files (filter by pattern: *.ts, *test*, etc.)
2. Grep — find class/function definitions and exports
3. Grep — find where those exports are imported in other files
```

This three-step pattern reveals recursive dependencies between modules without reading every file in full.

---

### Context degradation — two causes

| Cause | Effect |
|-------|--------|
| Context window fills up (80-90%) | Attention dilutes — quality drops |
| Auto-compaction triggers | Some information is lost — accuracy drops further |

Both effects compound: a long single-agent session progressively degrades in quality even before it hits the limit.

---

### Solutions

| Approach | When to use |
|----------|-------------|
| **Subagents** (fresh context per task) | Preferred — delegate verbose discovery tasks |
| **Scratchpad files** | Let agent export findings; re-read after degradation |
| **New session with summary** | Last resort — only works if summary is created before degradation is severe |

---

### Subagent strategy for large codebase

```
Coordinator
├── Subagent A: scan test files (fresh context)
├── Subagent B: refund flow modules (fresh context)
└── Subagent C: authentication logic (fresh context)
```

Each subagent brings back a summarized report — the coordinator's context stays clean.

---

### Decision: when to use subagents vs. compact

| Situation | Action |
|-----------|--------|
| Haven't started discovery yet | Delegate to subagent (fresh context from the start) |
| Already did discovery, context at 70-80% | Compact now with a directed instruction, then continue |
| Already degraded | Start new session with best available summary |

---

### Key exam point

**Never start implementation with 70-80% context already used.** The subagent approach keeps the main agent's context lean for the actual work.
