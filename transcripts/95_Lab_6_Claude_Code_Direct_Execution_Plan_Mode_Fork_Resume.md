# Lecture 95: Lab 6: Claude Code Direct Execution, Plan Mode & Fork/Resume

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217533

---

## Transcript

### Lab objective

Work with a sample repository (`myCorp`) simulating a company with shop, payment, ordering, and notification services. Practice direct execution, plan mode, and session management (resume and fork).

---

### Repository structure

- `CLAUDE.md` — conventions (e.g., validator returns `None` if acceptable, or short message if wrong)
- `manifest.json` — structured state preservation for session resumption
- `notepad` — prose notes for long-term preservation
- Architecture Decision Records (ADRs) — including undecided refund notification design

---

### Task 1: Direct execution (small, well-defined bug)

**Bug:** `check_value()` raises `TypeError` when comparing string input to a float.  
**Fix:** return a text message instead of crashing.

```python
# Before: raises TypeError on string input
if value < 0:  # fails if value is "abc"

# After: validate type first
if not isinstance(value, (int, float)):
    return "refund amount must be a number"
```

**Why direct execution:** small, well-defined fix. No architecture decision needed.

**Permission mode used:** `auto` (Shift+Tab to cycle in Claude Code terminal).

---

### Task 2: Plan mode (architecture decision)

**Decision:** refund notification — two designs:
- **Design A:** `send_refund()` calls notification client inline after gateway succeeds. Must swallow notification exceptions (ADR rule: refund can never fail).
- **Design B:** write to `outbox.py`, let notifications worker drain it. Same-transaction write.

**Why plan mode:** complex architecture decision with alternative paths.

**Workflow:**
1. Switch to plan mode (Shift+Tab).
2. Paste prompt — Claude spins up a **sub-agent with fresh context** for discovery.
3. Sub-agent reads repository (read-only).
4. Returns plan with Design A and Design B — recommends Design B.
5. **Review the plan before approving** — plan changes are cheap; code changes are expensive.

---

### Claude Code terminal commands

| Command | Purpose |
|---------|---------|
| `/memory` | Show all loaded memory (CLAUDE.md, personal md, auto-memory) |
| `/compact [custom prompt]` | Compact session context with optional instructions |
| `/effort` | Set model effort level (low/medium/high) |
| `/model` | Switch model |
| `/mcp` | List MCP servers |
| `Shift+Tab` | Cycle permission modes (manual → acceptEdits → plan → auto) |

---

### Session state preservation

Two mechanisms for long-term state:
- **`manifest.json`** (structured) — for the agent; allows safe session resumption.
- **`notepad`** (prose) — human-readable notes.

**Why both:** if a session breaks, the manifest allows the next session to understand exactly where work left off.

---

### `.claude/settings.json` (project configuration)

```json
{
  "model": "claude-sonnet-5",
  "permissions": {
    "allow": ["Edit", "Write", "Read", "Bash(git add)", "Bash(git commit)", "Bash(pytest)"],
    "deny":  ["Bash(git push)"]
  }
}
```

Git push is blocked. Editing and local git operations are allowed.

---

### Fork and resume

- **Resume:** reopen the same session — continues from last state (uses manifest.json to recover safely).
- **Fork:** branch the session to explore an alternative path without affecting the original.

Use fork when: evaluating Design A vs. Design B without committing to either yet.  
Use resume when: session was interrupted and must continue from the same point.
