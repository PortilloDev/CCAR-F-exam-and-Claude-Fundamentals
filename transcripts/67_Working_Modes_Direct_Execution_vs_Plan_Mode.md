# Lecture 67: Working Modes: Direct Execution vs. Plan Mode

**Section:** Claude Code in a Team  
**Lecture ID:** 58217075

---

## Transcript

### Plan mode

```bash
claude --permission-mode plan
```

Creates a sub-agent with a **fresh context** and access to `Read`, `Grep`, `Glob` only (no write tools). The agent discovers the project and returns a plan proposal — you review and approve before implementation begins.

---

### Four signals to use plan mode (exam list)

| Signal | Example |
|--------|---------|
| Complex architecture | Microservices refactor, auth system redesign |
| Multiple alternative approaches | Slack integration: webhook vs. bot token vs. full app |
| Large file scope (>45 files) | Replace deprecated HTTP client spread across 45+ files |
| Multi-step exploration of unknown codebase | First need to understand architecture, modules, code quality |

**If none of these apply** → use direct execution.

---

### Three exam use cases — plan vs. direct

| Task | Mode | Reason |
|------|------|--------|
| Date field accepts "Feb 31" | Direct | Simple, well-defined fix |
| Add Slack to existing email+SMS notifier | **Plan** | Multiple alternative architectures |
| Replace deprecated HTTP client in 45 files | **Plan** | Large file scope |

---

### Plan mode mid-session

You can trigger plan mode during a conversation when you realize a change is larger than expected:
1. Plan sub-agent spins up in fresh context.
2. Runs discovery (read-only).
3. Returns proposal to main conversation.
4. You review and approve.
5. Main conversation executes.

---

### Why reviewing the plan matters

- **Changing a plan is cheap** (minutes, few line edits).
- **Undoing code changes is expensive** (hundreds of files, hours).
- Skimming and blindly approving = accepting unreviewed assumptions.
- Always read the plan thoroughly before approving.
