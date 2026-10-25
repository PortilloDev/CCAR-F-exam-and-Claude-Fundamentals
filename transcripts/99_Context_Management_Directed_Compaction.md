# Lecture 99: Context Management: Directed Compaction

**Section:** Context and Reliability  
**Lecture ID:** 58217279

---

## Transcript

### /compact with custom instructions

```bash
# Default: Claude decides what to keep
/compact

# Directed: you tell Claude what to preserve
/compact Keep all order IDs, amounts, dates, and open action items.
         Drop all tool output from order lookups.
```

**Limitation:** directed compaction is more robust than auto, but still less reliable than a code-controlled case-facts section. Use it when you can't predefine all the fields.

---

### What to tell the compact instruction to keep

- Decisions and the reasoning behind them
- Important identifiers: order numbers, file paths, class names
- References to scanned documents (not full content — just the reference)
- Constraints, policies, discovered rate limits
- Open questions and next steps

### What to tell it to drop

- Tool output that is no longer needed
- Dead-end paths explored and abandoned
- Back-and-forth acknowledgments and clarifications

---

### What compaction leaves untouched

```
Before compaction                 After compaction
─────────────────                 ────────────────
Turn history (large)    →         Summary (small)
Tool results (many)     →         Trimmed
Memory files on disk    →         UNCHANGED (always)
Case facts block        →         UNCHANGED (always)
Session-start hook      →         Re-fires on next session start
```

---

### Session-start hook for state recovery

```bash
# hooks/session_start.sh
cat ./state/progress.json   # inject progress file into context at session start
```

After compaction, the next session start re-fires the hook, injecting the saved progress file into the fresh context automatically.

---

### Verification

Compare the case facts block with the compaction summary. If they differ, the compaction instruction needs refinement.
