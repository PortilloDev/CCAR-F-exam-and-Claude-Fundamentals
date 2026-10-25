# Lecture 101: Context Management: Recovering State

**Section:** Context and Reliability  
**Lecture ID:** 58217285

---

## Transcript

### Two files for recovery

| File | Format | Purpose |
|------|--------|---------|
| **manifest.json** | Structured JSON | Track progress — which queries ran, which topics are covered, which have gaps, which subagents are done/in-progress/not-started |
| **scratchpad** | Markdown | Store findings and learnings from previous runs (prose, not status) |

---

### manifest.json example

```json
{
  "status": "in_progress",
  "topics": {
    "ai_music_market": {
      "status": "completed",
      "queries_run": ["market size 2024", "growth rate"],
      "gaps": ["distribution", "licensing"]
    },
    "competitor_analysis": {
      "status": "not_started"
    }
  }
}
```

---

### Recovery workflow

```
Coordinator restarts
  └── Read all subagent manifest files
        └── Skip subagents marked "completed"
              └── Restart remaining subagents
                    └── Inject their manifest file as system prompt context
                          → They resume from last known state
```

---

### Resume vs. restart decision

| Situation | Action |
|-----------|--------|
| Context is still valid (few files changed) | Resume — load manifest + scratchpad + continue chat history |
| Too many files changed around the agent | Restart from scratch — manifest state is stale |

---

### Testing the recovery pattern

1. Start multi-agent run.
2. Interrupt on purpose.
3. Restart coordinator.
4. Coordinator reads manifests, skips completed subagents, resumes others.
5. Verify work resumes correctly from last checkpoint.
