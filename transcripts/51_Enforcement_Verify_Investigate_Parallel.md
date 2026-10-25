# Lecture 51: Enforcement & Handoff: Verify & Investigate in Parallel

**Section:** Agentic Architecture  
**Lecture ID:** 58217027

---

## Transcript

### Multi-issue tickets

Customers often raise multiple concerns in one message:

> "You charged me twice on invoice #INV-4892, and the replacement for order #ORD-7721 never came through — tracking says it was delivered."

**Failure modes without decomposition:**
- Agent addresses only one concern and forgets the other.
- Agent sends two separate replies that don't relate to each other.
- Agent escalates too quickly without investigating.

---

### Correct approach: decompose into structured issues

**Step 1:** Break the message into discrete issues with enriched data.

```json
[
  {
    "issue_id": "ISS-001",
    "type": "duplicate_charge",
    "invoice_number": "INV-4892",
    "amount": 89.99,
    "claim": "Charged twice",
    "status": "open"
  },
  {
    "issue_id": "ISS-002",
    "type": "missing_delivery",
    "order_id": "ORD-7721",
    "shipment_number": "SHP-5530",    // enriched via a DB lookup during decomposition
    "claim": "Replacement never arrived despite tracking showing delivered",
    "status": "open"
  }
]
```

**Step 2:** Store case facts in a **separate file** outside the agent's context window (e.g., `case_facts.json` in the workspace). This persists across turns without bloating context.

**Step 3:** Work each issue in parallel (one subagent per issue) → aggregate into one resolution response.

---

### Single-agent alternative

If sub-agents are not available, steer one agent with a prompt that explicitly separates issues and tracks them by ID. The key is still to create the structured issue breakdown first.

---

### Outcome tracking

Track per issue:
- `resolved` — agent fully handled it.
- `partial` — agent made progress but couldn't complete.
- `escalated` — human handoff required for this issue specifically.

This allows partial escalation: some issues resolved automatically, others handed off — all in one customer-facing response.
