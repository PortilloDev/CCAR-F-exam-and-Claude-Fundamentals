# Lecture 72: Automation: GitHub Action

**Section:** Claude Code in a Team  
**Lecture ID:** 58217099

---

## Transcript

### GitHub Action workflow structure

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          prompt: |
            Review this PR for security vulnerabilities.
            Output findings as JSON matching the schema below.
            ...
          output-format: json
          permission-mode: dontAsk
          allowed-tools: "Read,Grep,Bash(git diff)"
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

### claude-code-action inputs

| Input | Description |
|-------|-------------|
| `prompt` | The task prompt sent to Claude Code |
| `output-format` | `json` for structured output |
  | `permission-mode` | Use `dontAsk` — required for CI |
| `allowed-tools` | Comma-separated tool whitelist |

---

### Inline PR comments

The action can post findings directly as inline PR review comments on the diff:

```yaml
- name: Claude Code Review
  uses: anthropics/claude-code-action@v1
  with:
    prompt: "Review for security issues and post inline comments"
    post-inline-comments: true
```

Claude reads the diff, identifies line-level issues, and posts GitHub review comments anchored to the specific lines — reviewers see findings in context.

---

### Structured findings format (exam pattern)

When using `--output-format json`, the prompt should instruct Claude to produce a predictable schema:

```json
{
  "findings": [
    {
      "file": "src/auth.py",
      "line": 42,
      "severity": "high",
      "category": "sql-injection",
      "description": "Unsanitized user input passed to query string"
    }
  ]
}
```

Downstream CI steps can parse this JSON to fail the build on high-severity findings, post to Slack, or file issues.

---

### Trigger: PR comment

The action can also trigger on PR review comments — a developer comments `/claude review` and the action runs:

```yaml
on:
  issue_comment:
    types: [created]

jobs:
  review:
    if: github.event.comment.body == '/claude review'
    ...
```
