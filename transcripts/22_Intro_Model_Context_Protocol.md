# Lecture 22: Intro: Model Context Protocol (MCP)

**Section:** Tools and MCP  
**Lecture ID:** 58006579

---

## Transcript

When multiple teams connect Claude to the same issue tracker, they end up maintaining separate versions of the same tools. Each version requires updates when the service changes, and small differences produce inconsistent results.

**Model Context Protocol (MCP)** provides a shared way to expose capabilities — a standard server sits between a service and the applications that use it.

### What this chapter covers

- How an MCP server exposes a service for reuse across applications.
- The three MCP primitives:
  - **Tools** — controlled by the model
  - **Resources** — accessed by the application
  - **Prompts** — selected by the user
- Adopting an existing server vs. building a custom one using decorated functions.
- Four error types: transient, validation, business, and permission — each requiring a different response.
- Shared project settings, personal servers, and credentials via environment variables.
- Local recovery and how to report remaining failures with attempted actions and partial results.

**Goal:** Choose an integration approach and provide errors that help the agent decide what to do next.
