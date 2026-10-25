# Lecture 15: Tool-Use Loop: Complete Cycle

**Section:** Tools and MCP  
**Duration:** ~7 min | **Lecture ID:** 58184789

---

## Transcript

All right, next we're looking at the loop, which is responsible for tool calling. So we'll look at how Claude actually communicates which tool it wants to call, how we can successfully dispatch this wish to call a tool, get the result, hand it back over to Claude, and we will also look at a code snippet of how you could actually build this yourself.

**Three components of the loop:**
1. **The Loop** (orchestrator) — code you run yourself, or the Agent SDK, or Claude Code (which does this under the hood)
2. **Claude** — the LLM accessed via the Messages API
3. **The Tool** — a code block containing the logic to run

**The flow:**
1. Send a message to Claude with available tools listed
2. Claude responds with `stop_reason: tool_use` — it wants to call a tool
3. Your code extracts: `id` (unique call identifier), `name` (tool to call), `input` (parameters)
4. Your code dispatches the call to the actual tool function
5. Result comes back (as string, or error message)
6. You send back the tool result with the same `id` mapped to the original request
7. Claude observes the result and either calls another tool or returns `stop_reason: end_turn`

**Key rules when sending tool results back:**
- Role is `user` (not `assistant`, and there is no `tool` role)
- Always resend the full tool schemas list (API is stateless)
- Always include the original `tool_use_id` so Claude can match result to request

**Chat history structure for tool calls:**
- Store the assistant's `tool_use` block in chat history
- When sending back results, add a `user` message with `content` type `tool_result` containing: `tool_use_id`, `content` (the result), optionally `is_error: true`

**The while loop pattern:**
```python
while True:
    response = call_agent(messages, tools)
    messages.append(assistant_message)
    if response.stop_reason == "tool_use":
        # dispatch tools, collect results, append to messages
        continue
    else:
        break  # end_turn or max_tokens
return messages
```

**Important:** Use `stop_reason` as the main indicator — `tool_use` means more tools are needed, anything else exits the loop. This handles both `end_turn` and `max_tokens` correctly.
