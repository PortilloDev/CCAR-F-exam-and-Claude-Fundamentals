# Lecture 106: Uncertainty: Provenance & Conflicting Sources

**Section:** Context and Reliability  
**Lecture ID:** 58217299

---

## Transcript

### Structured claim with provenance

Each claim from a subagent should include metadata — not just the value:

```json
{
  "claim": "AI music market is approximately $3.2 billion",
  "source_name": "Music Tech Report 2024",
  "url": "https://...",
  "excerpt": "Revenue from AI-generated music reached $3.2B...",
  "publication_date": "2024-03",
  "collection_method": "vendor revenue survey",
  "how_stated": "estimated"
}
```

**Why structured:** prose summaries lose these details across compaction passes. After 3 passes: only the number survives. Structured JSON survives intact.

---

### Conflicting sources

**Example:** AI music share on a platform:
- Platform self-reported: 12%
- Independent survey: 8%

### Anti-patterns

| Anti-pattern | Problem |
|-------------|---------|
| Average the two values | Hides the conflict |
| Pick one and add a footnote | Hides the conflict, adds noise |
| Look for more sources | Complicates without resolving |
| Ignore the conflict | Worst option |

### Correct approach

Keep both values in the structured output. Add a conflict flag:

```json
{
  "field": "ai_music_share",
  "values": [
    {"value": "12%", "source": "Platform report"},
    {"value": "8%",  "source": "Independent survey"}
  ],
  "conflict_detected": true,
  "possible_explanation": "Different collection methods and time periods"
}
```

---

### Agent roles

| Role | Responsibility |
|------|---------------|
| **Analysis subagent** | Annotate claims with provenance metadata |
| **Coordinator** | Reconcile — build union, flag conflicts, never resolve by averaging |
