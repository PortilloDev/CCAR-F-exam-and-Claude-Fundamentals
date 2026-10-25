# Lecture 55: Decomposition & Sessions: Resume, Fork or Restart

**Section:** Agentic Architecture  
**Lecture ID:** 58217035

---

## Transcript

### Decision tree: Resume, Fork, or Restart?

```
Is prior context mostly still valid?
├── NO  → Start fresh (with a summary from the old session)
└── YES → Do you want to compare two approaches from here?
           ├── YES → Fork the session
           └── NO  → Resume the session
```

---

### Resume

```bash
# Claude Code CLI
claude --name "my-session-name"
claude --resume <session-id>
```

```python
# Agent SDK
options = ClaudeAgentOptions(resume="<session-id>", ...)
await query(prompt="Two files changed: auth.py and config.py. Re-examine those.", options=options)
```

**When to resume:** ~80-90% of prior context is still valid. Communicate changed files explicitly in the new prompt.

---

### Fork

```python
# Agent SDK
options = ClaudeAgentOptions(resume="<baseline-session-id>", fork_session=True, ...)
```

Creates a new branch from the baseline session — new session ID, original baseline preserved. Multiple forks can be created from the same baseline.

**Use case:** After a code analysis baseline, fork one session to refactor Project A and another to refactor Project B — independent explorations from the same starting point.

**Critical:** The baseline must still be valid. A stale baseline makes all forks inherit the stale context.

---

### Start fresh

```python
# Pass a summary to bootstrap the new session
options = ClaudeAgentOptions(...)  # no resume field
prompt = """
Previously: test coverage implementation.
Status: 15 gaps found, 3 fixed.
Task: Fix the remaining 12 gaps.
"""
await query(prompt=prompt, options=options)
```

**When:** Many things have changed since the last session, or the baseline is no longer trustworthy. Provide a targeted summary rather than resuming stale context.

---

### What a session stores

- Full chat history (user ↔ agent).
- Tool call results.
- File contents read during the session.

**Risk on resume:** Files may have changed since the session. Always tell the agent which files changed so it can re-read them rather than using cached (stale) results.
