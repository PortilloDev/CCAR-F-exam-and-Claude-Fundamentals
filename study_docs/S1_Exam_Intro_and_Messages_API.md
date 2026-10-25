# Study Doc — Section 1: Exam Intro & Claude Fundamentals

**Domain:** All (foundational concepts)  
**Files:** 02–11

---

## The Exam — Key Facts

| Item | Detail |
|------|--------|
| Questions | 60 (single choice or multiple response) |
| Scenarios | 4 of 6 randomly selected per exam |
| Pass score | 720 / 1000 |
| Time | 120 minutes (2 min per question) |

### 6 scenarios (4 will appear on your exam)

1. Support agent (Agent SDK + MCP tools + escalation)
2. Claude Code (CLI, CLAUDE.md, skills, plan mode)
3. Multi-agent research system (coordinator + subagents + grounded facts)
4. Agent SDK with built-in tools (Grep, Glob) + MCP servers
5. Claude Code in CI/CD headless mode (PR reviews)
6. Messages API for unstructured → structured data extraction

### Domain weights (approximate priority)

1. Agentic Architecture (highest)
2. Claude Code configuration
3. Prompt Engineering
4. Tool Design + MCP
5. Context Management + Reliability

---

## Deterministic vs. Probabilistic Controls

**Core architect decision:** what goes in the model vs. what goes in code?

| Scenario | Control type |
|----------|-------------|
| Customer intent understanding, edge-case reasoning | Model (probabilistic) |
| Money, legal, access permissions, safety | **Code (deterministic)** |

**Rule of thumb:** if money, law, or harm is in the picture → add deterministic code, don't just improve the prompt.

**Example:** agent skipping customer ID verification in 12% of cases despite prompt instructions → fix with a `pre_tool_use` hook that checks identity before allowing the refund tool call.

**Why prompts alone fail:** adding more instructions or few-shot examples reduces error rate but never to zero. For sensitive operations, non-zero failure rate is unacceptable.

---

## Messages API — Core Concepts

### Stateless by design

- **No session ID.** Every API call is independent.
- **No memory.** Must resend full chat history every call.
- The `id` field in the response is for debugging only — NOT a session identifier.

**EXAM TRAP:** Looking for a session ID in the Messages API → there is none.

### Messages array structure

```python
messages = [
    {"role": "user",      "content": "Define quantum computing."},
    {"role": "assistant", "content": "Quantum computing uses..."},
    {"role": "user",      "content": "Write another sentence."},
]
```

You manage this list. Append user messages and assistant responses after each turn. Resend the full array every call.

**EXAM TRAP:** Forgetting to include `system` prompt on every call — it's stateless, must be re-sent every time.

### Response structure

```python
response.content  # list — filter by type, never index directly
response.stop_reason  # check this FIRST
response.usage  # input_tokens, output_tokens
```

### stop_reason values

| Value | Meaning |
|-------|---------|
| `end_turn` | Complete response — normal end |
| `tool_use` | Claude is waiting for a tool result (your code must call the tool) |
| `max_tokens` | **Response truncated** — increase max_tokens and retry |
| `stop_sequence` | Matched a predefined stop string |

**EXAM TRAP:** Don't look at the text content to determine if the run is complete — check `stop_reason` first.

### Key request parameters

| Parameter | Notes |
|-----------|-------|
| `max_tokens` | Hard limit on output; too small = truncation; pick generously |
| `temperature` | 0 = deterministic; 1 = creative/random |
| `thinking` | Adaptive mode (latest); budget_tokens deprecated |
| `tool_choice` | Force which tools can be called |
| `stream` | Streaming tokens as generated |

### Prompt caching

Saves up to 90% cost and reduces latency by caching known context. Does NOT give the model memory — you still must send the `messages` array.

### Managed options (Claude Code CLI, Agent SDK)

Both manage chat history and compaction automatically. The raw Messages API requires you to manage the history yourself.
