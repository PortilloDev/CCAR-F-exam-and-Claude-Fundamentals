# Lecture 69: Working Modes: Iterative Refinement

**Section:** Claude Code in a Team  
**Lecture ID:** 58217085

---

## Transcript

### When to switch from text instructions to examples

After 2+ iterations with text refinement, if the output is still wrong, switch to **input-output examples** (few-shot):

```markdown
Normalize these timestamps:

Input → Output
"2024-03-21T15:30:00Z" → "March 21, 2024 at 3:30 PM UTC"
"2024-03-21 15:30:00"  → "March 21, 2024 at 3:30 PM UTC"
null                    → "" (empty string)
```

Show the exact transformation including edge cases. This is more precise than any amount of textual description.

---

### Test-driven iteration (TDD pattern with Claude)

```bash
# 1. Clear existing state
git stash -u && /clear

# 2. Ask Claude to propose test cases (happy + error paths)
# "Propose test cases for the file converter including edge cases
#  like missing permissions"

# 3. Select the most important ones and have Claude write the tests

# 4. Have Claude write the implementation code

# 5. Run the test suite — iterate until all tests pass
```

**Key rule:** Write tests FIRST. Code second. Tests guide implementation through their failures.

---

### Interview mode (requirements discovery)

Ask Claude to interview you before implementing:

```
Before writing the caching layer, interview me about the requirements.
```

Claude asks questions like:
- What's the TTL (time-to-live)?
- Should caching be per-user or global?
- Is stale data acceptable during cache outage?
- What data volume is being cached?

**Use interview mode when:**
- You're unfamiliar with the domain (don't know which requirements matter).
- One problem has multiple solution paths (interview identifies the right path).
- Non-obvious implications exist (e.g., cache invalidation strategy).

---

### Batching vs. sequencing feedback

| Situation | Approach |
|-----------|---------|
| Issues interact (fixing timestamps affects schema) | One message with all related fixes |
| Issues are independent | One message per fix — verify each independently |
| Very independent + you want clean history | Fork session — fix each in its own fork |
