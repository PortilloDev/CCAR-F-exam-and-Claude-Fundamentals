# Lecture 50: Enforcement & Handoff: Structured Handoff

**Section:** Agentic Architecture  
**Lecture ID:** 58217023

---

## Transcript

### The problem with unstructured handoffs

When an agent escalates to a human with just a text summary, the human:
- May miss the customer ID, root cause, or actions already taken.
- Cannot prioritize the ticket based on escalation reason.
- Must re-read and re-investigate before acting.

### Solution: structured escalation tool

Use `escalate_to_human` as a **tool with a typed input schema** — not a free-text message:

```python
class EscalationRecord(BaseModel):
    customer_id: str
    root_cause: str
    actions_taken: list[str]
    amount_in_scope: float
    recommended_action: str
    escalation_reason: str   # "limit_exceeded" | "policy_exception" | "customer_request" | "repeated_failure"
```

A `PreToolUse` hook can validate that all required fields are present before the escalation is accepted.

---

### What to include in a handoff

| Field | Purpose |
|-------|---------|
| `customer_id` | Unique identifier (not just a name) |
| `root_cause` | What went wrong |
| `actions_taken` | What the agent already tried |
| `amount_in_scope` | The dollar/pound amount at stake |
| `recommended_action` | Agent's best suggestion |
| `escalation_reason` | Why automation stopped |

---

### When to escalate vs. continue

**Escalate to human when:**
- Customer explicitly requests a human or manager.
- Policy exception is needed (competitor price match, etc.).
- Refund exceeds the approved limit.
- Agent has retried repeatedly without resolution.

**Keep working when:**
- Case involves image analysis the agent can perform.
- Customer is upset but the case is procedurally solvable.
- Multi-step process is complex but all necessary tools are available.

**Rule:** Don't jump to human escalation too quickly. Escalate meaningfully, not defensively.
