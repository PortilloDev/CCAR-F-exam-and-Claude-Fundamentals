# Lecture 44: Orchestration: Recover Locally & Report What Remains

**Section:** Agentic Architecture  
**Lecture ID:** 58188391

---

## Transcript

### The failure communication problem

When a subagent's tool call fails, an **empty result** looks like success to the coordinator — it assumes "no data found" rather than "tool broken." The subagent must explicitly report failures in a structured way.

---

### Error handling at each layer

| Layer | When to retry | When NOT to retry |
|-------|--------------|------------------|
| Tool → Subagent | Transient (timeout, rate limit) — retry with exponential backoff | Validation error (fix input) / Business/Permission error |
| Subagent → Coordinator | Report partial results + failure details | Never return empty on failure |

**Backoff:** Exponential wait increase with each retry (e.g., 1s → 2s → 4s). Set a retry limit (2-3 attempts).

---

### Structured failure report from subagent to coordinator

```json
{
  "failure_type": "timeout",
  "attempted_query": "music generation AI adoption 2024",
  "partial_results": {
    "music_generation": "partial report available"
  },
  "alternative_approaches": [
    "Retry with a shorter time window",
    "Search with query: 'AI music tools market share'"
  ],
  "coverage_impact": "Missing music generation data; report will lack quantitative claims for this segment"
}
```

---

### What happens without proper failure reporting

| Bad behavior | Consequence |
|-------------|------------|
| Generic status message | Partial results and context are lost |
| Empty result on failure | Coordinator treats it as success — data silently missing |
| Exception breaks workflow | Workflow dies even when partial results could allow continuation |

---

### Three recovery strategies

1. **Retry with backoff** — for transient/timeout errors.
2. **Modify input and retry** — for validation errors (fix the schema mismatch).
3. **Proceed with partial results** — when retries/alternatives fail; coordinator continues with what's available.

**Rule:** Empty result = successful call with no data. Errors always use `is_error: true` + structured metadata.
