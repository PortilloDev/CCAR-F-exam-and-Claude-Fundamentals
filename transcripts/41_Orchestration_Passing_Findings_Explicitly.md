# Lecture 41: Orchestration: Passing Findings Explicitly

**Section:** Agentic Architecture  
**Lecture ID:** 58188379

---

## Transcript

### Why explicit passing matters

Subagents have **isolated context windows** — they cannot see the coordinator's history or each other's work. Everything a subagent needs must arrive in its initial prompt. Everything the coordinator needs must be in the subagent's final response.

---

### What flows INTO a subagent prompt

| Item | Notes |
|------|-------|
| The specific task/question | Clear, scoped instruction |
| Relevant findings from previous subagents | In structured format |
| Key decisions or constraints from the coordinator | Facts the subagent must respect |
| Files, error messages, or prior context | Anything the subagent cannot discover on its own |

**Rules are NOT passed via the coordinator prompt.** Rules belong in the subagent's own system prompt.

---

### Structured findings format (example)

```python
# Research subagent returns structured claims:
{
  "claim": "AI adoption in creative tools grew 47% in 2024",
  "source_name": "McKinsey AI Report 2024",
  "source_url": "https://mckinsey.com/ai-report-2024",
  "page": 12,
  "figure": "Figure 3.2",
  "publication_date": "2024-03-15"
}
```

Coordinator then passes these into the synthesis agent's prompt:
```
Write the market section.

Findings from the research agent:
[claim: "AI adoption in creative tools grew 47% in 2024", source: McKinsey AI Report 2024, p.12, Fig 3.2, 2024-03-15]
[... more claims ...]

Return format: one section per topic. Every claim must keep its source citation.
```

---

### Subagent response design

- Return **key facts** in structured records (JSON or labeled fields).
- Include **source citations** (name, URL, date, location in source).
- Optionally add relevance scores.
- **Keep it concise** — the parent's context window must not be flooded.

---

### Protecting the coordinator's context window

Only return what the coordinator needs to aggregate or act on. Raw scraped text, full documents, or verbose tool results should be summarized inside the subagent before returning.
