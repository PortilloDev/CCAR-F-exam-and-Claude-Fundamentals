# Lecture 39: Orchestration: Multi-Agents Coordinator & Subagents

**Section:** Agentic Architecture  
**Lecture ID:** 58188369

---

## Transcript

### Hub-and-spoke architecture (exam critical)

The most important multi-agent pattern for the exam. The **coordinator** sits at the center:

```
         Coordinator
        /     |      \
  Search  Analysis  Synthesis
```

- **Coordinator responsibilities:**
  1. Decompose the incoming task into subtasks.
  2. Decide which subagent handles each subtask.
  3. Write a complete brief (initial prompt) for each subagent.
  4. Aggregate and validate results.
  5. Handle errors and trigger retries.
  6. Produce the final answer to the user.

- **Subagents do NOT spawn other subagents** (design choice, not a technical limit). The exam focuses on the coordinator-only spawning pattern.
- Subagents always start with a **fresh context window** — the coordinator's context is never shared.

---

### Hub-and-spoke vs. spoke-to-spoke

| Pattern | Communication | Coordinator role |
|---------|--------------|-----------------|
| **Hub-and-spoke** (exam focus) | Only coordinator ↔ subagent | Always central |
| Spoke-to-spoke | Agents talk to each other | Less central — harder to manage |

---

### Benefits of the coordinator pattern

1. **Observability** — coordinator sees all subagent outputs in one place.
2. **Central error policy** — single place to define retry/escalation logic.
3. **Controlled information flow** — clear, auditable paths between agents.

---

### Information flow example

User → Coordinator → Search agent (finds FIFA 2026 info, returns with sources)
Coordinator → Synthesis agent (writes report from search results)
Coordinator → User (final answer)

On follow-up: Coordinator → Search agent again (history preserved in coordinator only).

---

### Subagent design principles

- **Bound its role** — only the tools it needs for its specific job.
- **Structured output** — clearly define the data points and format the subagent must return.
- **Noise reduction** — keep the parent's context window clean; only return what the coordinator needs.
