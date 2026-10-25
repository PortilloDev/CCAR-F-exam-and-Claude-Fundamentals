# Lecture 98: Context Management: The Levers

**Section:** Context and Reliability  
**Lecture ID:** 58217273

---

## Transcript

### Lever 1: Dedicated case-facts section

Place a structured block below the system prompt containing hard facts (order IDs, amounts, dates, statuses). Update it deterministically from tool results in code — not via prompting.

```
[System prompt]
[Case Facts — deterministic, code-controlled]
  order_id: ORD-12345
  charge_amount: $89.99
  discount: 15%
  return_window_closes: 2026-11-01
  refund_method: card
[Conversation history]
```

**Effect:** even if conversation history is compacted, case facts survive untouched.

---

### Lever 2: Post-tool-use trimming

Most external tools return 40+ fields; only 5 are relevant. Use a `post_tool_use` hook to strip irrelevant fields before they enter the context.

```python
# Hook: keep only what the agent needs
KEEP = {"order_id", "status", "total_amount", "return_eligible", "items"}
trimmed = {k: v for k, v in tool_result.items() if k in KEEP}
```

---

### Lever 3: Priority ordering within history

Structure conversation history so the most important content is at the top and bottom (where attention is highest):

```
1. Key findings (blocking issues, critical facts)
2. Detailed results and deep dives
3. Action items / next steps
```

---

### Lever 4: Subagent context isolation

A coordinator sending work to subagents should prepare a lean fact sheet — not the full context:

| Send | Don't send |
|------|-----------|
| Source location (URL/path) of a document | Full page content |
| The selected solution path | All candidate solution paths considered |
| Task-specific facts | Coordinator's chain of reasoning |

---

### Lever 5: Scratchpad files

Let the agent write findings to markdown files on disk, separate from the context window. Reference them from CLAUDE.md via import statements. A new session or a new subagent can read them without re-deriving the findings.
