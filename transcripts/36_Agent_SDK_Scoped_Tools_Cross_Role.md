# Lecture 36: Agent SDK: Scoped Tools & Cross-Role Exception

**Section:** Agentic Architecture  
**Lecture ID:** 58188365

---

## Transcript

### Why tool scope matters

Each agent should only see the tools relevant to its role. Adding extra tools causes:
- Agent drifts from its intended behavior.
- Performance drops (it does the extra task worse than the specialist agent).
- Separation of concerns blurs.

**Tools list ≠ pre-approval.** The `tools` field defines what the agent CAN access. `allowed_tools` defines what runs WITHOUT a prompt.

---

### Ideal scoping example

| Agent | Tools |
|-------|-------|
| Search agent | `WebSearch`, `WebFetch` |
| Synthesis agent | `Write`, `Edit` |
| Coordinator | `Agent` (to spawn sub-agents) |

If synthesis gets `WebSearch` → it starts doing its own searches instead of writing, reducing quality and blurring roles.

---

### Enforcing scope in `dontAsk` mode

```python
# dontAsk + allowed_tools = hard boundary
synthesis_agent = AgentDefinition(
    tools=["Write", "Edit"],
    allowed_tools=["Write", "Edit"],
    permission_mode="dontAsk",
    # WebSearch is not in tools or allowed_tools → cannot be called at all
)
```

In `default` mode, unlisted-but-available tools can still be approved at runtime — `dontAsk` makes the boundary hard.

---

### Constrained tools

Prevent hallucination-driven misuse by wrapping built-in tools:

```python
# Constrained WebFetch — only internal documentation URLs
@tool(name="fetch_doc", description="Fetch internal documentation by URL. Do not use for external sites.")
def fetch_doc(url: str) -> str:
    if not url.startswith("https://internal.docs."):
        raise ValueError("Only internal documentation URLs are allowed.")
    return requests.get(url).text
```

Replace the generic `WebFetch` with this tool in the agent's `tools` list.

---

### Cross-role exception (when breaking scope is justified)

**Scenario:** Synthesis agent needs to verify dates/names but must loop through the coordinator to use WebSearch → many extra turns → high latency.

**Solution:** Give the synthesis agent a narrow, purpose-built fact-check tool (e.g., `check_name_or_date(query)`) that only handles names and dates — not full web search.

**Rule:** Expand scope only when:
1. The extra capability is narrow and purpose-specific.
2. The alternative creates measurable latency or coordination overhead.
3. The expanded tool cannot be misused for broader tasks.
