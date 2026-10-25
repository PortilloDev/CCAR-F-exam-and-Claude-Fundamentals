# Lecture 34: Agent SDK: Loading From Disk

**Section:** Agentic Architecture  
**Lecture ID:** 58188347

---

## Transcript

### Configuration scopes

| Scope | Location | Shared? | Contains |
|-------|----------|---------|----------|
| **User** | `~/.claude/` | No (personal) | `settings.json` (permissions, hooks), `CLAUDE.md`, skills, sub-agents |
| **Project** | Project root | Yes (git) | Same as user — scoped to repo |
| **Local** | Project root (gitignored) | No (personal per repo) | Per-developer overrides |

**`CLAUDE.md`** — a text file loaded into every conversation on top of the system prompt. Use it to set rules, boundaries, and agent behavior guidelines.

**Skills** — structured documents containing prompts and best practices. They teach new behavior to the agent.

**Plugins** — combine MCP tools + skills into a single package (from Anthropic or third parties).

---

### Setting up `setting_sources` in code

```python
options = ClaudeAgentOptions(
    model="claude-sonnet-5",
    cwd="/path/to/repo",                    # working directory
    setting_sources=["project"],            # auto-load project folder config
    agents={"reviewer": reviewer_agent},    # additional agents on top
    plugins=["my-plugin"],                  # additional plugins
)

# At startup, log what was loaded:
print(options.loaded_skills)
print(options.loaded_plugins)
```

---

### System prompt options

| Option | What it provides |
|--------|----------------|
| Claude Code system prompt | Tool guidance + Anthropic safety rules + environment context (optimized) |
| Minimalistic system prompt | Tool calling guidance only — no safety rules |
| Custom string | Fully custom — you own everything |

---

### Workspace isolation strategy

| Strategy | `setting_sources` | Use when |
|----------|------------------|----------|
| **Isolated** | Empty `[]` | Each agent gets its own dedicated workspace |
| **Shared** | Point to team workspace | Multiple agents share a working directory |

This choice determines **tenant isolation** — one workspace per tenant (isolated) vs. a shared space with scoped access.
