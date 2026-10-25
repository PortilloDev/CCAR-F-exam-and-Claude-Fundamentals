# Lecture 110: Exam: Domains Recap

**Section:** Exam Readiness and Practice Tests  
**Lecture ID:** 58217307

---

## Transcript

### Domain 1: Agentic Architecture and Orchestration

| Question | Answer |
|----------|--------|
| Loop ends wrongly — where do you look? | `stop_reason`, not response text |
| Research pipeline has coverage gaps | Coordinator needs better task decomposition — one subagent per missing topic |
| Context missing in subagent | Improve coordinator → subagent context handoff |
| Important policy step got skipped | `pre_tool_use` hook — deterministic enforcement |
| Fixed plan vs. adaptive plan? | Fixed (prompt chaining) when steps are known; adaptive when codebase/scope is unknown |
| Agent crashed — resume or restart? | Resume if world around it unchanged; restart if files changed significantly |

**Pitfalls:**
- Subagents don't spawn sub-subagents (hub-and-spoke only).
- `post_tool_use` hooks cannot block a call — only `pre_tool_use` can.
- Subagents default to parallel — avoid sequential unless order matters.

---

### Domain 2: Tool Design and MCP Integration

| Question | Answer |
|----------|--------|
| Agent picked wrong tool | Expand tool description with more criteria |
| Two tools overlap | Rename one; rewrite both descriptions |
| Tool failed | Transient error → retry; business error → return structured error with category |
| Agent has too many tools | Scope tool access — assign only relevant tools |
| Share MCP server with team | `project mcp.json`; credentials as env vars |
| Glob vs. Grep | Glob = file/path search; Grep = content inside files |

**Pitfalls:**
- Never force tool choice (`tool_choice`) to fix a wrong tool selection — fix the description.
- Empty result ≠ failure. Empty should be a valid success.
- Don't disable Grep because an MCP tool wasn't called — improve MCP name/description.

---

### Domain 3: Claude Code Configuration and Workflows

| Question | Answer |
|----------|--------|
| Teammates have different CLAUDE.md versions | `/memory` to see which files are loaded; move to project root |
| Rule needed every session | CLAUDE.md |
| Rule needed for specific task only | Skill |
| Rules under certain paths only | Path-specific CLAUDE.md with frontmatter |
| Plan mode vs. direct execution | Plan for 40+ files; direct for small bug fixes |
| Instruction misread — edge case? | Add few-shot examples for the edge case |
| Claude Code in CI pipeline | JSON mode + `--permission-mode dontAsk` + `--allowedTools` + `-p` flag |

**Pitfalls:**
- `/memory` is read-only — it shows which files are loaded, doesn't change scope.
- CLAUDE.md in subdirectories extends the parent — doesn't overwrite.
- In CI, don't forget `-p` to disable interactive mode.

---

### Domain 4: Prompt Engineering and Structured Output

| Question | Answer |
|----------|--------|
| Vague criteria / edge cases | Make criteria explicit + add few-shot examples |
| Need structured output | Define a tool with input schema; force tool call; read `tool_block.input` |
| Extraction fails semantic check | Add deterministic semantic validation in code |
| Batch vs. synchronous | Batch: save cost, tolerate 24h; synchronous: tight SLA |
| Who reviews — current session or fresh? | Always a fresh session — current session is biased |

**Pitfalls:**
- "Be conservative" without specific criteria → vague, unreproducible.
- Extracting a field that never appears in the document → hallucination risk.
- Required field that can legitimately be absent → use nullable.

---

### Domain 5: Context Management and Reliability

| Question | Answer |
|----------|--------|
| Facts lost after compaction | Dedicated facts block in system prompt; update with deterministic code |
| When to escalate | Customer asks for human; policy gap; task outside agent's tools |
| Subagent failed — what to report? | Failure type + queries tried + partial results |
| Huge discovery task → context flood | Delegate to subagent (fresh context); or chunk into multiple subagents |
| What to automate vs. human review | Calibrated confidence score per field + stratified sampling by document type |
| Sources disagree | Keep both; transparently report conflict; never average |

**Pitfalls:**
- `/compact` with no instruction → add a custom instruction to preserve important facts.
- Escalating on sentiment alone → wrong; use policy criteria.
- Information missing from tools → agent should ask the user (if user can know it).

---

### Fix complexity hierarchy (innermost = cheapest)

```
1. Prompt / CLAUDE.md rule change (cheapest)
2. Few-shot examples for boundary cases
3. Tool name / description / schema changes
4. Multi-agent architecture (scope, roles)
5. Hooks + validation + retry logic (most complex)
```

**Always try the innermost fix first.**
