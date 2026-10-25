# Lecture 59: Configuration: Writing CLAUDE.md Rules

**Section:** Claude Code in a Team  
**Lecture ID:** 58217043

---

## Transcript

### Key principles for effective rules

1. **CLAUDE.md always loads into the context window** (unless directory-scoped — then on demand).
2. Claude **weighs rules automatically** — not every rule applies to every task.
3. **Add verifiable steps** — rules that can be checked with code (e.g., "run tests after changes") are far more effective than vague guidance.
4. Use sub-files for long rule sets — reference them from the main file.
5. **Use `IMPORTANT` sparingly** — overusing it dilutes its effect.

---

### Specificity: vague vs. good rules

| ❌ Vague | ✅ Specific |
|---------|----------|
| "Follow best practices for API routes" | "For new API routes, create one file per route under `src/api/handlers/`" |
| "Don't use default exports" | "Use named exports, not default exports" |
| "Handle errors properly" | "Test the happy path AND error path for every new handler" |
| "Follow best practices" | (already in model pre-training — don't repeat it) |

**Rule:** Always pair a prohibition with its alternative.

---

### `IMPORTANT` flag — use selectively

Mark sensitive topics `IMPORTANT` (secrets, migrations). Leave day-to-day conventions (commit format, export style, test location) unmarked. This focuses Claude's attention where it matters.

---

### Well-structured CLAUDE.md example

```markdown
## Commands
- Run tests: `npm test`
- Build: `npm run build`

## Testing
- Test files live in `__tests__/` alongside the source file.
- Every new handler needs tests for: happy path, error path.
- Mock all external calls: Stripe, S3, email module.

## IMPORTANT: Secrets
- Never hardcode credentials. Use environment variables only.
```
