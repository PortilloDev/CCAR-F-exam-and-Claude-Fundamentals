# Lecture 43: Orchestration: Subagent Routing

**Section:** Agentic Architecture  
**Lecture ID:** 58188389

---

## Transcript

### Cost of unnecessary subagent calls

Each subagent call = tokens consumed + latency + cost. The coordinator must:
- Call the right subagents in the right sequence.
- Use the **minimum necessary number of calls**.
- Not call subagents for trivial work the coordinator can handle itself.
- Not trigger a full synthesis pipeline for a small follow-up detail.

---

### The incomplete decomposition problem

The coordinator may forget entire topic categories. Example:
- Task: "Research AI and the creative industries."
- Coordinator decomposes into: digital art, graphic design, photography → triggers synthesis.
- **Missed:** music, writing, film.

**Fix options:**
1. Show the decomposition to the user first and get feedback before executing.
2. Use a **goal-based approach** with explicit quality criteria.

---

### Goal-based prompt pattern

```
Goal: Survey how AI is changing the creative industries across every major field.

Quality criteria:
- No major field (photography, music, writing, film) is left uncovered.
- Every claim carries a source and a date.
- Conflicting findings are both reported.
- Stop when ALL criteria hold — not after N fixed rounds.
- Maximum 5 research rounds.
```

**How it works:**
1. Coordinator runs research subagents.
2. Evaluates the result against the quality criteria.
3. If criteria are not met → triggers targeted follow-up research.
4. Repeats until criteria hold or max rounds reached.

**Benefit:** Guarantees coverage; the coordinator self-corrects instead of the developer hardcoding every step.
