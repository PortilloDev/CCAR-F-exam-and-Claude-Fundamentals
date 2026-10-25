# Lecture 33: Agent SDK: Conversations & Sessions

**Section:** Agentic Architecture  
**Lecture ID:** 58188343

---

## Transcript

### Managing chat history: Client SDK vs. Agent SDK

| Approach | Who manages history? | Resumable? | Use when |
|----------|---------------------|-----------|----------|
| Client SDK (`messages.create`) | You — manual array | No | Maximum control, custom logic |
| Agent SDK `query()` | SDK | No (one-shot) | Simple one-off tasks |
| Agent SDK `ClaudeSDKClient` | SDK — stored on disk | Yes | Chat interfaces, long-running agents |

---

### ClaudeSDKClient (session-aware)

```python
from claude_agent_sdk import ClaudeSDKClient

async with ClaudeSDKClient(options=options) as client:
    # First turn
    await client.query("Summarize the auth module")
    response = await client.receive_response()
    print(response.message)

    # Second turn — SDK remembers the conversation automatically
    await client.query("Which of those functions lack tests?")
    response = await client.receive_response()
    print(response.message)
```

**Session ID** is stored locally. Use `resume=<session_id>` in `ClaudeAgentOptions` to continue from where you left off.

---

### Interrupting a live session

```python
async with ClaudeSDKClient(options=options) as client:
    await client.query("Refactor the payment module")
    await asyncio.sleep(30)
    
    await client.interrupt()           # stop the current run
    await drain_stop_turn(client)      # consume the terminal_reason message
    
    await client.query("Focus only on the checkout flow")  # fresh restart
```

`result.terminal_reason` is set whenever a session is interrupted.

---

### Auto-compaction

When the context window fills up (prompts + tool results + replies), auto-compaction triggers a **pre-compact hook**:
1. Summarizes the full context window.
2. Replaces the old content with the summary.
3. Frees up space for new content.

This is a built-in feature in both Claude Code and the Agent SDK — no extra setup required.

---

### query() vs. ClaudeSDKClient

| | `query()` | `ClaudeSDKClient` |
|--|-----------|-----------------|
| Memory across turns | ❌ | ✅ |
| Session resumption | ❌ | ✅ |
| Interrupts | ❌ | ✅ |
| Best for | One-off tasks | Chat / long-running agents |
