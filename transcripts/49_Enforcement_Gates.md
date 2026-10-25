# Lecture 49: Enforcement & Handoff: Gates

**Section:** Agentic Architecture  
**Lecture ID:** 58217019

---

## Transcript

### Gate types

| Type | What it checks | Example |
|------|---------------|---------|
| **Value gate** | An argument value in the tool's input | Block refunds > $500 |
| **Sequence gate** | Whether a required prior tool was already called | Block `process_refund` until `get_customer` ran |

---

### Value gate — refund limit example

```python
REFUND_LIMIT = 500.0

async def refund_gate(tool_name, tool_input, context):
    if tool_name != "process_refund":
        return {}  # no opinion for other tools

    amount = tool_input.get("amount", 0)
    if amount <= REFUND_LIMIT:
        return {}  # allow

    return {
        "hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "denied",
            "permissionDecisionReason": (
                f"Refund of ${amount} exceeds limit ${REFUND_LIMIT}. "
                "Use the escalate_to_human tool."
            ),
        }
    }
```

---

### Sequence gate — require `get_customer` before `process_refund`

```python
GATED_TOOLS = {"lookup_order", "process_refund"}
customer_id_state = {}

async def require_customer_id(tool_name, tool_input, context):
    if tool_name not in GATED_TOOLS:
        return {}
    if "customer_id" not in customer_id_state:
        return {
            "hookSpecificOutput": {
                "hookEventName": "PreToolUse",
                "permissionDecision": "denied",
                "permissionDecisionReason": "Call get_customer first to establish the customer ID.",
            }
        }
    return {}

async def record_customer_id(tool_name, tool_output, context):
    if tool_name == "get_customer":
        customer_id_state["customer_id"] = tool_output.get("customer_id")
    return {}

options = ClaudeAgentOptions(
    hooks={
        "pre_tool_use":  [{"hooks": [require_customer_id]}],
        "post_tool_use": [{"hooks": [record_customer_id]}],
    }
)
```

**Flow:**
1. Claude tries `process_refund` → denied (no customer ID on record).
2. Claude calls `get_customer` → post-hook records the ID.
3. Claude retries `process_refund` → allowed.

---

### Manipulating tool input with a gate

```python
return {
    "hookSpecificOutput": {
        "hookEventName": "PreToolUse",
        "permissionDecision": "approved",
        "updatedInput": {**tool_input, "currency": "USD"},  # enforce currency
    }
}
```

`updatedInput` replaces the tool's input arguments before the tool runs.
