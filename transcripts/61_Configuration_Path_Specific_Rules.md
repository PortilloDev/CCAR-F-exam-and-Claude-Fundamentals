# Lecture 61: Configuration: Path-Specific Rules

**Section:** Claude Code in a Team  
**Lecture ID:** 58217047

---

## Transcript

### Path-specific rules (glob rules)

Rules that only load when Claude is working on files matching a glob pattern. Save context window by not loading irrelevant rules.

```markdown
---
path: "src/**/*.tsx"
---

## React component rules
- Use functional components only.
- Every component must have a corresponding `.test.tsx` file.
```

The rule loads only when Claude opens a `.tsx` file anywhere under `src/`. Other file types are unaffected.

---

### Glob pattern examples

| Pattern | Matches |
|---------|---------|
| `src/**/*.tsx` | All `.tsx` files recursively under `src/` |
| `src/api/**` | Everything under `src/api/` |
| `**/*.test.ts` | All TypeScript test files anywhere |
| `terraform/**/*.tf` | All Terraform files |

---

### When path-specific rules outperform alternatives

**Use path-specific rules when:** The rule applies to all files of a specific type, even across 50 subfolders. One glob covers all of them — no need for per-folder CLAUDE.md files.

**Keyword trigger (exam hint):** "automatically" → think path-specific glob rule.

---

### Limitations of path-specific rules

| Cannot do | Use instead |
|-----------|------------|
| Differentiate by task type (develop vs. debug) | Skills (task-based) |
| Hard enforcement (zero exceptions) | Hooks (`pre_tool_use`) |
| Context window savings with `@` imports | Directory CLAUDE.md or skills |

---

### Summary: which rule mechanism to use

| Mechanism | Loaded | Saves tokens? | Enforced? |
|-----------|--------|-------------|----------|
| Main CLAUDE.md | Always | ❌ | Probabilistic |
| `@` import | Always (at startup) | ❌ | Probabilistic |
| Directory CLAUDE.md | On demand (folder) | ✅ | Probabilistic |
| Glob (path-specific) rule | On demand (file type/path) | ✅ | Probabilistic |
| Skill | On demand (task match) | ✅ | Probabilistic |
| Hook | Always on match | N/A | **Deterministic** |
