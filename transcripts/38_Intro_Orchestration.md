# Lecture 38: Intro: Orchestration

**Section:** Agentic Architecture  
**Lecture ID:** 58006585

---

## Transcript

When a research task spans several topics, work can be divided among specialized agents. Key challenges:
- Assignments must cover the whole question without gaps.
- Findings must be combinable into a coherent answer.
- The coordinator must handle errors and retries.

### What this chapter covers

- Coordinator's role: select only the agents the query needs, assign work, handle errors, return the answer.
- How a subagent's **description**, **system prompt**, **tool list**, and **model** define its role.
- Why the coordinator needs approval to delegate (Task/Agent tool in `allowed_tools`).
- Why findings must be **passed explicitly** between isolated conversations (with sources and dates).
- Launching independent tasks in parallel and dividing scope to avoid duplicate work.
- Using **goals and quality criteria** to check the combined answer, request targeted follow-up, and iterate until coverage is sufficient.

**Goal:** Design suitable assignments, pass useful context, and correct gaps in the coordinator's plan.
