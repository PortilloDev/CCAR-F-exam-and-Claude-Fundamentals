# Lecture 56: Lab 3b: Integrate Hooks into Support Agent with Agent SDK

**Section:** Agentic Architecture  
**Lecture ID:** 58217545

---

## Transcript

### Complete agent setup

```python
options = ClaudeAgentOptions(
    model="claude-haiku-4-5",
    system_prompt=support_system_prompt,
    mcp_servers=[support_mcp_server],
    tools=["Task"],
    allowed_tools=["mcp.support.*"],
    hooks={
        "pre_tool_use": [
            {"matcher": {"name": "mcp.support.lookup_order"},
             "hooks": [require_verification]},
            {"matcher": {"name": "mcp.support.process_refund"},
             "hooks": [require_verification, enforce_refund_limit]},
        ],
        "post_tool_use": [
            {"matcher": {"name": "mcp.support.get_customer"},
             "hooks": [record_verification]},
            {"hooks": [normalise_tool_output]},   # all tools
        ],
    }
)
```

---

### Without hooks vs. with hooks

| Scenario | No hooks | With hooks |
|----------|---------|-----------|
| £650 refund request | Agent executes — refund goes through | Denied by `enforce_refund_limit`; `escalate_to_human` called |
| Lookup order before verifying identity | Allowed | Denied by `require_verification` |
| `lookup_order` with 33 fields | All 33 sent to Claude | Trimmed to 5 fields |

---

### Lab test cases

| Case | Expected outcome |
|------|----------------|
| Broken item, £650 refund | Gate blocks refund → structured escalation |
| Policy request (competitor price) | Business error → escalation |
| Customer won't share email | Verification fails → cannot proceed |
| Customer requests a human | Agent recognizes signal → escalates |
| Two issues (double charge + missing delivery) | Decomposed into two issues, worked in parallel |

---

### Key takeaways

- Hooks make business policies **deterministic** — prompts alone are probabilistic and will eventually fail.
- `PreToolUse` gates enforce sequences (verify before refund) and value limits (£500 cap).
- `PostToolUse` reduces token usage (33 → 5 fields) and normalizes formats.
- `escalate_to_human` as a typed tool ensures human agents always receive a structured, actionable record.
- Model tier matters: Haiku 4.5 handles simple cases well; Sonnet/Opus performs better on complex multi-issue tickets.
