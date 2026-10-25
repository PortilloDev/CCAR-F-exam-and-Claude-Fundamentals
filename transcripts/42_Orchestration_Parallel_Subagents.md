# Lecture 42: Orchestration: Parallel Subagents

**Section:** Agentic Architecture  
**Lecture ID:** 58188385

---

## Transcript

### Parallel vs. sequential subagent execution

**Parallel:** Total time = slowest subagent's time.  
**Sequential:** Total time = sum of all subagent times + turns between them.

Parallelization happens **automatically by default** — no code change needed. The trigger is in the **coordinator's prompt**.

---

### When to parallelize

| Parallel | Sequential |
|----------|-----------|
| Tasks are **independent** of each other | Output of one task is input to the next |
| Researching different topics simultaneously | Research → fact-check → synthesize |
| Fan-out of the same query to different sources | Any pipeline with dependencies |

---

### Triggering parallelization in the prompt

```
Research how AI is used in creative industries.

Launch ALL tasks in THIS SINGLE TURN:
- Subagent 1: Music and audio — sources stay in music domain only
- Subagent 2: Writing and publishing — sources stay in writing domain only
- Subagent 3: Film, TV, and animation — sources stay in film domain only
- Subagent 4: Visual art and design — sources stay in visual art domain only

Each brief names its own sources. Do not overlap research domains.
```

**Key phrase:** "all in this one turn" / "launch ALL tasks in THIS SINGLE TURN" → triggers fan-out.

---

### Avoiding duplicate sources (scope partitioning)

❌ **Bad scoping:** "AI in music" + "AI in music production" → overlapping domains → duplicate sources in synthesis.

✅ **Good scoping:** "AI in music and audio" + "AI in writing and publishing" + "AI in film, TV, animation" + "AI in visual art" → distinct, non-overlapping domains.

**Benefits of clean scope partitioning:**
- No duplicate sources reach the synthesis agent.
- Coordinator's context stays clean.
- Coordinator's own hallucination risk decreases.
- Context window is less likely to overflow.

---

### Inspecting parallelization

Each parallel subagent call appears as a separate `tool_use` block in the coordinator's response — multiple `Task` calls in one assistant message = they run in parallel.
