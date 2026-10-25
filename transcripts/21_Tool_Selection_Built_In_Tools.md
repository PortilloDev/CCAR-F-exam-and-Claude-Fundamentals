# Lecture 21: Tool Selection: Built-In Tools Grep, Glob, Read, Write, Bash

**Section:** Tools and MCP  
**Lecture ID:** 58184805

---

## Transcript

Built-in tools supported natively inside Claude Code and the Agent SDK help avoid overflowing the context window (loading an entire repo into context would break the application).

**Glob** — finds files by path or name using pattern matching (e.g., all `.ts` or `.js` files, files in a specific folder, files with a known name prefix).

**Grep** — searches content *inside* files (not file paths). Use it to locate strings, methods, classes, or error message origins. AI agents use Grep to find the exact spot in code to replace.

**Read** — reads the full contents of a file.

**Write** — replaces the complete file. Use when Edit can't find a unique anchor.

**Edit** — changes a specific portion of a file using a unique anchor text (one or more lines that appear exactly once). If the anchor is not unique, Edit fails → fall back to Write.

**Bash** — runs shell scripts: clone repos, write commit messages, push changes, compile, run scripts. Enables the full code development/deployment pipeline.

---

### Grep vs. Glob

| Grep | Glob |
|------|------|
| Finds content *inside* files | Finds files by *path/name* |
| Locate every call to a function | Find all test files at once |
| Trace where an error message is produced | Scan a directory for `.ts` files |
| Understand which files import a class | Find files with the same extension |

---

### How the Edit anchor works

1. Agent wants to edit `old_string` in `handlers.py`.
2. Grep returns **3 matches** → not unique → Edit cannot be used.
3. Agent reads the full file with Read.
4. Agent rewrites the full file with Write.

**Best practice:** Always start with Grep, then Read only if needed. Reading too many files exhausts the context window.

---

### Dependency-chain pattern (finding all consumers of a changed symbol)

1. Grep for the definition point of the class/function.
2. Grep for files that reference it (imports).
3. Check if those files are wrappers that re-export under a new name.
4. Grep for the wrapper's exported name.
5. Repeat until the full dependency graph is mapped.

**Example:** `config.py` defines `parse_config` → used in `utils.py` (re-exported as `load_settings`) and `legacy.py` (re-exported as `read_conf`). Without the second Grep pass, `load_settings` and `read_conf` consumers would be missed.
