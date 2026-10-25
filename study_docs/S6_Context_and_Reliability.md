# Study Doc — Section 6: Context and Reliability

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
