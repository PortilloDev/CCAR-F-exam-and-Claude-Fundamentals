# Lecture 74: Automation: Testing Standards

**Section:** Claude Code in a Team  
**Lecture ID:** 58217111

---

## Transcript

### Supply the existing test suite

When asking Claude to generate tests, always point it at the existing test suite first:

```bash
claude -p "
Generate tests for the new UserService.create() method.
Existing test suite: tests/unit/services/
Test runner: pytest
Fixtures: tests/conftest.py
Coverage standard: happy path + at least 2 error paths per method
"
```

**Why:** Claude matches the style, fixture patterns, and assertion conventions already in use. Generated tests look like the team wrote them.

---

### CLAUDE.md test runner + standards block

```markdown
# Testing Standards

Test runner: pytest
Test location: tests/unit/ (unit), tests/integration/ (integration)
Fixtures: tests/conftest.py
Mocking: use unittest.mock — never mock the database
Coverage: every public method needs a happy path + 2 error paths
Naming convention: test_<method>_<scenario>
```

**Effect:** every headless test-generation run uses these standards automatically — no per-run flags needed.

---

### Redundancy checking

Before generating new tests, have Claude check if the scenario is already covered:

```bash
claude -p "
Check if UserService.create() is already covered in tests/unit/services/test_user_service.py.
If not, generate missing test cases following the standards in CLAUDE.md.
"
```

**Why:** avoids duplicate tests that bloat the suite and slow CI.

---

### Exam pattern: full test generation invocation

```bash
claude -p "
Generate unit tests for src/services/payment_service.py.
- Read existing tests in tests/unit/services/ for style reference.
- Follow testing standards in CLAUDE.md.
- Check for redundancy before generating.
- Output tests as a Python file.
" \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Write"
```

The `Write` tool allows Claude to create the test file directly in the repo.

---

### Key exam distinctions

| Approach | Problem |
|----------|---------|
| "Write tests for this function" | No style reference → inconsistent tests |
| Point at existing suite | Matches conventions |
| Point at CLAUDE.md | Matches standards across all runs |
| Skip redundancy check | Duplicate tests accumulate |
