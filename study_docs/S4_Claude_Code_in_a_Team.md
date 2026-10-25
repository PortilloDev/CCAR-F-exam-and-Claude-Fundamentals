# Study Doc — Section 4: Claude Code in a Team

**Domain:** Claude Code Configuration and Workflows  
**Files:** 57–75

---

## CLAUDE.md — Configuration That Gets Followed

### Hierarchy (innermost wins)

```
Global (~/.claude/CLAUDE.md)
  └── Project root (./CLAUDE.md)
        └── Subdirectory (./src/CLAUDE.md)  ← extends parent, doesn't overwrite
```

### Writing effective rules

| Write | Avoid |
|-------|-------|
| "Never use `var` — use `const` or `let`" | "Write modern JS" |
| "All API endpoints must be authenticated" | "Follow security best practices" |
| "Return `null`, never throw, for missing records" | "Handle errors properly" |

Rules fail when they use vague adjectives. Make them specific and testable.

### Splitting CLAUDE.md

Put in CLAUDE.md: standing conventions that apply all the time.  
Move to skill: task-specific procedures (PR review, deployments, migrations).  
Move to hook: zero-exception business rules.

**EXAM TRAP:** A 400-line CLAUDE.md causes context overflow. Move task-specific content to skills.

---

## Path-Specific Rules (frontmatter format)

```markdown
---
globs: ["src/api/**/*.ts", "tests/**/*.test.ts"]
---
# API rules — only loaded for these paths
All endpoints must validate authentication.
```

**EXAM TRAP:** Path rules are loaded based on the FILES being worked on, not the directory you're in.

---

## Skills vs. CLAUDE.md vs. Hooks

| | CLAUDE.md | Skill | Hook |
|--|-----------|-------|------|
| When loaded | Always | Description always; body on demand | Always on pattern match |
| Enforcement | Probabilistic | Probabilistic | **Deterministic** |
| Best for | Standing conventions | Task-specific procedures | Zero-exception rules |

### Decision tree

```
Should this rule NEVER be skipped?
├── YES → Hook (pre_tool_use shell script)
└── NO  → Should it always apply?
           ├── YES → CLAUDE.md
           └── NO  → Skill (task-based, on demand)
```

**EXAM TRAP:** Need to block `psql` against production? CLAUDE.md = probabilistic (will eventually be ignored). Use a hook.

---

## Working Modes

### Plan mode vs. direct execution

Use **plan mode** when:
- Complex architecture decision
- Multiple alternative approaches exist
- Large file scope (40+ files)
- Multi-step exploration of unknown codebase

Use **direct execution** for: small, well-defined bug fixes.

**Why plan mode matters:** changing a plan is cheap (minutes). Undoing code is expensive (hours, hundreds of files).

### Permission modes

| Mode | Use when |
|------|---------|
| `plan` | Complex tasks needing exploration first |
| `default` | Interactive development |
| `acceptEdits` | Let Claude edit; review result |
| `auto` | Reduce approval fatigue interactively |
| `dontAsk` | **CI/CD pipelines — mandatory** |
| `bypassPermissions` | Isolated containers only |

**EXAM TRAP:** `dontAsk` is required for CI — any approval prompt blocks a pipeline indefinitely.  
**EXAM TRAP:** `bypassPermissions` still respects `pre_tool_use` hooks that return `denied` and `disallowed_tools`.

---

## Iterative Refinement

| Technique | When |
|-----------|------|
| Few-shot input/output examples | After 2+ text iterations still failing |
| TDD (write tests first) | Any implementation task |
| Interview mode | Unknown domain or multiple solution paths |
| Batch feedback | Issues interact with each other |
| Sequential feedback | Issues are independent |

---

## Headless Mode (CLI flags)

```bash
claude -p "<prompt>" \
  --output-format json \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Bash(git diff)" \
  --max-turns 10
```

| Flag | Purpose |
|------|---------|
| `-p` | Non-interactive single prompt |
| `--output-format json` | Structured output |
| `--permission-mode dontAsk` | No approval prompts (mandatory for CI) |
| `--allowedTools` | Tool whitelist |
| `--max-turns` | Hard cap on agentic loop |
| `--bare` | Raw text only (no JSON overhead) |

---

## GitHub Action

```yaml
uses: anthropics/claude-code-action@v1
with:
  permission-mode: dontAsk
  allowed-tools: "Read,Grep,Bash(git diff)"
  post-inline-comments: true
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

permissions:
  contents: read
  pull-requests: write   # required for inline comments
```

---

## Repeating Reviews

- Always fresh session per review (no anchoring bias).
- CLAUDE.md as the review standard (evolves with the team).
- Pass previous findings as context to avoid re-reporting resolved issues.
- Use explicit criteria — not "review for security" but "check these 4 specific patterns."

---

## Testing Standards

- Point Claude at the existing test suite for style reference.
- CLAUDE.md test runner + fixture locations + standards.
- Check for redundancy before generating.
- `--allowedTools "Read,Grep,Write"` — allows Claude to write the test file directly.
