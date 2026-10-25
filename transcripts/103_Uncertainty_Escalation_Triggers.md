# Lecture 103: Uncertainty: Escalation Triggers

**Section:** Context and Reliability  
**Lecture ID:** 58217289

---

## Transcript

### Over-escalation vs. under-escalation

| | Covered by policy | Uncovered by policy |
|--|------------------|---------------------|
| **Agent resolves it** | ✅ Correct | ❌ Under-escalation (agent invents policy) |
| **Agent escalates** | ❌ Over-escalation (wasted human time) | ✅ Correct |

**Tone does not drive escalation.** An angry customer asking for something within policy → agent resolves it. A calm customer asking for something outside policy → agent escalates immediately.

---

### Anti-patterns (exam)

| Anti-pattern | Problem |
|-------------|---------|
| Self-reported confidence score | Not grounded in labeled data — unreliable |
| Sentiment analysis as escalation signal | Sentiment ≠ policy coverage |
| Trained classifier for escalation decisions | Over-engineering — use criteria + few-shot prompts first |

---

### Correct approach: explicit criteria + few-shot examples

```
Escalate now if:
- Customer explicitly asked for a human
- Request is not covered by policy
- Two resolution attempts left the case unresolved

Resolve (even if customer is angry) if:
- Policy covers the request AND
- The agent's tools can complete it
```

**Few-shot examples:**

| Customer message | Action | Reason |
|-----------------|--------|--------|
| "This is outrageous, the quality is terrible." | Resolve — offer replacement | Sentiment high, but covered by policy |
| "A rival is 30% cheaper. Match it." | Escalate | Policy is silent on competitor pricing |

---

### Key exam rule

Escalation criteria must be **policy-based** (what the agent is authorized to handle), not sentiment-based (how the customer feels).
