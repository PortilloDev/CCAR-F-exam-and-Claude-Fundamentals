# Lecture 26: MCP: Structured Error Responses

**Section:** Tools and MCP  
**Lecture ID:** 58188307

---

## Transcript

When an error occurs in an MCP server, it must be communicated clearly so the model can decide what to do next.

### The `isError` flag

FastMCP sets `isError: true` automatically when your tool raises an exception. If you build without a framework, you must set this flag manually in the MCP response. The Agent SDK handles this automatically.

### Rich error format (what to return)

❌ **Bad:** `"Operation failed"` — too generic, model cannot act.

✅ **Good:**
```json
{
  "content": [
    {
      "error_category": "transient",
      "retriable": true,
      "message": "GitHub API timeout after 30s",
      "query": "list open PRs for repo X"
    }
  ]
}
```

---

### Four error categories

| Category | Cause | Retriable? | Model action |
|----------|-------|-----------|--------------|
| **Transient** | Timeout, rate limit | ✅ Yes (with backoff) | Retry with increasing wait |
| **Validation** | Bad input format | ✅ Yes (fix input) | Correct the arguments and retry |
| **Business** | Policy/threshold triggered | ❌ No | Explain the policy to the user or offer an alternative |
| **Permission** | Agent lacks access rights | ❌ No | Escalate to a human reviewer |

**Backoff:** For transient errors, increase the wait time with each retry (e.g., 1s → 2s → 4s).

---

### Business error example

```json
{
  "error_category": "business",
  "retriable": false,
  "message": "Refund window expired: purchase was 45 days ago, policy allows 30 days",
  "alternative": "Offer store credit or escalate for a manager exception"
}
```

---

### Critical rule: empty result ≠ error

| Situation | Correct response |
|-----------|-----------------|
| Search returns 0 results | Normal result — `isError: false`, empty list |
| API timeout | Error — `isError: true`, category: transient |
| Permission denied | Error — `isError: true`, category: permission |

**Why this matters:**
- Treating empty as failure → model retries unnecessarily (wastes tokens, loops).
- Treating failure as empty → model assumes no results when the tool actually broke (misses valid data).

**Memorize for the exam:** Always set `isError: true` with an error category for real failures. Never use an empty result to signal an error.
