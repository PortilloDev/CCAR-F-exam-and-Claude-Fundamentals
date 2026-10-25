# Study Doc — Section 7: Exam Readiness

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
