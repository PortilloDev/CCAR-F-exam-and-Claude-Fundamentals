# Study Doc — Section 3: Agentic Architecture

**Domain:** Agentic Architecture (highest exam weight)  
**Files:** 29–56

---

## Agent SDK — Three Interaction Layers

| Layer | When |
|-------|------|
| **Messages API (Client SDK)** | Maximum control, custom loop, stateless calls |
| **Agent SDK (`query()` / `ClaudeSDKClient`)** | SDK manages history, hooks, subagents |
| **Claude Code** | Developer tool, CLAUDE.md, CLI / IDE |

---

## Agent SDK — The Loop as a Library

### ClaudeAgentOptions key fields

```python
from claude_agent_sdk import ClaudeAgentOptions, query

options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    system_prompt="You are a research assistant...",
    tools=["Read", "Grep", "WebSearch"],
    allowed_tools=["Read", "Grep"],       # pre-approved (no prompt)
    disallowed_tools=["mcp.github.*"],    # always blocked
    permission_mode="dontAsk",
    max_turns=10,
    cwd="/path/to/project",
)

result = await query(prompt="Find all API endpoints", options=options)
```

### Result message subtypes

| Subtype | Meaning |
|---------|---------|
| `text` | Claude's final text response |
| `tool_use` | Tool call Claude wants to make |
| `tool_result` | Result returned after a tool call |
| `error` | Tool or runtime error |

---

## Built-In & Custom Tools

### Built-in tools (Agent SDK)

All built-in tools require permission unless pre-approved in `allowed_tools`.

| Tool | Purpose |
|------|---------|
| Read, Glob, Grep | File read/search |
| Edit, Write | File modification |
| Bash | Shell commands |
| WebSearch, WebFetch | Web access |
| Agent / Task | Spawn subagents |

```python
# Restrict to specific tools only
options = ClaudeAgentOptions(tools=["Read", "Glob", "Grep"])

# No built-in tools — custom MCP server only
options = ClaudeAgentOptions(tools=[], mcp_servers=[my_server])

# Match Claude Code default set
options = ClaudeAgentOptions(tools=[{"type": "preset", "preset": "claude_code"}])
```

### Custom tool

```python
from claude_agent_sdk import tool

@tool(
    name="get_order",
    description="Retrieves order details by order ID. Use when the user asks about a specific order.",
    annotations={"readOnlyHint": True},  # enables parallel execution with other read-only tools
)
def get_order(order_id: str) -> dict:
    try:
        return {"type": "text", "text": str(db.fetch_order(order_id))}
    except Exception as e:
        return {"is_error": True, "content": [{"type": "text", "text": str(e)}]}
```

**EXAM TRAP:** `readOnlyHint=True` tells the SDK the tool can run in parallel with other read-only tools — a performance optimization, not a security boundary.

---

## Hooks — Deterministic Control

### Why hooks instead of prompts

Prompts are probabilistic — they can never reduce error rate to zero. Hooks fire on every match, always.

### Hook lifecycle

| Hook | Runs | Can block? |
|------|------|-----------|
| `pre_tool_use` | Before tool executes | YES — return `denied` |
| `post_tool_use` | After tool executes | No blocking; can modify output |
| `can_use_tool` | Gate 3 permission check (default mode) | YES — auto-approved tools skip this |

**Key fact:** Hooks run BEFORE permission modes — even `bypassPermissions` cannot skip `pre_tool_use` or `post_tool_use`.

### Hook anatomy

```python
async def my_hook(tool_name: str, tool_input: dict, hook_name: str):
    return {
        "hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "denied",          # or "approved" or "ask"
            "permissionDecisionReason": "Reason for model",
        }
    }
    # return {} → no opinion; tool proceeds
```

### Registering hooks

```python
options = ClaudeAgentOptions(
    hooks={
        "pre_tool_use":  [{"matcher": {"name": "mcp.support.*"}, "hooks": [my_pre_hook]}],
        "post_tool_use": [{"hooks": [my_post_hook]}],  # no matcher = runs for ALL tools
    }
)
```

**EXAM TRAP:** No matcher = hook runs for every tool. Add a matcher to scope it.

---

## Conversations & Sessions

### query() vs. ClaudeSDKClient

| | `query()` | `ClaudeSDKClient` |
|--|-----------|-----------------|
| Memory across turns | ❌ | ✅ |
| Session resumption | ❌ | ✅ |
| Interrupts | ❌ | ✅ |
| Best for | One-off tasks | Chat / long-running agents |

```python
async with ClaudeSDKClient(options=options) as client:
    await client.query("Summarize the auth module")
    response = await client.receive_response()

    await client.query("Which functions lack tests?")  # SDK remembers prior turn
    response = await client.receive_response()
```

Use `resume=<session_id>` in `ClaudeAgentOptions` to continue a prior session.

### Auto-compaction

When context fills up, the Agent SDK auto-triggers a pre-compact hook: summarizes the full context, replaces old content with the summary, frees space. No extra setup required.

---

## Loading From Disk — Configuration Scopes

| Scope | Location | Shared? |
|-------|----------|---------|
| **User** | `~/.claude/` | No (personal) |
| **Project** | Project root | Yes (git) |
| **Local** | Project root (gitignored) | No (per-developer) |

```python
options = ClaudeAgentOptions(
    setting_sources=["project"],     # auto-load project folder config
    agents={"reviewer": reviewer},
    plugins=["my-plugin"],
)
```

**Workspace isolation:** Set `setting_sources=[]` for isolated workspaces (one per tenant). Share `setting_sources` for shared workspaces.

---

## Permission Mode & Allowed Tools

### Permission gate flow (order matters)

```
Gate 1: disallowed_tools?  YES → BLOCKED always
Gate 2: allowed_tools?     YES → APPROVED (no prompt)
Gate 3: permission_mode?   → see table below
```

### Permission modes

| Mode | Gate 3 behavior |
|------|----------------|
| `default` | Triggers `can_use_tool` hook — user can approve/deny |
| `acceptEdits` | Auto-approves reads + edits; other tools need approval |
| `plan` | Claude can plan but cannot execute other tools |
| `dontAsk` | Anything NOT in `allowed_tools` is auto-denied |
| `bypassPermissions` | ALL tools approved (dangerous — isolated containers only) |
| `auto` | Classifier model decides |

**EXAM TRAP:** `dontAsk` + `allowed_tools` = only listed tools run; everything else silently denied.  
**EXAM TRAP:** In `default` mode, unlisted tools still reach `can_use_tool` and can be approved at runtime.  
**EXAM TRAP:** `bypassPermissions` still respects `pre_tool_use` hooks that return `denied`.

---

## Scoped Tools & Cross-Role Exception

Each agent should only see tools relevant to its role. Extra tools cause:
- Agent drifts from intended behavior.
- Performance drops (specialist beats generalist).
- Separation of concerns blurs.

### Enforcing hard scope

```python
synthesis_agent = AgentDefinition(
    tools=["Write", "Edit"],
    allowed_tools=["Write", "Edit"],
    permission_mode="dontAsk",   # hard boundary — WebSearch cannot be called
)
```

### Constrained tool wrapper

```python
@tool(name="fetch_doc", description="Fetch internal documentation by URL. Do not use for external sites.")
def fetch_doc(url: str) -> str:
    if not url.startswith("https://internal.docs."):
        raise ValueError("Only internal documentation URLs are allowed.")
    return requests.get(url).text
```

### Cross-role exception (when justified)

Expand scope ONLY when:
1. Extra capability is narrow and purpose-specific.
2. Alternative creates measurable latency overhead.
3. Expanded tool cannot be misused for broader tasks.

---

## Orchestration — Hub-and-Spoke Pattern (EXAM CRITICAL)

```
         Coordinator
        /     |      \
  Search  Analysis  Synthesis
```

### Coordinator responsibilities

1. Decompose incoming task into subtasks.
2. Decide which subagent handles each.
3. Write a complete brief for each subagent.
4. Aggregate and validate results.
5. Handle errors and trigger retries.
6. Return final answer to the user.

**Subagents do NOT spawn subagents** (hub-and-spoke design choice).  
Subagents always start with a **fresh context window** — coordinator context is never shared.

### Benefits

| Benefit | Why |
|---------|-----|
| Observability | Coordinator sees all outputs in one place |
| Central error policy | Single place for retry/escalation logic |
| Controlled info flow | Clear, auditable paths |

---

## Defining Subagents

```python
research_agent = AgentDefinition(
    description="Searches the web for factual information. Call when external info is needed.",  # coordinator reads this
    prompt="You are a research specialist. Return structured findings with source, URL, and date.",  # subagent reads this
    tools=["WebSearch", "WebFetch"],
    model="claude-haiku-4-5",     # cheaper model for sub-tasks
)

options = ClaudeAgentOptions(
    agents={"research": research_agent, "synthesis": synthesis_agent},
    allowed_tools=["Task"],   # REQUIRED — Task is what spawns subagents
)
```

**EXAM TRAP:** `allowed_tools` must include `"Task"` (or `"Agent"`) — without it, subagents cannot be spawned.  
**EXAM TRAP:** `description` = coordinator routing decision. `prompt` = subagent behavior.  
**EXAM TRAP:** `AgentDefinition` has no `agents` field — subagents cannot spawn subagents natively.  
**EXAM TRAP:** Coordinator sees only the subagent's final message — never intermediate tool calls.

---

## Passing Findings Explicitly

Subagents have **isolated context windows** — they cannot see coordinator history or peer findings. Everything must be explicitly passed.

### Into a subagent prompt

| Item | Notes |
|------|-------|
| Task/question | Clear, scoped |
| Prior subagent findings | Structured format with sources |
| Key constraints from coordinator | Must-respect facts |
| Changed files, errors, relevant context | Anything the subagent can't discover itself |

**Rules go in the subagent's system prompt — NOT in the coordinator prompt.**

### Return from subagent (compact structured record)

```json
{
  "claim": "AI adoption in creative tools grew 47% in 2024",
  "source_name": "McKinsey AI Report 2024",
  "source_url": "https://mckinsey.com/ai-report-2024",
  "page": 12,
  "publication_date": "2024-03-15"
}
```

Raw scraped text must be summarized inside the subagent before returning — never flood the coordinator's context.

---

## Parallel Subagents

**Parallel:** total time = slowest subagent.  
**Sequential:** total time = sum of all subagents.

Parallelization is triggered **in the coordinator's prompt** — multiple `Task` calls in one assistant message = parallel execution.

```
Research how AI is used in creative industries.

Launch ALL tasks in THIS SINGLE TURN:
- Subagent 1: Music and audio — sources stay in music domain only
- Subagent 2: Writing and publishing — sources stay in writing domain only
- Subagent 3: Film, TV, animation — sources stay in film domain only
```

### Scope partitioning (avoid duplicates)

❌ Overlapping: "AI in music" + "AI in music production" → duplicate sources.  
✅ Distinct: "music and audio" + "writing and publishing" + "film/TV" + "visual art".

Clean partitioning → no duplicate sources → smaller coordinator context → lower hallucination risk.

---

## Subagent Routing & Goal-Based Coverage

### The incomplete decomposition problem

Coordinator may miss entire topic categories. Two fixes:
1. Show decomposition to user first, get feedback before executing.
2. Use a goal-based prompt with explicit quality criteria.

### Goal-based prompt pattern

```
Goal: Survey how AI is changing the creative industries across every major field.

Quality criteria:
- No major field (photography, music, writing, film) is left uncovered.
- Every claim carries a source and a date.
- Conflicting findings are both reported.
- Stop when ALL criteria hold — not after N fixed rounds.
- Maximum 5 research rounds.
```

Coordinator evaluates result against criteria → triggers targeted follow-up if not met → iterates.

---

## Recovering Locally & Reporting Failures

**Empty result ≠ success.** Always use structured failure reports.

### Recovery strategies

| Error type | Strategy |
|-----------|----------|
| Transient (timeout) | Retry with exponential backoff |
| Validation error | Fix input schema, retry |
| Business/Permission | Proceed with partial; report gap |

### Structured failure report

```json
{
  "failure_type": "timeout",
  "attempted_query": "music generation AI adoption 2024",
  "partial_results": {"music_generation": "partial report available"},
  "alternative_approaches": ["Retry with shorter time window"],
  "coverage_impact": "Report will lack quantitative claims for this segment"
}
```

**EXAM TRAP:** Empty result on failure → coordinator treats it as success → data silently missing.

---

## Enforcement — Hooks as Gates

### Value gate — block refunds over limit

```python
async def refund_gate(tool_name, tool_input, context):
    if tool_name != "process_refund":
        return {}
    amount = tool_input.get("amount", 0)
    if amount <= 500.0:
        return {}
    return {
        "hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "denied",
            "permissionDecisionReason": f"Refund ${amount} exceeds $500 limit. Use escalate_to_human.",
        }
    }
```

### Sequence gate — require identity check first

```python
async def require_customer_id(tool_name, tool_input, context):
    if tool_name not in {"lookup_order", "process_refund"}:
        return {}
    if "customer_id" not in customer_id_state:
        return {"hookSpecificOutput": {"hookEventName": "PreToolUse",
                "permissionDecision": "denied",
                "permissionDecisionReason": "Call get_customer first."}}
    return {}
```

### Manipulating tool input in a hook

```python
return {
    "hookSpecificOutput": {
        "hookEventName": "PreToolUse",
        "permissionDecision": "approved",
        "updatedInput": {**tool_input, "currency": "USD"},  # enforce currency
    }
}
```

---

## Normalizing Tool Results (`post_tool_use`)

Multiple tools return same data in different formats (Unix timestamps, numeric status codes). Fix in the hook — not in the model.

```python
STATUS_MAP = {200: "OK", 404: "Not found", 500: "Error"}

async def normalize_tool_output(input_data, tool_use_id, context):
    r = input_data["tool_response"]
    ts = r.get("created")
    iso_ts = datetime.utcfromtimestamp(ts).isoformat() + "Z" if isinstance(ts, (int, float)) else ts
    readable_status = STATUS_MAP.get(r.get("status"), str(r.get("status")))
    return {
        "hookSpecificOutput": {
            "hookEventName": "PostToolUse",
            "updatedToolOutput": {**r, "created": iso_ts, "status": readable_status},
        }
    }
    # return {} → no change
```

**EXAM TRAP:** `post_tool_use` cannot undo an already-executed tool — use `pre_tool_use` to block before execution.

---

## Structured Handoff

Use a typed tool schema for escalation — not free-text.

```python
class EscalationRecord(BaseModel):
    customer_id: str
    root_cause: str
    actions_taken: list[str]
    amount_in_scope: float
    recommended_action: str
    escalation_reason: str  # "limit_exceeded" | "policy_exception" | "customer_request" | "repeated_failure"
```

### When to escalate vs. continue

| Escalate | Keep working |
|----------|-------------|
| Customer requests human | Case is procedurally solvable |
| Policy exception needed | Customer is upset but request is covered |
| Refund exceeds limit | Tools available and process is clear |
| Repeated failures (2+ attempts) | Complex but fully within agent's scope |

**EXAM TRAP:** Don't escalate too quickly — escalate meaningfully, not defensively.  
**EXAM TRAP:** Sentiment ≠ escalation trigger. An angry customer with a covered request stays with the agent.

---

## Verify & Investigate in Parallel (Multi-Issue Tickets)

Decompose multiple concerns into structured issues:

```json
[
  {"issue_id": "ISS-001", "type": "duplicate_charge", "invoice_number": "INV-4892", "amount": 89.99, "status": "open"},
  {"issue_id": "ISS-002", "type": "missing_delivery", "order_id": "ORD-7721", "status": "open"}
]
```

Store case facts in a **separate file** (`case_facts.json`) outside the context window. Work each issue in parallel (one subagent per issue) → aggregate into one resolution response.

Track per issue: `resolved` | `partial` | `escalated`. Partial escalation is valid: some resolved automatically, some handed off — one response to the customer.

---

## Decomposition — Fixed Chain vs. Adaptive Plan

### Fixed chain (prompt chaining)

Use when all steps and their sequence are known before execution.

```python
metadata = scan_documents(docs)           # step 1
dataset  = extract_data(metadata, docs)   # step 2
report   = validate(dataset)              # step 3
```

**PR review pattern (parallel scan → integration):**
```python
results = await asyncio.gather(*[run_pass(file=f) for f in pr_files])
final_report = await integrate(local=results)
```

Each file = one agent, fresh context. Avoids attention diffusion.

**EXAM TRAP:** Attention diffusion — passing 14 files into one context → some files get deep analysis, others get shallow. Fix: one agent per file, not a bigger model.

### Adaptive plan (dynamic decomposition)

Use when discoveries during execution change the plan.

```
Agent creates plan → executes sub-steps → reviews results → revises plan → continues
```

Example: refactoring an unknown legacy codebase. Next steps cannot be known until scanning reveals which modules have tests.

### Decision matrix

| Question | Answer | Choose |
|----------|--------|--------|
| All steps known before running? | Yes | Fixed chain |
| Discoveries change the plan? | Yes | Adaptive plan |
| Steps are independent? | Yes | Parallelize within chain |
| One step feeds the next? | Yes | Sequential chain |

---

## Session Resume, Fork, or Restart

### Decision tree

```
Prior context mostly still valid?
├── NO  → Start fresh (with a targeted summary)
└── YES → Compare two approaches from here?
           ├── YES → Fork
           └── NO  → Resume
```

### Resume

```python
options = ClaudeAgentOptions(resume="<session-id>", ...)
await query(prompt="Two files changed: auth.py, config.py. Re-examine those.", options=options)
```

Use when ~80-90% of prior context still valid. Name the changed files explicitly.

### Fork

```python
options = ClaudeAgentOptions(resume="<baseline-session-id>", fork_session=True, ...)
```

Creates new branch; original baseline preserved. Multiple forks from the same baseline.

**EXAM TRAP:** Stale baseline → all forks inherit stale context. Verify baseline validity before forking.

### Start fresh

```python
prompt = """
Previously: test coverage implementation.
Status: 15 gaps found, 3 fixed.
Task: Fix the remaining 12 gaps.
"""
await query(prompt=prompt, options=ClaudeAgentOptions())  # no resume field
```

### What a session stores

- Full chat history.
- Tool call results.
- File contents read during the session.

**Risk:** Files may have changed. Always tell the agent which files changed so it re-reads them instead of using cached stale results.
