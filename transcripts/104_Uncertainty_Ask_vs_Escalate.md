# Lecture 104: Uncertainty: Ask vs. Escalate

**Section:** Context and Reliability  
**Lecture ID:** 58217291

---

## Transcript

### Three-way decision when agent cannot proceed

```
Agent is stuck — what to do?
├── Is the missing data in a database? → Tool call (next turn)
├── Is the data ambiguous (e.g., multiple customer records)?
│     └── Can the customer clarify? → Ask the customer
│         (Only if it's something the customer can know — not an internal ID)
└── Is it a policy gap (no business rule exists)? → Escalate to human
```

---

### "Ask" examples

| Situation | Who has the answer | Action |
|-----------|-------------------|--------|
| 3 customer accounts match the name | The customer | Ask: "Can you confirm your email address?" |
| Contract ID needed | The customer | Ask: "Please provide your contract ID" |
| Internal system ID | Nobody outside the system | Do NOT ask customer; use tool lookup or escalate |

---

### Escalate examples

| Situation | Why escalate |
|-----------|-------------|
| Customer requesting competitor price match | Policy gap — no business rule covers this |
| Explicitly asks for a manager | Clear escalation signal |
| Two resolution attempts failed | Agent has exhausted its options |

---

### Never use heuristics for identity resolution

If 3 customer accounts match:
- "Take the one with most orders" → wrong
- "Take closest name match" → wrong  
- Self-confidence score → wrong

The customer must confirm — or escalate to a human who can verify identity through another channel.

---

### Tone vs. policy (repeated — exam critical)

| Customer state | Policy covered | Decision |
|---------------|----------------|---------|
| Angry | YES | Resolve |
| Calm | NO | Escalate |
| Asking for manager | Either | Escalate |
