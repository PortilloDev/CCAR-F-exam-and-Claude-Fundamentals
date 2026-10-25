# Lecture 52: Lab 3a: PreToolUse and PostToolUse Hooks for Support Agent

**Section:** Agentic Architecture  
**Lecture ID:** 58217519

---

## Transcript

### Lab structure

**Part A (this lab):** Build the MCP server + hooks.  
**Part B (next lab):** Integrate into the Agent SDK.

**MCP tools:**
- `get_customer(email | phone | customer_id)` — returns verified customer ID only when exactly one account matches.
- `lookup_order(order_id)` — returns order details.
- `process_refund(order_id, amount)` — processes a refund.
- `escalate_to_human(EscalationRecord)` — structured handoff.

---

### Structured tool responses

Every tool returns structured output on both success and failure:

```python
# Failure
{"is_error": True, "error_type": "validation", "message": "...", "next_step": "..."}

# Business error
{"is_error": True, "error_type": "business",
 "message": "Order delivered 67 days ago — outside the 30-day refund window.",
 "next_step": "Offer store credit or escalate for a manager exception."}

# Success
{"is_error": False, "customer_id": "CUST-123", "verified": True, ...}
```

`get_customer`: returns `verified_customer_id: None` when multiple accounts match (asks for more info rather than guessing).

---

### PostToolUse: field trimming

`lookup_order` returns 33 fields. The hook trims it to 5 key fields before Claude sees it:

```python
async def normalise_tool_output(input_data, tool_use_id, context):
    fields_to_keep = {"order_id", "status", "amount", "placed_at", "customer_id"}
    trimmed = {k: v for k, v in input_data["tool_response"].items() if k in fields_to_keep}
    return {"hookSpecificOutput": {"hookEventName": "PostToolUse", "updatedToolOutput": trimmed}}
```

**Result:** 33 fields → 5 fields → fewer tokens, less noise for Claude.

---

### PreToolUse: customer verification gate

```python
GATED_TOOLS = {"lookup_order", "process_refund"}
verified_customers = {}

async def require_verification(tool_name, tool_input, context):
    if tool_name not in GATED_TOOLS:
        return {}
    customer_id = tool_input.get("customer_id")
    if customer_id not in verified_customers:
        return {
            "hookSpecificOutput": {
                "hookEventName": "PreToolUse",
                "permissionDecision": "denied",
                "permissionDecisionReason": "Call get_customer first to verify the customer identity.",
            }
        }
    return {}

async def record_verification(tool_name, tool_output, context):
    if tool_name == "get_customer" and tool_output.get("verified"):
        verified_customers[tool_output["customer_id"]] = True
    return {}
```

---

### PreToolUse: refund limit gate

```python
REFUND_LIMIT = 500.0

async def enforce_refund_limit(tool_name, tool_input, context):
    if tool_name != "process_refund":
        return {}
    if tool_input.get("amount", 0) <= REFUND_LIMIT:
        return {}
    return {
        "hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "denied",
            "permissionDecisionReason": f"Refund exceeds £{REFUND_LIMIT} limit. Use escalate_to_human.",
        }
    }
```
