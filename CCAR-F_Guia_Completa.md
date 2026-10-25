# CCAR-F — Guía Completa de Estudio

**Claude Certified Architect Foundations**  
Examen: 60 preguntas · 120 minutos · Pass = 720 / 1000 · 4 de 6 escenarios

---

# SECCIÓN 1 — EXAM INTRO & CLAUDE FUNDAMENTALS

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

---

# SECCIÓN 2 — TOOLS AND MCP

**Domain:** Tool Design and MCP Integration  
**Files:** 12–28

---

## The Tool-Use Loop

### Four components

| Component | Who builds it |
|-----------|--------------|
| Tool function (the logic) | You |
| Tool schema (name, description, input_schema) | You |
| Dispatcher (routes tool calls to real functions) | You (or Agent SDK / Claude Code) |
| Loop (orchestrates turns) | You (or Agent SDK / Claude Code) |

### Flow per turn

```
1. Send messages + tools to Claude
2. stop_reason == "tool_use" → extract id, name, input
3. Dispatch to real function
4. Send back tool_result with same tool_use_id (role: "user")
5. Repeat until stop_reason == "end_turn" or "max_tokens"
```

**EXAM TRAP:** Tool results are sent with role `user` — there is no `tool` role.  
**EXAM TRAP:** Always include `tool_use_id` in the result — Claude matches by ID.  
**EXAM TRAP:** Resend full tool schemas on every call (stateless API).  
**EXAM TRAP:** Always store the full assistant message (including tool_use blocks) in history.

---

## Tool Definition — 3 Parts

```python
{
    "name": "search_knowledge_base",
    "description": "...",   # WHEN and how to call
    "input_schema": {       # HOW: parameters Claude fills
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "The search term"}
        },
        "required": ["query"]
    }
}
```

---

## Tool Description — 6-Point Checklist

1. **What** — what does the tool do?
2. **When** — when should it be triggered?
3. **When NOT** — inverse condition (boundary sentence)
4. **Inputs** — best practices for parameters
5. **Outputs** — format of the return value
6. **Edge cases** — known tricky cases

**EXAM TRAP:** Vague descriptions ("Search function") → wrong tool selection.  
**EXAM TRAP:** Never force tool_choice to fix wrong selection — fix the description.

---

## Diagnosing Wrong Tool Selection

```
1. Check system prompt — any keyword over-triggering a tool?
2. Check for overlapping tool names/descriptions → rename + rewrite
3. Expand descriptions (add all 6 checklist points)
4. Split into dedicated agents with scoped tools
5. Few-shot examples (last resort — high cost, high latency)
```

---

## Split vs. Consolidate vs. Scope

| Decision | When |
|----------|------|
| **Split** | Tool does too many jobs (one tool = one job) |
| **Consolidate** | Multiple similar lookup tools with same input pattern |
| **Scope** | Remove unused tools; 4-5 tools max per agent |

**EXAM TRAP:** `analyze_document` doing 3 different jobs → split into 3 tools.  
**EXAM TRAP:** `get_customer(id)` + `get_order(id)` → consolidate into `lookup_entity(type, id)`.

---

## Built-In Tools (Claude Code + Agent SDK)

| Tool | Does |
|------|------|
| **Glob** | Finds files by path/name pattern |
| **Grep** | Searches content inside files |
| **Read** | Reads full file contents |
| **Edit** | Replaces a unique anchor in a file |
| **Write** | Replaces entire file (when Edit anchor not unique) |
| **Bash** | Runs shell commands |

**Pattern:** Glob first (find files) → Grep (find content) → Read only if needed.  
**EXAM TRAP:** Grep = content inside files. Glob = file names/paths.  
**EXAM TRAP:** Empty result ≠ failure. Never return an error for zero results.

---

## MCP — Model Context Protocol

### Problem it solves

Without MCP: N apps × M services = N×M integrations.  
With MCP: Each service publishes one MCP server → any app connects via standard URL.

### Architecture

```
Host (Claude Desktop / your app)
  └── MCP Client
        └── MCP Server (exposes tools, resources, prompts)
              └── Real service (GitHub, Jira, etc.)
```

### Three primitives

| Primitive | Controlled by | Use case |
|-----------|--------------|---------|
| **Tools** | The AI model | Model decides when to call |
| **Resources** | The application | App pre-selects documents before the request |
| **Prompts** | The user | Slash commands that trigger packaged workflows |

**Resources rule:** use resources when the app can pre-select content. Use tools when the model must dynamically fetch.

### Transport modes

| Mode | Use when |
|------|---------|
| `stdio` | Local MCP server |
| `HTTP / Streamable HTTP` | Remote MCP server |

---

## MCP Error Types (EXAM CRITICAL)

| Type | Cause | Retriable | Model action |
|------|-------|-----------|-------------|
| **Transient** | Timeout, rate limit | YES (backoff) | Retry with increasing wait |
| **Validation** | Bad input format | YES (fix input) | Correct arguments, retry |
| **Business** | Policy limit hit | NO | Explain to user, offer alternative |
| **Permission** | Agent lacks access | NO | Escalate to human |

**EXAM TRAP:** Empty result ≠ error. `isError` must be `false` for zero results.  
**EXAM TRAP:** Treating failure as empty → model misses real errors (loops or gives wrong info).

```json
// Correct error format
{
  "error_category": "business",
  "retriable": false,
  "message": "Refund window expired: 45 days ago, policy is 30 days",
  "alternative": "Offer store credit or escalate"
}
```

---

## MCP Configuration

### Project-scoped (`.mcp.json`) — share with team

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {"GITHUB_TOKEN": "$GITHUB_TOKEN"}
    }
  }
}
```

**Always use env vars for credentials. Never hardcode.**

### Permission patterns

```
mcp.github.create_issue   → one tool
mcp.github.*              → all tools on server
mcp.github.get_*          → all tools starting with get_
```

**EXAM TRAP:** All MCP tools load at session start — trim unused ones to reduce confusion and token waste.

---

# SECCIÓN 3 — AGENTIC ARCHITECTURE

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

---

# SECCIÓN 4 — CLAUDE CODE IN A TEAM

**Domain:** Claude Code Configuration and Workflows  
**Files:** 57–75

---

## CLAUDE.md — Configuration That Gets Followed

### Hierarchy (innermost wins)

```
Global (~/.claude/CLAUDE.md)
  └── Project root (./CLAUDE.md)
        └── Subdirectory (./src/CLAUDE.md)  ← extends parent, doesn't overwrite
```

### Writing effective rules

| Write | Avoid |
|-------|-------|
| "Never use `var` — use `const` or `let`" | "Write modern JS" |
| "All API endpoints must be authenticated" | "Follow security best practices" |
| "Return `null`, never throw, for missing records" | "Handle errors properly" |

Rules fail when they use vague adjectives. Make them specific and testable.

### Splitting CLAUDE.md

Put in CLAUDE.md: standing conventions that apply all the time.  
Move to skill: task-specific procedures (PR review, deployments, migrations).  
Move to hook: zero-exception business rules.

**EXAM TRAP:** A 400-line CLAUDE.md causes context overflow. Move task-specific content to skills.

---

## Path-Specific Rules (frontmatter format)

```markdown
---
globs: ["src/api/**/*.ts", "tests/**/*.test.ts"]
---
# API rules — only loaded for these paths
All endpoints must validate authentication.
```

**EXAM TRAP:** Path rules are loaded based on the FILES being worked on, not the directory you're in.

---

## Skills vs. CLAUDE.md vs. Hooks

| | CLAUDE.md | Skill | Hook |
|--|-----------|-------|------|
| When loaded | Always | Description always; body on demand | Always on pattern match |
| Enforcement | Probabilistic | Probabilistic | **Deterministic** |
| Best for | Standing conventions | Task-specific procedures | Zero-exception rules |

### Decision tree

```
Should this rule NEVER be skipped?
├── YES → Hook (pre_tool_use shell script)
└── NO  → Should it always apply?
           ├── YES → CLAUDE.md
           └── NO  → Skill (task-based, on demand)
```

**EXAM TRAP:** Need to block `psql` against production? CLAUDE.md = probabilistic (will eventually be ignored). Use a hook.

---

## Working Modes

### Plan mode vs. direct execution

Use **plan mode** when:
- Complex architecture decision
- Multiple alternative approaches exist
- Large file scope (40+ files)
- Multi-step exploration of unknown codebase

Use **direct execution** for: small, well-defined bug fixes.

**Why plan mode matters:** changing a plan is cheap (minutes). Undoing code is expensive (hours, hundreds of files).

### Permission modes

| Mode | Use when |
|------|---------|
| `plan` | Complex tasks needing exploration first |
| `default` | Interactive development |
| `acceptEdits` | Let Claude edit; review result |
| `auto` | Reduce approval fatigue interactively |
| `dontAsk` | **CI/CD pipelines — mandatory** |
| `bypassPermissions` | Isolated containers only |

**EXAM TRAP:** `dontAsk` is required for CI — any approval prompt blocks a pipeline indefinitely.  
**EXAM TRAP:** `bypassPermissions` still respects `pre_tool_use` hooks that return `denied` and `disallowed_tools`.

---

## Iterative Refinement

| Technique | When |
|-----------|------|
| Few-shot input/output examples | After 2+ text iterations still failing |
| TDD (write tests first) | Any implementation task |
| Interview mode | Unknown domain or multiple solution paths |
| Batch feedback | Issues interact with each other |
| Sequential feedback | Issues are independent |

---

## Headless Mode (CLI flags)

```bash
claude -p "<prompt>" \
  --output-format json \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Bash(git diff)" \
  --max-turns 10
```

| Flag | Purpose |
|------|---------|
| `-p` | Non-interactive single prompt |
| `--output-format json` | Structured output |
| `--permission-mode dontAsk` | No approval prompts (mandatory for CI) |
| `--allowedTools` | Tool whitelist |
| `--max-turns` | Hard cap on agentic loop |
| `--bare` | Raw text only (no JSON overhead) |

---

## GitHub Action

```yaml
uses: anthropics/claude-code-action@v1
with:
  permission-mode: dontAsk
  allowed-tools: "Read,Grep,Bash(git diff)"
  post-inline-comments: true
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

permissions:
  contents: read
  pull-requests: write   # required for inline comments
```

---

## Repeating Reviews

- Always fresh session per review (no anchoring bias).
- CLAUDE.md as the review standard (evolves with the team).
- Pass previous findings as context to avoid re-reporting resolved issues.
- Use explicit criteria — not "review for security" but "check these 4 specific patterns."

---

## Testing Standards

- Point Claude at the existing test suite for style reference.
- CLAUDE.md test runner + fixture locations + standards.
- Check for redundancy before generating.
- `--allowedTools "Read,Grep,Write"` — allows Claude to write the test file directly.

---

# SECCIÓN 5 — PROMPT ENGINEERING AND STRUCTURED OUTPUT

**Domain:** Prompt Engineering and Structured Output  
**Files:** 76–95

---

## Precision — Explicit Criteria

**Problem:** vague adjectives ("be conservative", "check accuracy") → non-reproducible results.  
**Fix:** categorical criteria.

```
Flag a comment only if it:
- Contradicts the code
- Names something that does not exist
- Points at a bug already fixed

Do NOT flag:
- Stylistic differences
- Minor wording inaccuracies
- Missing comments (absence)
```

Severity calibration: add one few-shot example per severity level (Critical/High/Low) as a bar to compare against.

**EXAM TRAP:** "Be conservative" without examples = uncalibrated = anti-pattern.  
**EXAM TRAP:** Self-assigned confidence scores without labeled data = anti-pattern.

---

## Precision vs. Coverage

**Do two things well instead of five things poorly.**

If a category has >20% false-positive rate:
1. **Disable** it from the pipeline
2. **Fix** offline with few-shot prompts
3. **Re-enable** once performance improves

**EXAM TRAP:** Reducing strictness (fewer results) also removes real positives from high-precision categories.  
**EXAM TRAP:** Adding a confidence score to noisy results doesn't rebuild trust.

---

## Few-Shot Examples — When and How

Use ONLY for ambiguous boundary cases that survive criteria improvements.

Decision flow:
```
Tool descriptions well-defined?  NO → fix descriptions first
Categorical criteria in place?   NO → add criteria first
Still false positives at boundary? YES → add few-shot examples for that boundary only
```

One or two examples per boundary is enough — Claude generalizes.

**Evidence the prompt works:** false positives drop on cases NOT in the few-shot set (generalization).  
**Not evidence:** only performs better on the exact examples provided.

---

## Document Extraction — Missing Fields

Two failure modes:
| Mode | Cause |
|------|-------|
| Claude **invents** a value | Field required but absent; retry pressure increases hallucination |
| Claude returns **empty** but value was present | Layout not understood |

**EXAM TRAP:** Recommending a retry when the value is absent from the document → increases hallucination, not accuracy.

When field is missing — fix options:
- Use `null` type: `"type": ["string", "null"]`
- Add enum escape values: `"unclear"`, `"other"`
- Remove `required` if truly optional

---

## Structured Output — The Forced Tool Call Pattern (EXAM PREFERRED)

```python
tools = [{"name": "extract_invoice", "input_schema": schema}]
response = client.messages.create(
    ...,
    tools=tools,
    tool_choice={"type": "tool", "name": "extract_invoice"}
)
extracted = response.content[0].input  # structured output — no prose
```

**Key schema parameters:**
- `additionalProperties: false` — prevents extra fields
- `strict: true` — stops flow if required field missing
- `tool_choice` forced — guarantees exactly this tool is called

**EXAM TRAP:** Plain prompt asking for JSON → prose response (unreliable). Always use forced tool call.  
**EXAM TRAP:** Schema validates structure — NOT semantic correctness. Add deterministic code for arithmetic checks.

---

## Schema Validation — Retry vs. Human Review

Retry helps:
- Date in wrong format
- Value in wrong field
- Arithmetic error (single digit)

Retry does NOT help:
- Value not in the document at all → send to **human review**

---

## Grounded Confidence Score

Anti-patterns:
- Pure self-rating (no calibration data)
- Single score for whole document
- Threshold set by intuition

Correct pattern:
1. Score **per field** (not per document)
2. Calibration dataset (labeled few-shot examples)
3. Threshold derived from data
4. Route: above threshold → automate; below → human review

**Routing matrix:**

| Source contradicts itself | Confidence | Action |
|--------------------------|------------|--------|
| No | Above threshold | Automate |
| Yes | High | Human review |
| Either | Below threshold | Human review |

---

## Message Batch API

| Dimension | Batch API | Synchronous |
|-----------|-----------|-------------|
| Cost | Up to 50% cheaper | Full price |
| Latency | Up to 24h | Seconds |
| Tool calls | Single forced tool only | Full multi-turn |
| Best for | Large volume, flexible SLA | Tight deadlines, interactive |

**EXAM TRAP:** Batch cannot do multi-turn. It can do ONE forced tool call only.  
**EXAM TRAP:** 24h is the max SLA — plan your time budget accordingly (preparation + batch + downstream).

---

## Multi-Pass Review

| Pass | What it finds |
|------|--------------|
| Pass 1 (per-file, parallel) | Bugs in code, local issues, integration points |
| Pass 2 (integration, serial) | Cross-file data flow errors, import/export mismatches |

Order is mandatory: Pass 1 → Pass 2.

---

## Multi-Instance Review

Fresh session for every review. Do NOT pass:
- Generating session's chat history
- Any hint "the code is mostly fine"
- Generator's reasoning trace

Reconcile findings: **union** (not majority vote). Keep all unique findings. For duplicates: keep higher severity.

**EXAM TRAP:** Majority voting cuts real positives — unacceptable in security contexts.

---

## Test Small → Batch Corpus

1. Build representative sample (one example per document type, longest docs, all layouts)
2. Optimize prompt on sample using **synchronous API** (fast feedback)
3. Submit full corpus to **Batch API** only after one clean pass

Resubmission: investigate only the failed items (never rerun full job). Track by custom ID. Upsert — never duplicate.

---

## Learning From Dismissed Findings

Add `detected_pattern` field to schema → Claude categorizes dismissals → group by pattern → build "do not flag" blocks + few-shot examples → watch for generalization.

---

# SECCIÓN 6 — CONTEXT AND RELIABILITY

**Domain:** Context Management and Reliability  
**Files:** 96–107

---

## Context Window — Ground Rules

| Fact | Detail |
|------|--------|
| Limited | ~1M tokens (most models); 200K (Haiku) |
| Stateless (Messages API) | Nothing persists across calls |
| Loss in the middle | Focus highest at start/end; middle content can be missed |
| More context = higher cost | All prior tokens count as input each request |

**EXAM TRAP:** Prompt caching reduces cost and latency but does NOT eliminate the context limit.

---

## Context Levers (5 tools)

| Lever | How |
|-------|-----|
| **Case-facts section** | Dedicated block below system prompt; updated by deterministic code; survives compaction |
| **Post-tool-use trim** | Hook strips irrelevant fields before they enter context (keep 5 of 40 fields) |
| **Priority ordering** | Key findings at top, detailed results in middle, action items at bottom |
| **Subagent isolation** | Send only the selected path and source location — not full docs or reasoning chains |
| **Scratchpad files** | Agent writes findings to disk; new sessions or subagents re-read them |

---

## Directed Compaction

```bash
/compact Keep all order IDs, amounts, dates, and open action items.
         Drop all tool output from order lookups.
```

**What compaction leaves untouched:** memory files on disk, case-facts block.  
**What gets summarized:** turn history, tool results.

Keep in compaction instruction: decisions + reasoning, identifiers, document references, constraints, open questions.  
Drop: stale tool output, dead-end paths, acknowledgment back-and-forth.

---

## Context Degradation — How to Avoid

```
Haven't started discovery?
  └── Delegate to subagent (fresh context from the start)
Already did discovery, context at 70–80%?
  └── /compact with directed instruction, then continue
Already degraded?
  └── New session with best available summary
```

**Codebase discovery pattern:**
1. Glob — list files by pattern
2. Grep — find definitions and exports
3. Grep — find where those exports are imported

**EXAM TRAP:** Never start implementation with 70-80% context already used.

---

## Recovering State

| File | Purpose |
|------|---------|
| `manifest.json` | Track progress per subagent (status: completed/in_progress/not_started, queries run, gaps) |
| `scratchpad.md` | Store findings and learnings (prose) |

Resume if world unchanged. Restart if files changed significantly around the agent.

---

## Escalation Triggers

**Policy-based, NOT sentiment-based.**

| Condition | Decision |
|-----------|---------|
| Customer asks for a human | Escalate |
| Request not covered by policy | Escalate |
| Two resolution attempts failed | Escalate |
| Customer angry BUT request is covered | Resolve (don't escalate) |
| Customer calm BUT request not covered | Escalate |

**EXAM TRAP:** Sentiment analysis as escalation signal = anti-pattern.  
**EXAM TRAP:** Self-reported confidence without labeled data = anti-pattern.

---

## Ask vs. Escalate

```
Agent can't proceed:
├── Answer is in a database? → Tool call
├── Answer is ambiguous (e.g., 3 customer records)?
│     └── Customer can know it? → Ask customer
│         Internal ID only? → Don't ask; use tools or escalate
└── Policy gap (no business rule exists)? → Escalate to human
```

**EXAM TRAP:** Never use heuristics (most orders, closest name match) for identity resolution — always confirm.

---

## Human Review & Calibration

**Stratified sampling:** take ~3% from EACH document type — not just from the aggregate.

Why: a 97% aggregate can hide a 60% accuracy rate on a small document type.

**What confidence scores cover:** field presence, schema structure.  
**What they do NOT cover:** arithmetic errors, wrong-field values, invented values.

---

## Provenance & Conflicting Sources

```json
{
  "claim": "AI music market ~$3.2B",
  "source": "Music Tech Report 2024",
  "collection_method": "vendor revenue survey",
  "conflict_detected": true,
  "values": [
    {"value": "12%", "source": "Platform"},
    {"value": "8%",  "source": "Survey"}
  ],
  "possible_explanation": "Different methods and time periods"
}
```

**EXAM TRAP:** Never average conflicting values. Never ignore the conflict. Keep both + flag.

---

## Research Report Structure

1. **Well Established** — overlapping sources, cite them
2. **Contested** — both values, possible explanation, no winner picked
3. **Coverage Gaps** — sources attempted, attempts count, conclusions that can't be drawn

**Format preservation:** tables stay tables, lists stay lists, prose stays prose. Converting everything to bullets loses meaning encoded in the format.

**Synthesis step:** merge overlapping claims. **Formatting step:** polish only — never change certainty level.

---

# SECCIÓN 7 — EXAM READINESS

**Domain coverage:** All 5 domains  
**Files:** 108–112

---

## How to read an exam question (4-step method)

1. **Setting** — what is the system doing?
2. **Symptom** — what is broken/wrong?
3. **Constraint** — what is the hard rule that cannot be violated?
4. **Ask** — what specifically is being asked?

> Always identify the **constraint** first — it points directly to the correct answer.

---

## Fix complexity hierarchy (cheapest → most complex)

```
1. Prompt / CLAUDE.md rule
2. Few-shot examples for boundary cases
3. Tool name / description / schema change
4. Multi-agent architecture (scope, roles)
5. Hooks + validation + retry logic
```

**Exam rule:** always try the innermost fix. If an option adds a trained classifier or a larger model when a schema fix would work — it's a distractor.

---

## Domain 1 — Agentic Architecture quick answers

| Symptom | Fix |
|---------|-----|
| Loop ends wrongly | Check `stop_reason`, not response text |
| Coverage gaps in research pipeline | Add subagent per missing topic |
| Important step skipped | `pre_tool_use` hook (deterministic) |
| Agent crashed — resume or restart? | Resume if world unchanged; restart if files changed |
| Fixed vs. adaptive plan? | Fixed when steps known; adaptive for unknown codebase |

**EXAM TRAP:** `post_tool_use` cannot block a call — only `pre_tool_use` can.  
**EXAM TRAP:** Subagents default to parallel — don't run sequentially unless order matters.

---

## Domain 2 — Tool Design quick answers

| Symptom | Fix |
|---------|-----|
| Agent picked wrong tool | Expand tool description |
| Two tools overlap | Rename + rewrite both descriptions |
| Tool failed (transient) | Retry |
| Tool failed (business error) | Return structured error with category |
| Too many tools | Scope tool access |

**EXAM TRAP:** Never force `tool_choice` to fix a wrong tool selection — fix the description.  
**EXAM TRAP:** Empty result ≠ failure. Empty should be a valid success.

---

## Domain 3 — Claude Code quick answers

| Symptom | Fix |
|---------|-----|
| Teammates have different CLAUDE.md | `/memory` to diagnose; move file to project root |
| Rule needed every session | CLAUDE.md |
| Rule needed for specific task | Skill |
| Rules under certain paths | Path-specific CLAUDE.md |
| CI pipeline | `-p` + `--permission-mode dontAsk` + `--allowedTools` + `--output-format json` |

**EXAM TRAP:** CLAUDE.md in subdirectory extends the parent — does NOT overwrite it.

---

## Domain 4 — Prompt Engineering quick answers

| Symptom | Fix |
|---------|-----|
| Vague criteria / false positives | Categorical criteria + few-shot boundary examples |
| Need structured output | Forced tool call — read `tool_block.input` |
| Extraction fails semantic check | Deterministic code validation |
| Batch vs. synchronous? | Batch: cost saving, 24h SLA; sync: tight deadline |
| Who reviews? | Always a **fresh session** — current session is biased |

**EXAM TRAP:** "Be conservative" without specific criteria = vague and unreproducible.  
**EXAM TRAP:** Extracting a field absent from the document → hallucination; use nullable.  
**EXAM TRAP:** Retry on a field absent from the document → increases hallucination, not accuracy.

---

## Domain 5 — Context & Reliability quick answers

| Symptom | Fix |
|---------|-----|
| Facts lost after compaction | Dedicated facts block in system prompt (code-controlled) |
| When to escalate? | Customer asks for human; policy gap; two failed attempts |
| Subagent failed — what to report? | Failure type + queries tried + partial results |
| Huge discovery → context flood | Delegate to subagent (fresh context) |
| Sources disagree | Keep both; flag conflict; never average |

**EXAM TRAP:** Escalating on sentiment = wrong. Escalate only on policy criteria or explicit request.  
**EXAM TRAP:** Self-rated confidence without labeled data = unreliable anti-pattern.  
**EXAM TRAP:** `/compact` without instructions → add custom instruction to preserve important facts.

---

## Sample question — correct reasoning

**Problem:** Required batch_number field → lines without a batch number came back with one (invented). Every response passes schema validation. Wrong batch number hides stock from recall.

**Correct answer:** Make batch_number nullable in schema + describe null as the answer when absent.

**Why others fail:**
- System prompt instruction: probabilistic, schema still forces a value
- Retry with supplier check: treats symptom, not root cause
- Larger model: capability is not the issue — schema design is

---

# PRACTICE TEST — SIMULACRO COMPLETO

**60 questions · 120 minutes · Pass = 720 / 1000**  
Domains: Agentic Architecture · Claude Code · Prompt Engineering · Tool Design + MCP · Context & Reliability

---

## DOMAIN 1 — AGENTIC ARCHITECTURE (Questions 1–20)

---

**Q1.**  
A support agent built with the Agent SDK is supposed to verify customer identity before processing any refund. During testing, engineers observe that in about 8% of cases the agent processes a refund without calling `get_customer` first. The current setup uses a detailed system prompt and three few-shot examples enforcing the rule. What is the BEST fix?

A) Add more few-shot examples showing the correct sequence.  
B) Increase the model temperature to make the agent more consistent.  
C) Add a `pre_tool_use` hook that denies `process_refund` if no customer ID has been recorded.  
D) Add a `post_tool_use` hook that reverses the refund if no customer ID was set.  

**Answer: C**  
> Hooks are deterministic — a `pre_tool_use` hook reduces error rate to zero. More examples (A) are probabilistic; they reduce error rate but never to zero, which is unacceptable for money. Temperature (B) does not fix sequencing. A `post_tool_use` hook (D) runs AFTER the refund already executed — it cannot undo a completed action.

---

**Q2.**  
An agent coordinator is decomposing a research task into subagent briefs. The coordinator's `allowed_tools` list contains: `["Read", "Grep", "WebSearch"]`. The coordinator attempts to spawn a subagent but it silently never runs. What is the most likely cause?

A) The subagents are not registered in `agents` on `ClaudeAgentOptions`.  
B) The coordinator's `allowed_tools` does not include `"Task"`.  
C) The subagent's `model` field was omitted.  
D) The coordinator's `permission_mode` is set to `dontAsk`.  

**Answer: B**  
> `"Task"` (or `"Agent"`) must be in `allowed_tools` for the coordinator to spawn subagents. Without it, the coordinator simply cannot call the Task tool. A missing `model` field (C) inherits from parent — not a blocker. `dontAsk` (D) would deny non-listed tools but Task needs to be listed; the root cause is still the missing listing.

---

**Q3.**  
A multi-agent research system sends subagent findings to a synthesis agent. The synthesis agent returns a report with some claims that cannot be traced back to any source. What is the most likely architectural mistake?

A) The research subagent used `WebSearch` instead of `WebFetch`.  
B) The coordinator passed the synthesis agent a plain text summary of findings instead of structured records with source metadata.  
C) The synthesis agent's model should be cheaper (Haiku instead of Sonnet).  
D) The coordinator did not set `readOnlyHint=True` on the research tools.  

**Answer: B**  
> When findings are passed as unstructured text, the synthesis agent has no anchor for citations. The fix is to pass structured records (claim, source_name, source_url, date) so the synthesis agent can preserve provenance. `readOnlyHint` (D) is a performance flag, not a provenance control.

---

**Q4.**  
You need subagents that research four independent topics simultaneously to minimize total latency. What is the correct way to trigger parallel execution?

A) Call each subagent in a separate turn, one at a time.  
B) Set `parallel=True` in `ClaudeAgentOptions`.  
C) Instruct the coordinator to "Launch ALL tasks in THIS SINGLE TURN" in its prompt.  
D) Use `asyncio.gather()` in the coordinator's Python code around four `query()` calls.  

**Answer: C**  
> Parallelization is triggered by the coordinator's prompt — multiple `Task` calls in one assistant message run in parallel automatically. There is no `parallel=True` flag (B). Separate turns (A) = sequential. `asyncio.gather()` on `query()` (D) would work at the SDK level but the exam-preferred pattern is coordinator-driven fan-out via prompt.

---

**Q5.**  
A subagent times out while searching for data on music AI market share. It returns an empty object `{}` to the coordinator. What consequence does this have?

A) The coordinator raises an exception and halts the pipeline.  
B) The coordinator correctly identifies the failure and triggers a retry.  
C) The coordinator treats the result as a successful empty search and continues without the data.  
D) The coordinator escalates to a human automatically.  

**Answer: C**  
> Empty result `{}` looks like success with no data. The coordinator has no way to distinguish a timeout from a genuine zero-result search. The subagent must return a structured failure report with `failure_type`, `coverage_impact`, and `alternative_approaches`.

---

**Q6.**  
A synthesis agent occasionally browses the web on its own instead of writing the report. Its tools are `["Write", "Edit", "WebSearch"]`. What is the BEST fix?

A) Add a system prompt rule: "Do not use WebSearch. Write only."  
B) Remove `WebSearch` from the synthesis agent's `tools` list and set `permission_mode="dontAsk"`.  
C) Add a `pre_tool_use` hook that denies `WebSearch` for the synthesis agent.  
D) Add a few-shot example where the synthesis agent correctly avoids WebSearch.  

**Answer: B**  
> The correct fix is to remove the tool from scope entirely. With `dontAsk`, anything not in `allowed_tools` is auto-denied. A system prompt rule (A) is probabilistic — it can be ignored. A hook (C) works but is more complex than simply not giving the tool. Few-shot (D) is probabilistic.

---

**Q7.**  
You want to continue a previous Agent SDK session. Most context is still valid but two files have changed. What is the correct approach?

A) Start a fresh session and summarize the full previous session in the new prompt.  
B) Use `resume="<session-id>"` and explicitly name the changed files in the new prompt.  
C) Fork the session and re-run all prior steps with the new files.  
D) Use `ClaudeSDKClient` to re-send the full chat history from the prior session.  

**Answer: B**  
> When ~80-90% of context is still valid, resume and name the changed files so the agent re-reads them instead of using cached stale data. Start fresh (A) when many things have changed. Fork (C) is for comparing two different approaches from a shared baseline — not for file updates.

---

**Q8.**  
An agent coordinator checks its research results against quality criteria and finds that one topic category is missing. There were no explicit instructions for how many rounds to run. What happens next?

A) The coordinator stops and returns a partial report.  
B) The coordinator escalates to a human because it cannot proceed.  
C) The coordinator triggers targeted follow-up research for the missing category.  
D) The coordinator loops indefinitely until all criteria are met.  

**Answer: C**  
> In a goal-based prompt with quality criteria, the coordinator self-corrects: if criteria are not met, it spawns targeted follow-up subagents. It does not loop indefinitely (D) — a maximum rounds limit (e.g., 5) bounds it. It does not stop prematurely (A) or escalate (B) when it can still self-correct.

---

**Q9.**  
A `post_tool_use` hook receives the result of a `process_refund` call and detects that the refund amount exceeds the policy limit. What can the hook do?

A) Reverse the refund by calling the refund tool again with a negative amount.  
B) Block the refund from completing.  
C) Modify the tool output that Claude will receive.  
D) Both A and B.  

**Answer: C**  
> `post_tool_use` runs AFTER the tool has executed — it cannot block or reverse. It can only modify what Claude sees in the result (`updatedToolOutput`). The gate for blocking must be a `pre_tool_use` hook.

---

**Q10.**  
A coordinator uses a fixed pipeline: scan → extract → validate. The extract step fails for one document type due to a schema mismatch. Which recovery strategy is correct?

A) Retry the extract step with exponential backoff.  
B) Modify the input schema to match the document type, then retry.  
C) Proceed with partial results and report the coverage gap.  
D) Fork the session and retry from the scan step.  

**Answer: B**  
> Schema mismatch = validation error. Fix the input (schema correction), then retry. Exponential backoff (A) is for transient/timeout errors. Proceeding with partial results (C) is the fallback if retries fail, not the first step for a validation error.

---

**Q11.**  
An agent uses `query()` for every user message. After the first turn, the user asks a follow-up question that depends on the first answer. The agent has no context about the first answer. What is the root cause?

A) `query()` does not support `allowed_tools`.  
B) `query()` is stateless — it does not preserve history across calls.  
C) The agent's `permission_mode` was set to `dontAsk`.  
D) The system prompt was not resent.  

**Answer: B**  
> `query()` is a one-shot call with no session state. For multi-turn memory, use `ClaudeSDKClient`, which stores the session on disk and can be resumed.

---

**Q12.**  
A tool's `post_tool_use` hook receives a `status` field with value `404` and a `created` field with value `1718000000` (Unix timestamp). The hook's job is to normalize these. What should it return?

A) `{"status": "404", "created": "1718000000"}`  
B) `{"status": "Not found", "created": "2024-06-10T..."}`  
C) Leave the values as-is and return `{}`.  
D) Deny the tool call.  

**Answer: B**  
> Normalization converts numeric status codes to natural language and Unix timestamps to ISO 8601. LLMs process natural language more reliably, reducing token use and hallucination risk. Returning `{}` (C) means no change — the raw values reach Claude.

---

**Q13.**  
When forking a session, the fork inherits stale context from three days ago. Several key files have been rewritten since then. What is the risk?

A) The fork will crash because session IDs expire.  
B) The fork will use cached file contents that no longer match the codebase.  
C) The fork will automatically detect the changed files and re-read them.  
D) The fork will run correctly because files are always re-read on session start.  

**Answer: B**  
> A session stores tool results including file contents read during the session. If those files changed, the fork inherits stale data. The agent will not detect changes automatically — you must tell it which files changed.

---

**Q14.**  
A customer sends one message with two issues: a billing dispute and a missing shipment. What is the BEST initial step for an agent handling this?

A) Ask the customer which issue they want to address first.  
B) Escalate immediately to a human since there are multiple issues.  
C) Decompose the message into structured issue records, then work each in parallel.  
D) Address only the first issue mentioned and wait for the customer to follow up on the second.  

**Answer: C**  
> The correct pattern is to decompose into structured issue records (issue_id, type, claim, status) and work them in parallel with one subagent per issue. Asking the customer (A) adds unnecessary friction. Escalating (B) too quickly is an anti-pattern. Ignoring the second issue (D) misses the requirement.

---

**Q15.**  
A synthesis agent needs to verify a date claim mid-task. The current design routes all fact-checking through the coordinator, adding 3-4 extra turns. The team is considering giving the synthesis agent a narrow `check_date(query)` tool. What condition justifies this scope expansion?

A) The synthesis agent is the most expensive model so it should have more tools.  
B) The extra capability is narrow and purpose-specific, the coordination overhead is measurable, and the tool cannot be misused for broader tasks.  
C) The coordinator explicitly allows it in the `allowed_tools` list.  
D) The synthesis agent is currently failing more than 10% of tasks.  

**Answer: B**  
> The three conditions for a justified cross-role exception are: narrow + purpose-specific capability, measurable overhead from the alternative, and the tool cannot be misused for broader tasks. Cost (A) is irrelevant. Coordinator's `allowed_tools` (C) controls coordinator tools, not subagent scope.

---

**Q16.**  
Which of these correctly describes the `description` field vs. the `prompt` field in `AgentDefinition`?

A) `description` defines behavior; `prompt` defines routing.  
B) `description` is read by the coordinator to decide which subagent to call; `prompt` defines the subagent's behavior and output format.  
C) Both are read by the subagent at startup.  
D) `description` is optional; `prompt` is required.  

**Answer: B**  
> `description` = coordinator routing decision (who reads: parent agent). `prompt` = subagent system prompt (who reads: subagent itself). Role and behavior instructions must go in `prompt` — putting them in `description` does not improve subagent performance.

---

**Q17.**  
An agent calls `process_refund` with `amount=750`. A `pre_tool_use` hook checks this and returns `permissionDecision: "denied"`. The agent's `allowed_tools` list includes `"process_refund"`. What happens?

A) The refund executes because `allowed_tools` pre-approves it.  
B) The hook denial is ignored because `bypassPermissions` mode overrides hooks.  
C) The tool is blocked — hooks run before permission gates and override them.  
D) The `can_use_tool` hook is triggered next.  

**Answer: C**  
> Hooks run BEFORE permission modes — they override even `allowed_tools` pre-approval and even `bypassPermissions`. A `pre_tool_use` denial always blocks, regardless of other settings.

---

**Q18.**  
You are designing a pipeline to review a PR with 15 files. Which architecture avoids the attention diffusion problem?

A) Send all 15 files in one context window to a single agent for deep analysis.  
B) Use a larger model (Opus instead of Sonnet) for better attention across all files.  
C) One agent per file (parallel), then one integration agent to aggregate findings.  
D) Run the same single agent three times sequentially to catch missed issues.  

**Answer: C**  
> Attention diffusion is caused by too much content in one context — not by model size. The fix is one agent per file (fresh context, full attention), then one integration pass for cross-file issues. A bigger model (B) does not solve attention distribution.

---

**Q19.**  
A coordinator prompt says "Research how AI is used in creative industries. Subagent 1: AI in music. Subagent 2: AI in music production." After aggregation, the synthesis agent finds many duplicate sources. What is the root cause?

A) Both subagents used the same `WebSearch` tool.  
B) The scope partitioning overlapped — "music" and "music production" are not distinct domains.  
C) The coordinator did not set `allowed_tools=["Task"]`.  
D) The synthesis agent lacks a `post_tool_use` normalization hook.  

**Answer: B**  
> Overlapping scope → duplicate sources → messy synthesis. The fix is distinct, non-overlapping domains: "music and audio" vs. "writing and publishing" vs. "film, TV, animation" vs. "visual art."

---

**Q20.**  
A support agent has the tools: `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`. The agent sometimes calls `process_refund` without ever calling `lookup_order` first, resulting in refunds applied to wrong orders. What is the BEST fix?

A) Add a system prompt rule: "Always call lookup_order before process_refund."  
B) Add a sequence gate: a `pre_tool_use` hook that denies `process_refund` until `lookup_order` has been called and an order ID has been recorded.  
C) Remove `process_refund` from the tools list.  
D) Use `tool_choice` to force the model to always call `lookup_order`.  

**Answer: B**  
> A sequence gate is the deterministic solution. A system prompt rule (A) is probabilistic. Removing the tool (C) breaks functionality. `tool_choice` (D) forces a specific tool on the next call — it doesn't enforce a required sequence across multiple turns.

---

## DOMAIN 2 — CLAUDE CODE CONFIGURATION (Questions 21–34)

---

**Q21.**  
A team's CLAUDE.md file has grown to 420 lines. During a complex refactoring task, Claude ignores several rules from the middle of the file. What is the most likely cause and fix?

A) The file has a syntax error; fix the formatting.  
B) CLAUDE.md files have a 400-line hard limit; trim to 400.  
C) A long CLAUDE.md causes context overflow; move task-specific content to skills and zero-exception rules to hooks.  
D) CLAUDE.md rules are case-sensitive; rewrite them in lowercase.  

**Answer: C**  
> A 400+ line CLAUDE.md bloats the system prompt, leading to context overflow where content in the middle gets less attention. Fix: CLAUDE.md = standing conventions; skills = task-specific procedures; hooks = zero-exception rules.

---

**Q22.**  
A CLAUDE.md rule states: "Be careful with database operations." Engineers report Claude still makes risky DB changes. What is the problem?

A) The rule needs to be placed at the top of the file.  
B) The rule uses a vague adjective — it is not specific and testable.  
C) CLAUDE.md does not support database-related rules.  
D) The rule needs a code example attached.  

**Answer: B**  
> Vague adjectives ("be careful") produce non-reproducible results. Effective rules are specific and testable: "Never run DELETE without a WHERE clause. Never run migrations against production without a dry-run flag."

---

**Q23.**  
Which permission mode is MANDATORY when running Claude Code in a CI/CD pipeline?

A) `auto`  
B) `acceptEdits`  
C) `dontAsk`  
D) `bypassPermissions`  

**Answer: C**  
> `dontAsk` is required for CI. In any other mode, Claude can trigger approval prompts that block the pipeline indefinitely. `bypassPermissions` (D) is only for isolated containers — it is dangerous for general use.

---

**Q24.**  
A project has `./CLAUDE.md` at the root and `./src/payments/CLAUDE.md` for the payments module. A rule in the root CLAUDE.md says "Return null for missing records." The payments CLAUDE.md says "Throw PaymentNotFoundException for missing records." Claude is editing `./src/payments/processor.py`. Which rule applies?

A) The root rule always wins.  
B) The more specific (innermost) CLAUDE.md wins — Claude throws PaymentNotFoundException.  
C) Both rules apply simultaneously, causing a conflict.  
D) CLAUDE.md files in subdirectories are ignored.  

**Answer: B**  
> The CLAUDE.md hierarchy gives precedence to the innermost file. The payments-specific rule overrides the root rule for files in that directory.

---

**Q25.**  
A path-specific rule in CLAUDE.md has `globs: ["src/api/**/*.ts"]`. An engineer is editing `src/api/auth/login.ts`. Is this rule loaded?

A) Yes, because the file matches the glob pattern.  
B) No, because the engineer's working directory is not `src/api`.  
C) Only if the engineer explicitly loads the rule with `/skill`.  
D) Yes, but only if the file is open in the IDE.  

**Answer: A**  
> Path rules are loaded based on the FILES being worked on, not the current working directory. The glob `src/api/**/*.ts` matches `login.ts`, so the rule loads.

---

**Q26.**  
A team needs a rule that blocks any `psql` command targeting the production database hostname. The rule must NEVER be skipped. Where should it go?

A) CLAUDE.md at the project root.  
B) A skill document.  
C) A `pre_tool_use` hook (shell script) that checks the command string before execution.  
D) A path-specific CLAUDE.md for the `migrations/` folder.  

**Answer: C**  
> CLAUDE.md rules are probabilistic — they can be overridden or forgotten over many turns. Hooks are deterministic: the shell script fires on every `Bash` tool call matching the pattern. The exam decision tree: "Must NEVER be skipped? → Hook."

---

**Q27.**  
A GitHub Action runs Claude Code to review PRs. The action needs Claude to post inline comments on the diff. Which permission is required?

A) `contents: write`  
B) `pull-requests: write`  
C) `issues: write`  
D) `actions: read`  

**Answer: B**  
> Inline PR comments require `pull-requests: write`. `contents: read` is sufficient for reading files. `actions: read` is for reading workflow metadata.

---

**Q28.**  
A developer asks Claude to refactor a module that spans 45 files with complex interdependencies and multiple valid architecture approaches. Which working mode is most appropriate?

A) Direct execution — small bug fix.  
B) `acceptEdits` mode — let Claude edit and review afterward.  
C) Plan mode — explore and draft a plan before executing.  
D) `bypassPermissions` — large scope needs all tools approved.  

**Answer: C**  
> Plan mode is appropriate for: complex architecture decisions, multiple valid approaches, large file scope (40+ files), unknown codebase. Changing a plan is cheap; undoing 45 files of code is expensive.

---

**Q29.**  
A skill document describes a PR review checklist. When is the skill's BODY loaded into Claude's context?

A) At session start, always.  
B) Only when Claude decides it is relevant to the current task.  
C) Only when the user explicitly invokes it (e.g., `/pr-review`).  
D) Never — skills are metadata only.  

**Answer: C**  
> Skill descriptions are always loaded (for routing). The body is loaded on demand — when the skill is explicitly invoked. This keeps context lean for tasks that don't need the skill.

---

**Q30.**  
A team runs repeated PR reviews with Claude. After several reviews, engineers notice Claude is anchoring on its previous assessments and not flagging things it flagged before. What is the correct fix?

A) Use a single long-running session so Claude remembers all previous reviews.  
B) Pass the previous review's findings as context so Claude avoids re-reporting resolved issues, but always use a fresh session per review.  
C) Increase `max_turns` so Claude has more time per review.  
D) Use `bypassPermissions` so Claude runs faster.  

**Answer: B**  
> Fresh session per review prevents anchoring bias. Pass previous findings to avoid re-reporting already-resolved issues. One long-running session (A) causes anchoring — the opposite of what's needed.

---

**Q31.**  
A headless Claude Code invocation looks like:
```
claude -p "Review auth.py for SQL injection" --output-format json --permission-mode dontAsk --allowedTools "Read,Grep"
```
The review never touches `auth.py` even though it exists. What is the most likely cause?

A) `--permission-mode dontAsk` blocks all tool use.  
B) `Bash` is not in `--allowedTools`, but that's unrelated.  
C) `Read` and `Grep` are in `--allowedTools` so they should work — the issue is elsewhere.  
D) `--allowedTools "Read,Grep"` is correctly scoped; Claude may have used a search and found nothing suspicious.  

**Answer: D**  
> `Read` and `Grep` in `allowedTools` with `dontAsk` means those tools run; everything else is denied. The most likely explanation is the review ran correctly and found nothing — not a misconfiguration. If `Read` were missing, Claude couldn't open the file.

---

**Q32.**  
A CLAUDE.md rule should only apply when working on test files (`tests/**/*.test.ts`) and API source files (`src/api/**/*.ts`). How is this configured?

A) Create two separate CLAUDE.md files in each directory.  
B) Use a frontmatter `globs` field listing both patterns in one rule block.  
C) Set an environment variable `CLAUDE_PATHS` at startup.  
D) Use a hook that checks file paths.  

**Answer: B**  
> Path-specific rules use frontmatter: `globs: ["src/api/**/*.ts", "tests/**/*.test.ts"]`. Multiple patterns can be listed in one rule.

---

**Q33.**  
After two iterations with text feedback, Claude is still generating tests in the wrong structure. What technique should be tried next?

A) Increase `max_tokens` for longer responses.  
B) Switch to a cheaper model.  
C) Provide few-shot input/output examples showing the correct test structure.  
D) Add the test format to CLAUDE.md.  

**Answer: C**  
> The iterative refinement decision: after 2+ text iterations still failing → add few-shot examples. CLAUDE.md (D) handles standing conventions but doesn't demonstrate the exact test shape — examples do.

---

**Q34.**  
`bypassPermissions` mode is enabled in Claude Code. A `pre_tool_use` hook denies a `Bash` command that attempts to access `/etc/passwd`. Does the command execute?

A) Yes — `bypassPermissions` overrides all hooks.  
B) No — hooks run before permission modes and override them, including `bypassPermissions`.  
C) Yes — `bypassPermissions` + `allowed_tools` always wins.  
D) It depends on whether the hook is registered in `settings.json` or `.mcp.json`.  

**Answer: B**  
> Hooks run before permission modes. Even `bypassPermissions` cannot override a `pre_tool_use` hook that returns `denied`. This is a critical safety design.

---

## DOMAIN 3 — PROMPT ENGINEERING & STRUCTURED OUTPUT (Questions 35–46)

---

**Q35.**  
A review pipeline flags 35% of stylistic comments as bugs. The team adds confidence scores: the model now returns 0.9 confidence on stylistic issues it misidentifies as bugs. What does this change accomplish?

A) It makes the pipeline accurate again.  
B) It allows the team to route low-confidence items to human review.  
C) It adds a number to unreliable results without rebuilding trust — confidence scores on noisy results don't fix the noise.  
D) It enables batch processing for high-confidence items.  

**Answer: C**  
> Adding a confidence score to a noisy category does not reduce noise. The fix is: disable the category, fix it offline with few-shot prompts, re-enable once false-positive rate is acceptable.

---

**Q36.**  
A document extraction pipeline sometimes invents values for a required field when the field is absent in the source document. What is the most effective schema fix?

A) Add a retry with exponential backoff.  
B) Add a `strict: true` flag to the schema.  
C) Change the field type to `["string", "null"]` or add `"unclear"` to its enum, and remove it from `required` if truly optional.  
D) Add a few-shot example where the field is populated.  

**Answer: C**  
> When a field is absent, retrying (A) increases hallucination pressure. The schema must allow `null` or an escape value like `"unclear"`. `strict: true` (B) stops the flow entirely on missing fields — it does not prevent hallucination when the field is marked required.

---

**Q37.**  
A team uses a plain prompt: "Extract the invoice data and return it as JSON." Claude occasionally returns prose with embedded JSON, or JSON with extra fields. What is the EXAM-PREFERRED fix?

A) Add "Return ONLY valid JSON, no prose" to the prompt.  
B) Post-process the response with a regex to extract JSON.  
C) Use the forced tool call pattern: define a tool with the exact schema and set `tool_choice` to force it.  
D) Use a higher temperature to make responses more structured.  

**Answer: C**  
> The forced tool call pattern guarantees structured output with no prose. The exam's preferred pattern is always forced tool call over prompt-only JSON extraction. `additionalProperties: false` prevents extra fields.

---

**Q38.**  
A batch API job processes 10,000 invoices. Some invoices need a follow-up tool call based on the first extraction result (e.g., if `status=disputed`, look up the dispute record). Can this multi-turn flow run in the Batch API?

A) Yes — the Batch API supports full multi-turn agentic loops.  
B) No — the Batch API supports a single forced tool call only; multi-turn flows require the synchronous API.  
C) Yes, with `max_turns=2`.  
D) Only if the second tool call is a `WebFetch`.  

**Answer: B**  
> The Batch API supports one forced tool call per request. Multi-turn conversations (where the second step depends on the first result) require the synchronous API.

---

**Q39.**  
A precision review pipeline flags 25% false positives in the "unclear variable names" category. Another category, "null pointer dereference," has 2% false positives. A proposed fix is to lower the overall strictness threshold to reduce total false positives. What is the problem with this approach?

A) Lowering strictness increases latency.  
B) Lowering strictness removes real findings from high-precision categories like null pointer dereference.  
C) Lowering strictness requires re-training the model.  
D) There is no problem — this is the correct approach.  

**Answer: B**  
> Reducing overall strictness hurts ALL categories, including high-precision ones. The correct fix is category-level: disable the noisy category, fix it offline, re-enable independently.

---

**Q40.**  
Few-shot examples are added to a review pipeline. After deployment, false positives drop not just on the examples provided but also on similar boundary cases that were NOT in the examples. What does this indicate?

A) The model is memorizing the examples — overfitting.  
B) The examples are working correctly — the model is generalizing.  
C) The model needs more examples to maintain performance.  
D) The drop in false positives is coincidental.  

**Answer: B**  
> Generalization (improvement on cases NOT in the few-shot set) is the evidence that few-shot examples are working. If performance only improves on the exact examples, that's memorization — not useful.

---

**Q41.**  
Confidence scores should be grounded in labeled data. An engineer proposes using a single confidence score per document. What is the correct approach?

A) One confidence score per document, derived from expert intuition.  
B) One confidence score per field, calibrated against a labeled dataset, with a threshold set from data.  
C) No confidence score — use the model's textual certainty instead.  
D) One confidence score per document type.  

**Answer: B**  
> Correct pattern: per-field scores (not per document), calibrated against labeled data (not expert intuition), threshold derived from data. Items above threshold → automate; below → human review.

---

**Q42.**  
A research pipeline collects market size data: Platform A reports 12% growth; Survey B reports 8% growth. How should the final report handle this conflict?

A) Average the two values: report 10% growth.  
B) Use the more recent source.  
C) Report both values with their sources, note the conflict, and provide a possible explanation without picking a winner.  
D) Use the higher value to be conservative.  

**Answer: C**  
> Never average conflicting values. Never ignore the conflict. Keep both + flag it. The report structure: contested claim → both values → source for each → possible explanation (different methods, different time periods).

---

**Q43.**  
A multi-pass review runs Pass 1 (per-file, parallel) and Pass 2 (integration, serial) in reverse order: Pass 2 first, then Pass 1. What problem does this cause?

A) Pass 2 cannot run without Pass 1's per-file findings — it has no local issues to aggregate.  
B) Pass 1 runs slower in serial.  
C) The model loses context between passes.  
D) There is no problem — order does not matter for multi-pass review.  

**Answer: A**  
> Order is mandatory: Pass 1 → Pass 2. Pass 2 (integration) looks for cross-file errors based on Pass 1 findings. Without Pass 1, Pass 2 has nothing to integrate. Running Pass 2 first means it runs on incomplete data.

---

**Q44.**  
A team wants to use the Batch API to process 50,000 contracts overnight. Before submitting the full batch, what is the recommended workflow?

A) Submit all 50,000 at once to minimize setup time.  
B) Build a representative sample, optimize the prompt synchronously, then submit the full corpus to the Batch API after a clean pass.  
C) Submit in batches of 1,000 and monitor each.  
D) Use the synchronous API for all 50,000 — the Batch API has a 24h SLA risk.  

**Answer: B**  
> Test small → batch corpus. Build a representative sample (one per document type, longest docs, all layouts). Optimize the prompt on the sample using the synchronous API (fast feedback). Only after one clean pass, submit to Batch API.

---

**Q45.**  
A pipeline is failing on a specific document layout. On investigation, the team finds 12 dismissed findings all share the pattern `"flag_pattern": "title_case_variable"`. What is the recommended next step?

A) Remove the "variable naming" category entirely.  
B) Increase the `max_tokens` limit.  
C) Add `detected_pattern` to the schema, group the dismissals by pattern, and build "do not flag" blocks + few-shot examples for that pattern.  
D) Submit the failing documents to human review.  

**Answer: C**  
> Learning from dismissed findings: add `detected_pattern` → group by pattern → build "do not flag" blocks + targeted few-shot examples → watch for generalization on unseen cases of the same pattern.

---

**Q46.**  
A document extraction schema has `additionalProperties: false` and `strict: true`. Claude's extraction returns a field that is not in the schema. What happens?

A) The field is included in the output alongside the schema fields.  
B) The flow stops — `strict: true` means no deviations from schema.  
C) `additionalProperties: false` prevents the extra field from appearing.  
D) Claude ignores `additionalProperties` unless the field is `required`.  

**Answer: C**  
> `additionalProperties: false` prevents Claude from adding fields not defined in the schema. `strict: true` stops the flow if a required field is missing — it does not block extra fields. Both play different roles.

---

## DOMAIN 4 — TOOL DESIGN & MCP (Questions 47–54)

---

**Q47.**  
A tool called `process_data` takes a customer record and returns a summary. After deployment, Claude calls it when asked to "search for orders," "get reports," and "check inventory." What is the most likely cause?

A) The tool has too many parameters.  
B) The tool's description is vague and does not include a "when NOT to use" boundary sentence.  
C) The tool should be split into three tools.  
D) Claude is using the wrong model.  

**Answer: B**  
> The 6-point description checklist includes a "When NOT" clause. Without it, the tool over-triggers on adjacent intents. Fix the description — do not change `tool_choice` to compensate.

---

**Q48.**  
Claude keeps calling `get_customer(id)` when the intent is actually to look up an order. Both `get_customer` and `get_order` take a single `id` parameter with identical descriptions. What is the BEST fix?

A) Force `tool_choice` to route based on user intent.  
B) Rename to `get_customer_by_id` and `get_order_by_id`.  
C) Consolidate into `lookup_entity(type, id)` with a single clear description.  
D) Add a few-shot example of correct routing.  

**Answer: C**  
> When multiple tools have identical input patterns and descriptions, consolidate them. This eliminates the routing ambiguity. Renaming (B) helps but doesn't fix overlapping semantics. `tool_choice` (A) is a workaround — the exam says fix the description, not force the routing.

---

**Q49.**  
A tool returns `{"results": []}` when a database query finds no matching records. Is this correct?

A) No — the tool should return `is_error: true` to signal the absence.  
B) No — the tool should raise an exception.  
C) Yes — empty result is a valid successful response with zero matches; `is_error` should be `false`.  
D) Yes, but it should also include a `"message": "No records found"` field.  

**Answer: C**  
> Empty result ≠ failure. An empty array with `is_error: false` (or omitted) is the correct signal for zero matches. Setting `is_error: true` would cause Claude to treat an empty search as an error and potentially retry unnecessarily.

---

**Q50.**  
An MCP server tool returns a `permission_error`. Which action is correct?

A) Retry the tool call with exponential backoff.  
B) Retry with corrected input.  
C) Escalate to a human — the agent lacks access and cannot retry its way out.  
D) Return an empty result to the coordinator.  

**Answer: C**  
> Permission errors are non-retriable. The agent lacks the access level needed — retrying won't change that. Escalate to a human for access remediation.

---

**Q51.**  
A project-level `.mcp.json` contains a GitHub MCP server configuration with the token hardcoded: `"GITHUB_TOKEN": "ghp_abc123xyz"`. What is the problem?

A) The file format is incorrect.  
B) Hardcoding credentials in a git-committed file exposes them to anyone with repo access.  
C) MCP servers do not support GitHub tokens.  
D) `.mcp.json` does not support the `env` field.  

**Answer: B**  
> Always use environment variable references: `"GITHUB_TOKEN": "$GITHUB_TOKEN"`. Hardcoding in a committed file exposes the credential to all repo contributors and git history.

---

**Q52.**  
An MCP tool returns an error: "Order total exceeds refund policy of 30 days." Should this be retried?

A) Yes — retry with a shorter time window.  
B) Yes — retry with corrected parameters.  
C) No — this is a business error (policy limit hit). Non-retriable. Explain to the user and offer an alternative.  
D) No — this is a permission error. Escalate.  

**Answer: C**  
> Business errors (policy limits, business rules) are non-retriable. The agent cannot retry its way past a policy constraint. The correct response is to explain the situation and offer alternatives (e.g., store credit, escalation).

---

**Q53.**  
A coordinator agent has 12 MCP tools from three different servers. During testing, the coordinator calls tools from all servers even when the task only needs one server's tools. What is the BEST fix?

A) Rename the tools to be more distinct.  
B) Remove unused tools from the session — only load the tools relevant to the coordinator's role.  
C) Add a `pre_tool_use` hook to block the unwanted servers.  
D) Set `tool_choice` to restrict to one server.  

**Answer: B**  
> Trim unused tools — 4-5 tools max per agent is the recommended ceiling. All MCP tools load at session start; removing unused ones reduces confusion, token waste, and incorrect routing. The exam rule: scope down, don't block.

---

**Q54.**  
When does the Glob tool apply vs. the Grep tool in the Agent SDK?

A) Glob = content search inside files; Grep = file name pattern matching.  
B) Glob = file name/path pattern matching; Grep = content search inside files.  
C) Both do the same thing — use either.  
D) Glob is for directories; Grep is for single files.  

**Answer: B**  
> Glob finds files by name/path pattern (e.g., `**/*.ts`). Grep searches content inside files. The pattern: Glob first (find the right files) → Grep (find the right content) → Read only if needed.

---

## DOMAIN 5 — CONTEXT MANAGEMENT & RELIABILITY (Questions 55–60)

---

**Q55.**  
An agent has used 75% of its context window and is about to start implementing a complex feature. What is the correct action?

A) Proceed — 25% remaining is sufficient for most implementations.  
B) Switch to a larger model with a bigger context window.  
C) Run `/compact` with a directed instruction specifying what to keep, then continue.  
D) Start a fresh session immediately.  

**Answer: C**  
> At 70-80% context used and BEFORE starting implementation: run directed compaction to free space. Directed = specify what to preserve (decisions, identifiers, constraints). Don't start implementation with an already-crowded context. Fresh session (D) is for when context is already degraded.

---

**Q56.**  
A directed `/compact` command is run. Which content survives compaction?

A) All turn history and tool results.  
B) Everything in the context window equally.  
C) Memory files on disk and the case-facts block. Turn history and tool results are summarized.  
D) Only the most recent 10 turns.  

**Answer: C**  
> Compaction summarizes turn history and tool results. It leaves untouched: files on disk (memory, scratchpad, case-facts JSON). The case-facts block is designed specifically to survive compaction by being stored outside the rolling context.

---

**Q57.**  
A support agent is handling a multi-turn conversation. The case involves an order ID, a customer ID, refund amounts, and open action items. After 20 turns, these key facts keep being referenced. What is the BEST strategy to prevent them from being lost during compaction?

A) Repeat the key facts in every user message.  
B) Store key facts in a dedicated case-facts block updated by deterministic code; reference it below the system prompt.  
C) Increase `max_tokens` to preserve more context.  
D) Run `/compact` before every turn.  

**Answer: B**  
> The case-facts lever: a dedicated structured block below the system prompt, updated by code (deterministic), designed to survive compaction. It ensures critical identifiers and state always remain accessible.

---

**Q58.**  
An agent encounters a customer request that is partially covered by policy (the standard resolution applies) but the customer is very angry. What should the agent do?

A) Escalate immediately — angry customers always get a human.  
B) Apply the policy resolution — sentiment alone is not an escalation trigger.  
C) Ask the customer if they want a human.  
D) Offer a higher refund than policy allows to defuse the situation.  

**Answer: B**  
> Escalation is policy-based, not sentiment-based. Angry + policy-covered → resolve. Calm + policy gap → escalate. Sentiment analysis as an escalation trigger is an explicit anti-pattern.

---

**Q59.**  
A document extraction pipeline has 97% overall accuracy. A team member proposes sampling 3% of documents at random for quality review. What is the problem with this approach?

A) 3% is too small a sample — use 10%.  
B) Random sampling may under-represent rare document types that have low accuracy; stratified sampling (3% from EACH type) is required.  
C) There is no problem — random sampling is statistically sound.  
D) Sampling should only happen after the pipeline reaches 99% accuracy.  

**Answer: B**  
> A 97% aggregate can hide a 60% accuracy rate on a small document type. Stratified sampling takes ~3% from EACH document type to surface hidden subgroup failures. The exam concept: aggregate metrics can mask per-category failures.

---

**Q60.**  
A codebase discovery subagent is given the full documentation folder (40MB of markdown) plus the source files it needs to analyze. Its context fills up before it can analyze any source files. What is the correct architectural fix?

A) Use a larger model with a bigger context window.  
B) Use subagent isolation — send the subagent only the selected path and source location, not full documentation.  
C) Run `/compact` before launching the subagent.  
D) Split the documentation into two subagents.  

**Answer: B**  
> Subagent isolation: send only what the subagent needs for its specific task — the selected path and source location. Full docs, reasoning chains, and coordinator history should never be dumped into a subagent. A bigger model (A) doesn't solve an architectural data-flow problem.

---

## ANSWER KEY

| Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|
| 1 | C | 16 | B | 31 | D | 46 | C |
| 2 | B | 17 | C | 32 | B | 47 | B |
| 3 | B | 18 | C | 33 | C | 48 | C |
| 4 | C | 19 | B | 34 | B | 49 | C |
| 5 | C | 20 | B | 35 | C | 50 | C |
| 6 | B | 21 | C | 36 | C | 51 | B |
| 7 | B | 22 | B | 37 | C | 52 | C |
| 8 | C | 23 | C | 38 | B | 53 | B |
| 9 | C | 24 | B | 39 | B | 54 | B |
| 10 | B | 25 | A | 40 | B | 55 | C |
| 11 | B | 26 | C | 41 | B | 56 | C |
| 12 | B | 27 | B | 42 | C | 57 | B |
| 13 | B | 28 | C | 43 | A | 58 | B |
| 14 | C | 29 | C | 44 | B | 59 | B |
| 15 | B | 30 | B | 45 | C | 60 | B |

---

## SCORING

| Correct | Score (est.) | Result |
|---------|-------------|--------|
| 60 / 60 | 1000 | Pass |
| 54 / 60 | ~900 | Pass |
| 48 / 60 | ~800 | Pass |
| 43 / 60 | ~720 | Borderline pass |
| < 43 | < 720 | Fail |

**Focus areas si fallaste preguntas:**
- Q1, Q9, Q17, Q34 → Hook precedence y pre vs. post semantics
- Q2, Q16, Q20 → AgentDefinition y coordinator wiring
- Q5, Q49 → Empty result ≠ error
- Q6, Q36, Q37 → Deterministic fixes vs. probabilistic fixes
- Q23, Q26 → dontAsk para CI; Hooks para zero-exception rules
- Q35, Q41 → Confidence score anti-patterns
