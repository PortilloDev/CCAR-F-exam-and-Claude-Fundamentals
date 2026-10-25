# Lecture 29: Intro: The Agent SDK

**Section:** Agentic Architecture  
**Lecture ID:** 58006583

---

## Transcript

When building an agent, you choose how much of its execution your application manages. Three options:

1. **Claude API (Client SDK)** — raw stateless API; you manage the loop yourself.
2. **Agent SDK** — higher-level library with the loop built in.
3. **Claude Code CLI** — customer-facing product; run in terminal, desktop, or headless mode.

### What this chapter covers

- Responsibilities the SDK handles (agentic loop, tool execution).
- Comparing raw API loop vs. SDK vs. CLI.
- The `options` object and how its fields configure a run.
- Tool settings: `allowed_tools` (pre-approve without prompting) vs. the tools list (what the agent can access).
- Key distinction: **a tool not in `allowed_tools` is not necessarily unavailable** — it may still be accessible but require runtime approval.

**Goal:** Choose an integration approach, configure an SDK run, and explain which tools the agent can access.
