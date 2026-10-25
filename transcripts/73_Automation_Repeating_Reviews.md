# Lecture 73: Automation: Repeating Reviews

**Section:** Claude Code in a Team  
**Lecture ID:** 58217101

---

## Transcript

### The core pattern

Run the same review on every PR — always fresh, never anchored to prior findings.

```bash
claude -p "$(cat review-prompt.md)" \
  --output-format json \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Bash(git diff)"
```

---

### CLAUDE.md as the review standard

Put the review criteria in CLAUDE.md (or a referenced skill):

```markdown
# PR Review Standards

When reviewing a PR:
1. Check for SQL injection and XSS vulnerabilities
2. Verify all new API endpoints are authenticated
3. Confirm no secrets are committed
4. Ensure error responses do not leak stack traces
5. Check that logging does not include PII
```

**Why CLAUDE.md:** the review standard lives in the repository alongside the code. It evolves with the team's conventions. The reviewer session loads it automatically.

---

### Independent reviewer session (no anchoring)

Always start a **fresh Claude Code session** per review — do not reuse a session across PRs:

```bash
# New session per PR — no prior conversation context
claude -p "Review the diff for the standards in CLAUDE.md" \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Bash(git diff origin/main...HEAD)"
```

**Why fresh:** a session that reviewed a previous PR may be anchored to patterns from that code. A fresh session reads only the current diff.

---

### Feeding previous findings as context

To avoid re-reporting already-resolved issues, pass the prior findings:

```bash
claude -p "
Review the current diff against standards in CLAUDE.md.
Previous findings (already resolved — do NOT re-report):
$(cat previous-findings.json)

New diff:
$(git diff origin/main...HEAD)
" \
  --output-format json \
  --permission-mode dontAsk
```

---

### Explicit review criteria (exam pattern)

The prompt should name criteria explicitly, not rely on Claude's general judgment:

```
Review for:
1. Authentication on all new endpoints
2. No hardcoded secrets
3. SQL query parameterization
4. PII excluded from logs

Output JSON with one entry per finding.
```

**Why explicit:** repeatable reviews require deterministic criteria. "Review for security" produces variable output; "check for these 4 specific patterns" produces comparable output across runs.
