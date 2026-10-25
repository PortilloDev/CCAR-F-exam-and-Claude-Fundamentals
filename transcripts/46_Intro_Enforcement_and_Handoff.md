# Lecture 46: Intro: Enforcement and Handoff

**Section:** Agentic Architecture  
**Lecture ID:** 58006589

---

## Transcript

A support specialist receives a case from an agent with no context — no record of what was tried, why the workflow stopped, or what the customer's situation is. The specialist must repeat part of the investigation before they can help.

This lecture explains how to design **enforced workflows** (rules in code) and **useful handoffs** (structured information for humans to continue work).

### What this chapter covers

- Which requirements need enforcement **in code** vs. instructions left to the model.
- **Hooks** — deterministic checks that run at defined points around tool calls.
- Normalizing timestamps and status codes in `post_tool_use` hooks before Claude uses tool results.
- **Gates** — checks that block an action when a limit is exceeded or identity has not been verified.
- What a **human handoff** must contain: customer ID, order ID, amounts, actions taken, recommendations.
- Separating multiple customer concerns while preserving exact details.

**Goal:** Place necessary controls in a workflow and prepare a handoff someone can act on.
