# Lecture 47: Enforcement & Handoff: Hooks

**Section:** Agentic Architecture  
**Lecture ID:** 58217009

---

## Transcript

### Why hooks for enforcement?

Prompts, few-shot examples, and clever wording can never reduce tool-call failure rate to **zero** — they are probabilistic. Hooks provide **deterministic control**: if the hook denies, the tool is always blocked.

---

### Hook anatomy

```python
async def my_hook(tool_name: str, tool_input: dict, hook_name: str):
    # ... logic ...
    return {
        "hookSpecificOutput": {
            "hookEventName": "PreToolUse",      # or PostToolUse
            "permissionDecision": "denied",      # or "approved" or "ask"
            "permissionDecisionReason": "Reason message for the model",
        }
    }
    # return {} or return None → no opinion, tool proceeds
```

---

### Registering hooks in ClaudeAgentOptions

```python
options = ClaudeAgentOptions(
    hooks={
        "pre_tool_use": [
            {
                "matcher": {"name": "mcp.support.*"},   # wildcard for all support tools
                "hooks": [my_pre_hook],
                "timeout": 30,
            }
        ],
        "post_tool_use": [
            {"hooks": [my_post_hook]},   # no matcher = runs for ALL tools
        ],
    }
)
```

**No matcher** = hook runs for every tool call.

---

### Hook precedence

- Hooks run **before** permission modes — they can override even `bypassPermissions`.
- Even auto-approved tools (in `allowed_tools`) still trigger `pre_tool_use` and `post_tool_use` hooks.
- The `can_use_tool` hook is the only one auto-approved tools skip.

---

### Hooks vs. prompts

| Concern | Use hooks | Use prompts |
|---------|-----------|-------------|
| Business rule (must verify before refund) | ✅ | ❌ (probabilistic) |
| Tone and formatting | ❌ | ✅ |
| Blocking specific tool patterns | ✅ | ❌ |
| Nuanced scenario exceptions | ❌ (hooks are always-on) | ✅ |

**Tradeoff:** Hooks have no exceptions — they fire on every pattern match. They also require code maintenance.
