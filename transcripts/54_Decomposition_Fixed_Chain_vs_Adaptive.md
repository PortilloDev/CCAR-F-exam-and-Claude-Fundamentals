# Lecture 54: Decomposition & Sessions: Fixed Chain or Adaptive Plan

**Section:** Agentic Architecture  
**Lecture ID:** 58217033

---

## Transcript

### Prompt chaining (fixed pipeline)

**When:** You know all the steps and their sequence before execution starts.

```python
# Fixed pipeline: output of each step flows into the next
metadata = scan_documents(docs)          # step 1
dataset  = extract_data(metadata, docs)  # step 2 — depends on step 1
report   = validate(dataset)             # step 3 — depends on step 2
```

**Each step = one agent call.** Steps can be parallelized when independent.

**Example use cases:**
- Summarize every file in a repo (known steps: scan, summarize, collect).
- PR review: one agent per file (independent), then one final integration step.

**PR review pattern:**
```python
# Parallel scan — one fresh context per file
results = await asyncio.gather(*[
    run_pass(prompt=file_review_prompt, file=f) for f in pr_files
])
# One integration step
final_report = await integrate(local=results)
```
Benefits: full attention per file, reduced hallucination cross-contamination.

---

### Adaptive plan (dynamic decomposition)

**When:** You don't know the steps in advance — discoveries during execution change the plan.

```
Agent creates plan → executes sub-steps → reviews results → revises plan → continues
```

**Example:** Refactoring an unknown legacy codebase.
- Discovery: 3 modules have no tests, 2 are partially covered. (Unknown until scanned.)
- New finding mid-task: payments module depends on an external API → mock tests needed.
- These sub-steps could not have been known before execution.

**In Claude Code:** Use plan mode to let Claude draft a plan before executing it.

---

### Decision matrix

| Question | Answer | Choose |
|----------|--------|--------|
| Do you know all the steps before running? | Yes | Fixed chain |
| Will discoveries change the plan? | Yes | Adaptive plan |
| Steps are independent of each other? | Yes | Parallelize within chain |
| One step's output feeds the next? | Yes | Sequential chain |

---

### Model attention problem (why chaining beats one giant pass)

Passing 14 files into a single context causes attention diffusion:
- Some files get deep feedback, others get shallow.
- The same bug pattern gets flagged in one file, missed in another.
- Not a model size issue — it's an attention distribution issue.

**Fix:** One agent pass per file, fresh context each time, then one integration pass. Not a bigger model.
