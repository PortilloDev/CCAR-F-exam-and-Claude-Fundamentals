# Lecture 68: Working Modes: Permission Modes

**Section:** Claude Code in a Team  
**Lecture ID:** 58217079

---

## Transcript

### Permission modes in Claude Code

| Mode | What it does | Use when |
|------|-------------|---------|
| **plan** | Read-only discovery; proposes plan for review | Complex/exploratory tasks |
| **default** | Everyday tasks; asks approval for specific tools | Interactive development |
| **acceptEdits** | Auto-approves file reads and writes | Let Claude edit, then review result |
| **auto** | AI classifier decides; escalates dangerous tools | Reduce approval fatigue interactively |
| **dontAsk** | No prompts — allowed_tools list is enforced | CI/CD pipelines (critical for exam) |
| **bypassPermissions** | All tools allowed, no checks | Isolated containers only — dangerous |

---

### Switching modes in Claude Code terminal

```bash
# Interactive toggle (Shift+Tab):
# cycles through: manual → acceptEdits → plan → auto

# Start session in dontAsk or bypassPermissions:
claude --permission-mode dontAsk --allowedTools "Read,Grep,Bash(git diff)"
```

---

### CI/CD pipeline pattern (exam critical)

```bash
claude -p "Review the PR for security issues" \
  --output-format json \
  --json-schema ./review-schema.json \
  --max-turns 10 \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Bash(git diff)"
```

**`dontAsk` is mandatory for CI/CD** — any permission prompt would block the pipeline.

---

### auto mode clarification

Auto mode uses a classifier to detect dangerous tool calls and escalates those. It does **not**:
- Auto-correct mistakes.
- Verify whether results are correct.

It only classifies tool call danger level — everything else proceeds automatically.

---

### bypassPermissions — two overrides still apply

Even in `bypassPermissions` mode, these still block tool usage:
1. `pre_tool_use` hooks that return `denied`.
2. Entries in `disallowed_tools`.

These are the only mechanisms that can override `bypassPermissions`.
