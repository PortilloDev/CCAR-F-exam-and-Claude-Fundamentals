# Lecture 71: Automation: Headless Mode

**Section:** Claude Code in a Team  
**Lecture ID:** 58217091

---

## Transcript

### Core headless flags

```bash
claude -p "Review this PR for security issues" \
  --output-format json \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Bash(git diff)" \
  --max-turns 10
```

| Flag | Purpose |
|------|---------|
| `-p "<prompt>"` | Non-interactive single prompt (headless entry point) |
| `--output-format json` | Structured JSON output instead of conversational text |
| `--permission-mode dontAsk` | No approval prompts — mandatory for pipelines |
| `--allowedTools "..."` | Whitelist of tools Claude can use |
| `--max-turns <n>` | Hard cap on agentic loop iterations |

---

### --output-format options

| Value | Output |
|-------|--------|
| `text` (default) | Conversational text |
| `json` | Structured JSON — use for downstream parsing |
| `stream-json` | Streaming JSON events |

---

### --bare flag

```bash
claude -p "..." --bare
```

Strips all formatting metadata — returns the raw text of the final response only. Use when you need a plain string and no JSON overhead.

---

### Combining with --json-schema

```bash
claude -p "Review this PR" \
  --output-format json \
  --json-schema ./review-schema.json
```

Forces Claude to produce output that validates against the provided JSON schema. The schema acts as a contract between the Claude Code subprocess and the downstream consumer.

---

### Exam pattern: minimal safe headless invocation

```bash
claude -p "<task>" \
  --output-format json \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep"
```

**Why `dontAsk` is required:** any permission prompt in a pipeline blocks indefinitely — no human is watching to approve it.

**Why restrict `--allowedTools`:** limits blast radius; Claude cannot run arbitrary shell commands in CI.
