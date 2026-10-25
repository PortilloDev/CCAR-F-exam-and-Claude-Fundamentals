# Lecture 45: Lab 2: Multi-Agent Research Pipeline

**Section:** Agentic Architecture  
**Lecture ID:** 58217517

---

## Transcript

### Pipeline architecture

```
Research Coordinator (hub)
├── Web Searcher subagent  (search_web, load_document)
└── Document Searcher subagent  (search_internal, load_document)
          ↓ (fan-out per topic: visual art × 2 + music × 2)
     Results aggregated by coordinator
          ↓
Formatter Agent  (verify_fact only)
          ↓
report.json  (well-established / contested / coverage gaps)
```

---

### Goal-based research prompt structure

```python
topic = "AI in the creative industries"
goal = "Survey how AI is changing the creative industries across both fields below."
fields = ["visual art and design", "music and audio"]
criteria = [
    "Every finding carries a source URL, a verbatim excerpt, a publisher, and the period the source measured.",
    "Where two sources disagree, both values are kept with their publisher and period.",
    # ...
]
```

---

### MCP tool structured responses

```python
# Failure response
def failed(failure_type, query, partial_result, alternatives, coverage_impact):
    return {
        "status": "partial_failure",
        "failure_type": failure_type,
        "attempted_query": query,
        "partial_result": partial_result,
        "alternative_approaches": alternatives,
        "coverage_impact": coverage_impact,
    }

# Success response
def succeeded(channel, query, hits):
    return {
        "status": "success",
        "channel": channel,    # "web" or "internal"
        "attempted_query": query,
        "hits": hits,
    }
```

---

### Coordinator configuration

```python
options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    mcp_servers=[research_mcp_server],
    system_prompt="You delegate. Each subagent starts with an empty context — whatever it needs must be in the prompt.",
    tools=["Task", "Agent"],     # required to spawn subagents
    allowed_tools=["Task", "Agent"],
    agents={
        "web_searcher": web_searcher_agent,
        "document_searcher": document_searcher_agent,
    },
)
```

---

### Formatter agent (structured output with Pydantic)

```python
from pydantic import BaseModel, model_validator

class ResearchReport(BaseModel):
    topic: str
    well_established: list[dict]   # claims with source citations
    contested: list[dict]          # conflicting sources
    coverage_gaps: list[str]

    @model_validator(mode="after")
    def check_citations(self):
        for finding in self.well_established:
            if not finding.get("source_url"):
                raise ValueError(f"Claim missing source_url: {finding['claim']}")
        return self
```

Formatter agent has access only to `verify_fact` — no research tools — and uses `output_format` to enforce the Pydantic schema.

---

### Key takeaways

- Fan-out (parallel subagents per topic × per channel) dramatically reduces total research time.
- Structured tool responses (with `failure_type`, `partial_result`, `coverage_impact`) give the coordinator actionable information even on failure.
- The formatter agent validates provenance (every claim must have a source URL) using a Pydantic `model_validator`.
- Coverage gaps are explicitly reported in the output rather than silently omitted.
