# CCAR-F Practice Test — Full Exam Simulation

**60 questions · 120 minutes · Pass = 720 / 1000**  
Domains: Agentic Architecture · Claude Code · Prompt Engineering · Tool Design + MCP · Context & Reliability

---

## DOMAIN 1 — AGENTIC ARCHITECTURE (Questions 1–20)

---

**Q1.**  
A support agent built with the Agent SDK is supposed to verify customer identity before processing any refund. During testing, engineers observe that in about 8% of cases the agent processes a refund without calling `get_customer` first. The current setup uses a detailed system prompt and three few-shot examples enforcing the rule. What is the BEST fix?

A) Add more few-shot examples showing the correct sequence.  
B) Increase the model temperature to make the agent more consistent.  
C) Add a `pre_tool_use` hook that denies `process_refund` if no customer ID has been recorded.  
D) Add a `post_tool_use` hook that reverses the refund if no customer ID was set.  

**Answer: C**  
> Hooks are deterministic — a `pre_tool_use` hook reduces error rate to zero. More examples (A) are probabilistic; they reduce error rate but never to zero, which is unacceptable for money. Temperature (B) does not fix sequencing. A `post_tool_use` hook (D) runs AFTER the refund already executed — it cannot undo a completed action.

---

**Q2.**  
An agent coordinator is decomposing a research task into subagent briefs. The coordinator's `allowed_tools` list contains: `["Read", "Grep", "WebSearch"]`. The coordinator attempts to spawn a subagent but it silently never runs. What is the most likely cause?

A) The subagents are not registered in `agents` on `ClaudeAgentOptions`.  
B) The coordinator's `allowed_tools` does not include `"Task"`.  
C) The subagent's `model` field was omitted.  
D) The coordinator's `permission_mode` is set to `dontAsk`.  

**Answer: B**  
> `"Task"` (or `"Agent"`) must be in `allowed_tools` for the coordinator to spawn subagents. Without it, the coordinator simply cannot call the Task tool. A missing `model` field (C) inherits from parent — not a blocker. `dontAsk` (D) would deny non-listed tools but Task needs to be listed; the root cause is still the missing listing.

---

**Q3.**  
A multi-agent research system sends subagent findings to a synthesis agent. The synthesis agent returns a report with some claims that cannot be traced back to any source. What is the most likely architectural mistake?

A) The research subagent used `WebSearch` instead of `WebFetch`.  
B) The coordinator passed the synthesis agent a plain text summary of findings instead of structured records with source metadata.  
C) The synthesis agent's model should be cheaper (Haiku instead of Sonnet).  
D) The coordinator did not set `readOnlyHint=True` on the research tools.  

**Answer: B**  
> When findings are passed as unstructured text, the synthesis agent has no anchor for citations. The fix is to pass structured records (claim, source_name, source_url, date) so the synthesis agent can preserve provenance. `readOnlyHint` (D) is a performance flag, not a provenance control.

---

**Q4.**  
You need subagents that research four independent topics simultaneously to minimize total latency. What is the correct way to trigger parallel execution?

A) Call each subagent in a separate turn, one at a time.  
B) Set `parallel=True` in `ClaudeAgentOptions`.  
C) Instruct the coordinator to "Launch ALL tasks in THIS SINGLE TURN" in its prompt.  
D) Use `asyncio.gather()` in the coordinator's Python code around four `query()` calls.  

**Answer: C**  
> Parallelization is triggered by the coordinator's prompt — multiple `Task` calls in one assistant message run in parallel automatically. There is no `parallel=True` flag (B). Separate turns (A) = sequential. `asyncio.gather()` on `query()` (D) would work at the SDK level but the exam-preferred pattern is coordinator-driven fan-out via prompt.

---

**Q5.**  
A subagent times out while searching for data on music AI market share. It returns an empty object `{}` to the coordinator. What consequence does this have?

A) The coordinator raises an exception and halts the pipeline.  
B) The coordinator correctly identifies the failure and triggers a retry.  
C) The coordinator treats the result as a successful empty search and continues without the data.  
D) The coordinator escalates to a human automatically.  

**Answer: C**  
> Empty result `{}` looks like success with no data. The coordinator has no way to distinguish a timeout from a genuine zero-result search. The subagent must return a structured failure report with `failure_type`, `coverage_impact`, and `alternative_approaches`.

---

**Q6.**  
A synthesis agent occasionally browses the web on its own instead of writing the report. Its tools are `["Write", "Edit", "WebSearch"]`. What is the BEST fix?

A) Add a system prompt rule: "Do not use WebSearch. Write only."  
B) Remove `WebSearch` from the synthesis agent's `tools` list and set `permission_mode="dontAsk"`.  
C) Add a `pre_tool_use` hook that denies `WebSearch` for the synthesis agent.  
D) Add a few-shot example where the synthesis agent correctly avoids WebSearch.  

**Answer: B**  
> The correct fix is to remove the tool from scope entirely. With `dontAsk`, anything not in `allowed_tools` is auto-denied. A system prompt rule (A) is probabilistic — it can be ignored. A hook (C) works but is more complex than simply not giving the tool. Few-shot (D) is probabilistic.

---

**Q7.**  
You want to continue a previous Agent SDK session. Most context is still valid but two files have changed. What is the correct approach?

A) Start a fresh session and summarize the full previous session in the new prompt.  
B) Use `resume="<session-id>"` and explicitly name the changed files in the new prompt.  
C) Fork the session and re-run all prior steps with the new files.  
D) Use `ClaudeSDKClient` to re-send the full chat history from the prior session.  

**Answer: B**  
> When ~80-90% of context is still valid, resume and name the changed files so the agent re-reads them instead of using cached stale data. Start fresh (A) when many things have changed. Fork (C) is for comparing two different approaches from a shared baseline — not for file updates.

---

**Q8.**  
An agent coordinator checks its research results against quality criteria and finds that one topic category is missing. There were no explicit instructions for how many rounds to run. What happens next?

A) The coordinator stops and returns a partial report.  
B) The coordinator escalates to a human because it cannot proceed.  
C) The coordinator triggers targeted follow-up research for the missing category.  
D) The coordinator loops indefinitely until all criteria are met.  

**Answer: C**  
> In a goal-based prompt with quality criteria, the coordinator self-corrects: if criteria are not met, it spawns targeted follow-up subagents. It does not loop indefinitely (D) — a maximum rounds limit (e.g., 5) bounds it. It does not stop prematurely (A) or escalate (B) when it can still self-correct.

---

**Q9.**  
A `post_tool_use` hook receives the result of a `process_refund` call and detects that the refund amount exceeds the policy limit. What can the hook do?

A) Reverse the refund by calling the refund tool again with a negative amount.  
B) Block the refund from completing.  
C) Modify the tool output that Claude will receive.  
D) Both A and B.  

**Answer: C**  
> `post_tool_use` runs AFTER the tool has executed — it cannot block or reverse. It can only modify what Claude sees in the result (`updatedToolOutput`). The gate for blocking must be a `pre_tool_use` hook.

---

**Q10.**  
A coordinator uses a fixed pipeline: scan → extract → validate. The extract step fails for one document type due to a schema mismatch. Which recovery strategy is correct?

A) Retry the extract step with exponential backoff.  
B) Modify the input schema to match the document type, then retry.  
C) Proceed with partial results and report the coverage gap.  
D) Fork the session and retry from the scan step.  

**Answer: B**  
> Schema mismatch = validation error. Fix the input (schema correction), then retry. Exponential backoff (A) is for transient/timeout errors. Proceeding with partial results (C) is the fallback if retries fail, not the first step for a validation error.

---

**Q11.**  
An agent uses `query()` for every user message. After the first turn, the user asks a follow-up question that depends on the first answer. The agent has no context about the first answer. What is the root cause?

A) `query()` does not support `allowed_tools`.  
B) `query()` is stateless — it does not preserve history across calls.  
C) The agent's `permission_mode` was set to `dontAsk`.  
D) The system prompt was not resent.  

**Answer: B**  
> `query()` is a one-shot call with no session state. For multi-turn memory, use `ClaudeSDKClient`, which stores the session on disk and can be resumed.

---

**Q12.**  
A tool's `post_tool_use` hook receives a `status` field with value `404` and a `created` field with value `1718000000` (Unix timestamp). The hook's job is to normalize these. What should it return?

A) `{"status": "404", "created": "1718000000"}`  
B) `{"status": "Not found", "created": "2024-06-10T..."}`  
C) Leave the values as-is and return `{}`.  
D) Deny the tool call.  

**Answer: B**  
> Normalization converts numeric status codes to natural language and Unix timestamps to ISO 8601. LLMs process natural language more reliably, reducing token use and hallucination risk. Returning `{}` (C) means no change — the raw values reach Claude.

---

**Q13.**  
When forking a session, the fork inherits stale context from three days ago. Several key files have been rewritten since then. What is the risk?

A) The fork will crash because session IDs expire.  
B) The fork will use cached file contents that no longer match the codebase.  
C) The fork will automatically detect the changed files and re-read them.  
D) The fork will run correctly because files are always re-read on session start.  

**Answer: B**  
> A session stores tool results including file contents read during the session. If those files changed, the fork inherits stale data. The agent will not detect changes automatically — you must tell it which files changed.

---

**Q14.**  
A customer sends one message with two issues: a billing dispute and a missing shipment. What is the BEST initial step for an agent handling this?

A) Ask the customer which issue they want to address first.  
B) Escalate immediately to a human since there are multiple issues.  
C) Decompose the message into structured issue records, then work each in parallel.  
D) Address only the first issue mentioned and wait for the customer to follow up on the second.  

**Answer: C**  
> The correct pattern is to decompose into structured issue records (issue_id, type, claim, status) and work them in parallel with one subagent per issue. Asking the customer (A) adds unnecessary friction. Escalating (B) too quickly is an anti-pattern. Ignoring the second issue (D) misses the requirement.

---

**Q15.**  
A synthesis agent needs to verify a date claim mid-task. The current design routes all fact-checking through the coordinator, adding 3-4 extra turns. The team is considering giving the synthesis agent a narrow `check_date(query)` tool. What condition justifies this scope expansion?

A) The synthesis agent is the most expensive model so it should have more tools.  
B) The extra capability is narrow and purpose-specific, the coordination overhead is measurable, and the tool cannot be misused for broader tasks.  
C) The coordinator explicitly allows it in the `allowed_tools` list.  
D) The synthesis agent is currently failing more than 10% of tasks.  

**Answer: B**  
> The three conditions for a justified cross-role exception are: narrow + purpose-specific capability, measurable overhead from the alternative, and the tool cannot be misused for broader tasks. Cost (A) is irrelevant. Coordinator's `allowed_tools` (C) controls coordinator tools, not subagent scope.

---

**Q16.**  
Which of these correctly describes the `description` field vs. the `prompt` field in `AgentDefinition`?

A) `description` defines behavior; `prompt` defines routing.  
B) `description` is read by the coordinator to decide which subagent to call; `prompt` defines the subagent's behavior and output format.  
C) Both are read by the subagent at startup.  
D) `description` is optional; `prompt` is required.  

**Answer: B**  
> `description` = coordinator routing decision (who reads: parent agent). `prompt` = subagent system prompt (who reads: subagent itself). Role and behavior instructions must go in `prompt` — putting them in `description` does not improve subagent performance.

---

**Q17.**  
An agent calls `process_refund` with `amount=750`. A `pre_tool_use` hook checks this and returns `permissionDecision: "denied"`. The agent's `allowed_tools` list includes `"process_refund"`. What happens?

A) The refund executes because `allowed_tools` pre-approves it.  
B) The hook denial is ignored because `bypassPermissions` mode overrides hooks.  
C) The tool is blocked — hooks run before permission gates and override them.  
D) The `can_use_tool` hook is triggered next.  

**Answer: C**  
> Hooks run BEFORE permission modes — they override even `allowed_tools` pre-approval and even `bypassPermissions`. A `pre_tool_use` denial always blocks, regardless of other settings.

---

**Q18.**  
You are designing a pipeline to review a PR with 15 files. Which architecture avoids the attention diffusion problem?

A) Send all 15 files in one context window to a single agent for deep analysis.  
B) Use a larger model (Opus instead of Sonnet) for better attention across all files.  
C) One agent per file (parallel), then one integration agent to aggregate findings.  
D) Run the same single agent three times sequentially to catch missed issues.  

**Answer: C**  
> Attention diffusion is caused by too much content in one context — not by model size. The fix is one agent per file (fresh context, full attention), then one integration pass for cross-file issues. A bigger model (B) does not solve attention distribution.

---

**Q19.**  
A coordinator prompt says "Research how AI is used in creative industries. Subagent 1: AI in music. Subagent 2: AI in music production." After aggregation, the synthesis agent finds many duplicate sources. What is the root cause?

A) Both subagents used the same `WebSearch` tool.  
B) The scope partitioning overlapped — "music" and "music production" are not distinct domains.  
C) The coordinator did not set `allowed_tools=["Task"]`.  
D) The synthesis agent lacks a `post_tool_use` normalization hook.  

**Answer: B**  
> Overlapping scope → duplicate sources → messy synthesis. The fix is distinct, non-overlapping domains: "music and audio" vs. "writing and publishing" vs. "film, TV, animation" vs. "visual art."

---

**Q20.**  
A support agent has the tools: `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`. The agent sometimes calls `process_refund` without ever calling `lookup_order` first, resulting in refunds applied to wrong orders. What is the BEST fix?

A) Add a system prompt rule: "Always call lookup_order before process_refund."  
B) Add a sequence gate: a `pre_tool_use` hook that denies `process_refund` until `lookup_order` has been called and an order ID has been recorded.  
C) Remove `process_refund` from the tools list.  
D) Use `tool_choice` to force the model to always call `lookup_order`.  

**Answer: B**  
> A sequence gate is the deterministic solution. A system prompt rule (A) is probabilistic. Removing the tool (C) breaks functionality. `tool_choice` (D) forces a specific tool on the next call — it doesn't enforce a required sequence across multiple turns.

---

## DOMAIN 2 — CLAUDE CODE CONFIGURATION (Questions 21–34)

---

**Q21.**  
A team's CLAUDE.md file has grown to 420 lines. During a complex refactoring task, Claude ignores several rules from the middle of the file. What is the most likely cause and fix?

A) The file has a syntax error; fix the formatting.  
B) CLAUDE.md files have a 400-line hard limit; trim to 400.  
C) A long CLAUDE.md causes context overflow; move task-specific content to skills and zero-exception rules to hooks.  
D) CLAUDE.md rules are case-sensitive; rewrite them in lowercase.  

**Answer: C**  
> A 400+ line CLAUDE.md bloats the system prompt, leading to context overflow where content in the middle gets less attention. Fix: CLAUDE.md = standing conventions; skills = task-specific procedures; hooks = zero-exception rules.

---

**Q22.**  
A CLAUDE.md rule states: "Be careful with database operations." Engineers report Claude still makes risky DB changes. What is the problem?

A) The rule needs to be placed at the top of the file.  
B) The rule uses a vague adjective — it is not specific and testable.  
C) CLAUDE.md does not support database-related rules.  
D) The rule needs a code example attached.  

**Answer: B**  
> Vague adjectives ("be careful") produce non-reproducible results. Effective rules are specific and testable: "Never run DELETE without a WHERE clause. Never run migrations against production without a dry-run flag."

---

**Q23.**  
Which permission mode is MANDATORY when running Claude Code in a CI/CD pipeline?

A) `auto`  
B) `acceptEdits`  
C) `dontAsk`  
D) `bypassPermissions`  

**Answer: C**  
> `dontAsk` is required for CI. In any other mode, Claude can trigger approval prompts that block the pipeline indefinitely. `bypassPermissions` (D) is only for isolated containers — it is dangerous for general use.

---

**Q24.**  
A project has `./CLAUDE.md` at the root and `./src/payments/CLAUDE.md` for the payments module. A rule in the root CLAUDE.md says "Return null for missing records." The payments CLAUDE.md says "Throw PaymentNotFoundException for missing records." Claude is editing `./src/payments/processor.py`. Which rule applies?

A) The root rule always wins.  
B) The more specific (innermost) CLAUDE.md wins — Claude throws PaymentNotFoundException.  
C) Both rules apply simultaneously, causing a conflict.  
D) CLAUDE.md files in subdirectories are ignored.  

**Answer: B**  
> The CLAUDE.md hierarchy gives precedence to the innermost file. The payments-specific rule overrides the root rule for files in that directory.

---

**Q25.**  
A path-specific rule in CLAUDE.md has `globs: ["src/api/**/*.ts"]`. An engineer is editing `src/api/auth/login.ts`. Is this rule loaded?

A) Yes, because the file matches the glob pattern.  
B) No, because the engineer's working directory is not `src/api`.  
C) Only if the engineer explicitly loads the rule with `/skill`.  
D) Yes, but only if the file is open in the IDE.  

**Answer: A**  
> Path rules are loaded based on the FILES being worked on, not the current working directory. The glob `src/api/**/*.ts` matches `login.ts`, so the rule loads.

---

**Q26.**  
A team needs a rule that blocks any `psql` command targeting the production database hostname. The rule must NEVER be skipped. Where should it go?

A) CLAUDE.md at the project root.  
B) A skill document.  
C) A `pre_tool_use` hook (shell script) that checks the command string before execution.  
D) A path-specific CLAUDE.md for the `migrations/` folder.  

**Answer: C**  
> CLAUDE.md rules are probabilistic — they can be overridden or forgotten over many turns. Hooks are deterministic: the shell script fires on every `Bash` tool call matching the pattern. The exam decision tree: "Must NEVER be skipped? → Hook."

---

**Q27.**  
A GitHub Action runs Claude Code to review PRs. The action needs Claude to post inline comments on the diff. Which permission is required?

A) `contents: write`  
B) `pull-requests: write`  
C) `issues: write`  
D) `actions: read`  

**Answer: B**  
> Inline PR comments require `pull-requests: write`. `contents: read` is sufficient for reading files. `actions: read` is for reading workflow metadata.

---

**Q28.**  
A developer asks Claude to refactor a module that spans 45 files with complex interdependencies and multiple valid architecture approaches. Which working mode is most appropriate?

A) Direct execution — small bug fix.  
B) `acceptEdits` mode — let Claude edit and review afterward.  
C) Plan mode — explore and draft a plan before executing.  
D) `bypassPermissions` — large scope needs all tools approved.  

**Answer: C**  
> Plan mode is appropriate for: complex architecture decisions, multiple valid approaches, large file scope (40+ files), unknown codebase. Changing a plan is cheap; undoing 45 files of code is expensive.

---

**Q29.**  
A skill document describes a PR review checklist. When is the skill's BODY loaded into Claude's context?

A) At session start, always.  
B) Only when Claude decides it is relevant to the current task.  
C) Only when the user explicitly invokes it (e.g., `/pr-review`).  
D) Never — skills are metadata only.  

**Answer: C**  
> Skill descriptions are always loaded (for routing). The body is loaded on demand — when the skill is explicitly invoked. This keeps context lean for tasks that don't need the skill.

---

**Q30.**  
A team runs repeated PR reviews with Claude. After several reviews, engineers notice Claude is anchoring on its previous assessments and not flagging things it flagged before. What is the correct fix?

A) Use a single long-running session so Claude remembers all previous reviews.  
B) Pass the previous review's findings as context so Claude avoids re-reporting resolved issues, but always use a fresh session per review.  
C) Increase `max_turns` so Claude has more time per review.  
D) Use `bypassPermissions` so Claude runs faster.  

**Answer: B**  
> Fresh session per review prevents anchoring bias. Pass previous findings to avoid re-reporting already-resolved issues. One long-running session (A) causes anchoring — the opposite of what's needed.

---

**Q31.**  
A headless Claude Code invocation looks like:
```
claude -p "Review auth.py for SQL injection" --output-format json --permission-mode dontAsk --allowedTools "Read,Grep"
```
The review never touches `auth.py` even though it exists. What is the most likely cause?

A) `--permission-mode dontAsk` blocks all tool use.  
B) `Bash` is not in `--allowedTools`, but that's unrelated.  
C) `Read` and `Grep` are in `--allowedTools` so they should work — the issue is elsewhere.  
D) `--allowedTools "Read,Grep"` is correctly scoped; Claude may have used a search and found nothing suspicious.  

**Answer: D**  
> `Read` and `Grep` in `allowedTools` with `dontAsk` means those tools run; everything else is denied. The most likely explanation is the review ran correctly and found nothing — not a misconfiguration. If `Read` were missing, Claude couldn't open the file.

---

**Q32.**  
A CLAUDE.md rule should only apply when working on test files (`tests/**/*.test.ts`) and API source files (`src/api/**/*.ts`). How is this configured?

A) Create two separate CLAUDE.md files in each directory.  
B) Use a frontmatter `globs` field listing both patterns in one rule block.  
C) Set an environment variable `CLAUDE_PATHS` at startup.  
D) Use a hook that checks file paths.  

**Answer: B**  
> Path-specific rules use frontmatter: `globs: ["src/api/**/*.ts", "tests/**/*.test.ts"]`. Multiple patterns can be listed in one rule.

---

**Q33.**  
After two iterations with text feedback, Claude is still generating tests in the wrong structure. What technique should be tried next?

A) Increase `max_tokens` for longer responses.  
B) Switch to a cheaper model.  
C) Provide few-shot input/output examples showing the correct test structure.  
D) Add the test format to CLAUDE.md.  

**Answer: C**  
> The iterative refinement decision: after 2+ text iterations still failing → add few-shot examples. CLAUDE.md (D) handles standing conventions but doesn't demonstrate the exact test shape — examples do.

---

**Q34.**  
`bypassPermissions` mode is enabled in Claude Code. A `pre_tool_use` hook denies a `Bash` command that attempts to access `/etc/passwd`. Does the command execute?

A) Yes — `bypassPermissions` overrides all hooks.  
B) No — hooks run before permission modes and override them, including `bypassPermissions`.  
C) Yes — `bypassPermissions` + `allowed_tools` always wins.  
D) It depends on whether the hook is registered in `settings.json` or `.mcp.json`.  

**Answer: B**  
> Hooks run before permission modes. Even `bypassPermissions` cannot override a `pre_tool_use` hook that returns `denied`. This is a critical safety design.

---

## DOMAIN 3 — PROMPT ENGINEERING & STRUCTURED OUTPUT (Questions 35–46)

---

**Q35.**  
A review pipeline flags 35% of stylistic comments as bugs. The team adds confidence scores: the model now returns 0.9 confidence on stylistic issues it misidentifies as bugs. What does this change accomplish?

A) It makes the pipeline accurate again.  
B) It allows the team to route low-confidence items to human review.  
C) It adds a number to unreliable results without rebuilding trust — confidence scores on noisy results don't fix the noise.  
D) It enables batch processing for high-confidence items.  

**Answer: C**  
> Adding a confidence score to a noisy category does not reduce noise. The fix is: disable the category, fix it offline with few-shot prompts, re-enable once false-positive rate is acceptable.

---

**Q36.**  
A document extraction pipeline sometimes invents values for a required field when the field is absent in the source document. What is the most effective schema fix?

A) Add a retry with exponential backoff.  
B) Add a `strict: true` flag to the schema.  
C) Change the field type to `["string", "null"]` or add `"unclear"` to its enum, and remove it from `required` if truly optional.  
D) Add a few-shot example where the field is populated.  

**Answer: C**  
> When a field is absent, retrying (A) increases hallucination pressure. The schema must allow `null` or an escape value like `"unclear"`. `strict: true` (B) stops the flow entirely on missing fields — it does not prevent hallucination when the field is marked required.

---

**Q37.**  
A team uses a plain prompt: "Extract the invoice data and return it as JSON." Claude occasionally returns prose with embedded JSON, or JSON with extra fields. What is the EXAM-PREFERRED fix?

A) Add "Return ONLY valid JSON, no prose" to the prompt.  
B) Post-process the response with a regex to extract JSON.  
C) Use the forced tool call pattern: define a tool with the exact schema and set `tool_choice` to force it.  
D) Use a higher temperature to make responses more structured.  

**Answer: C**  
> The forced tool call pattern guarantees structured output with no prose. The exam's preferred pattern is always forced tool call over prompt-only JSON extraction. `additionalProperties: false` prevents extra fields.

---

**Q38.**  
A batch API job processes 10,000 invoices. Some invoices need a follow-up tool call based on the first extraction result (e.g., if `status=disputed`, look up the dispute record). Can this multi-turn flow run in the Batch API?

A) Yes — the Batch API supports full multi-turn agentic loops.  
B) No — the Batch API supports a single forced tool call only; multi-turn flows require the synchronous API.  
C) Yes, with `max_turns=2`.  
D) Only if the second tool call is a `WebFetch`.  

**Answer: B**  
> The Batch API supports one forced tool call per request. Multi-turn conversations (where the second step depends on the first result) require the synchronous API.

---

**Q39.**  
A precision review pipeline flags 25% false positives in the "unclear variable names" category. Another category, "null pointer dereference," has 2% false positives. A proposed fix is to lower the overall strictness threshold to reduce total false positives. What is the problem with this approach?

A) Lowering strictness increases latency.  
B) Lowering strictness removes real findings from high-precision categories like null pointer dereference.  
C) Lowering strictness requires re-training the model.  
D) There is no problem — this is the correct approach.  

**Answer: B**  
> Reducing overall strictness hurts ALL categories, including high-precision ones. The correct fix is category-level: disable the noisy category, fix it offline, re-enable independently.

---

**Q40.**  
Few-shot examples are added to a review pipeline. After deployment, false positives drop not just on the examples provided but also on similar boundary cases that were NOT in the examples. What does this indicate?

A) The model is memorizing the examples — overfitting.  
B) The examples are working correctly — the model is generalizing.  
C) The model needs more examples to maintain performance.  
D) The drop in false positives is coincidental.  

**Answer: B**  
> Generalization (improvement on cases NOT in the few-shot set) is the evidence that few-shot examples are working. If performance only improves on the exact examples, that's memorization — not useful.

---

**Q41.**  
Confidence scores should be grounded in labeled data. An engineer proposes using a single confidence score per document. What is the correct approach?

A) One confidence score per document, derived from expert intuition.  
B) One confidence score per field, calibrated against a labeled dataset, with a threshold set from data.  
C) No confidence score — use the model's textual certainty instead.  
D) One confidence score per document type.  

**Answer: B**  
> Correct pattern: per-field scores (not per document), calibrated against labeled data (not expert intuition), threshold derived from data. Items above threshold → automate; below → human review.

---

**Q42.**  
A research pipeline collects market size data: Platform A reports 12% growth; Survey B reports 8% growth. How should the final report handle this conflict?

A) Average the two values: report 10% growth.  
B) Use the more recent source.  
C) Report both values with their sources, note the conflict, and provide a possible explanation without picking a winner.  
D) Use the higher value to be conservative.  

**Answer: C**  
> Never average conflicting values. Never ignore the conflict. Keep both + flag it. The report structure: contested claim → both values → source for each → possible explanation (different methods, different time periods).

---

**Q43.**  
A multi-pass review runs Pass 1 (per-file, parallel) and Pass 2 (integration, serial) in reverse order: Pass 2 first, then Pass 1. What problem does this cause?

A) Pass 2 cannot run without Pass 1's per-file findings — it has no local issues to aggregate.  
B) Pass 1 runs slower in serial.  
C) The model loses context between passes.  
D) There is no problem — order does not matter for multi-pass review.  

**Answer: A**  
> Order is mandatory: Pass 1 → Pass 2. Pass 2 (integration) looks for cross-file errors based on Pass 1 findings. Without Pass 1, Pass 2 has nothing to integrate. Running Pass 2 first means it runs on incomplete data.

---

**Q44.**  
A team wants to use the Batch API to process 50,000 contracts overnight. Before submitting the full batch, what is the recommended workflow?

A) Submit all 50,000 at once to minimize setup time.  
B) Build a representative sample, optimize the prompt synchronously, then submit the full corpus to the Batch API after a clean pass.  
C) Submit in batches of 1,000 and monitor each.  
D) Use the synchronous API for all 50,000 — the Batch API has a 24h SLA risk.  

**Answer: B**  
> Test small → batch corpus. Build a representative sample (one per document type, longest docs, all layouts). Optimize the prompt on the sample using the synchronous API (fast feedback). Only after one clean pass, submit to Batch API.

---

**Q45.**  
A pipeline is failing on a specific document layout. On investigation, the team finds 12 dismissed findings all share the pattern `"flag_pattern": "title_case_variable"`. What is the recommended next step?

A) Remove the "variable naming" category entirely.  
B) Increase the `max_tokens` limit.  
C) Add `detected_pattern` to the schema, group the dismissals by pattern, and build "do not flag" blocks + few-shot examples for that pattern.  
D) Submit the failing documents to human review.  

**Answer: C**  
> Learning from dismissed findings: add `detected_pattern` → group by pattern → build "do not flag" blocks + targeted few-shot examples → watch for generalization on unseen cases of the same pattern.

---

**Q46.**  
A document extraction schema has `additionalProperties: false` and `strict: true`. Claude's extraction returns a field that is not in the schema. What happens?

A) The field is included in the output alongside the schema fields.  
B) The flow stops — `strict: true` means no deviations from schema.  
C) `additionalProperties: false` prevents the extra field from appearing.  
D) Claude ignores `additionalProperties` unless the field is `required`.  

**Answer: C**  
> `additionalProperties: false` prevents Claude from adding fields not defined in the schema. `strict: true` stops the flow if a required field is missing — it does not block extra fields. Both play different roles.

---

## DOMAIN 4 — TOOL DESIGN & MCP (Questions 47–54)

---

**Q47.**  
A tool called `process_data` takes a customer record and returns a summary. After deployment, Claude calls it when asked to "search for orders," "get reports," and "check inventory." What is the most likely cause?

A) The tool has too many parameters.  
B) The tool's description is vague and does not include a "when NOT to use" boundary sentence.  
C) The tool should be split into three tools.  
D) Claude is using the wrong model.  

**Answer: B**  
> The 6-point description checklist includes a "When NOT" clause. Without it, the tool over-triggers on adjacent intents. Fix the description — do not change `tool_choice` to compensate.

---

**Q48.**  
Claude keeps calling `get_customer(id)` when the intent is actually to look up an order. Both `get_customer` and `get_order` take a single `id` parameter with identical descriptions. What is the BEST fix?

A) Force `tool_choice` to route based on user intent.  
B) Rename to `get_customer_by_id` and `get_order_by_id`.  
C) Consolidate into `lookup_entity(type, id)` with a single clear description.  
D) Add a few-shot example of correct routing.  

**Answer: C**  
> When multiple tools have identical input patterns and descriptions, consolidate them. This eliminates the routing ambiguity. Renaming (B) helps but doesn't fix overlapping semantics. `tool_choice` (A) is a workaround — the exam says fix the description, not force the routing.

---

**Q49.**  
A tool returns `{"results": []}` when a database query finds no matching records. Is this correct?

A) No — the tool should return `is_error: true` to signal the absence.  
B) No — the tool should raise an exception.  
C) Yes — empty result is a valid successful response with zero matches; `is_error` should be `false`.  
D) Yes, but it should also include a `"message": "No records found"` field.  

**Answer: C**  
> Empty result ≠ failure. An empty array with `is_error: false` (or omitted) is the correct signal for zero matches. Setting `is_error: true` would cause Claude to treat an empty search as an error and potentially retry unnecessarily.

---

**Q50.**  
An MCP server tool returns a `permission_error`. Which action is correct?

A) Retry the tool call with exponential backoff.  
B) Retry with corrected input.  
C) Escalate to a human — the agent lacks access and cannot retry its way out.  
D) Return an empty result to the coordinator.  

**Answer: C**  
> Permission errors are non-retriable. The agent lacks the access level needed — retrying won't change that. Escalate to a human for access remediation.

---

**Q51.**  
A project-level `.mcp.json` contains a GitHub MCP server configuration with the token hardcoded: `"GITHUB_TOKEN": "ghp_abc123xyz"`. What is the problem?

A) The file format is incorrect.  
B) Hardcoding credentials in a git-committed file exposes them to anyone with repo access.  
C) MCP servers do not support GitHub tokens.  
D) `.mcp.json` does not support the `env` field.  

**Answer: B**  
> Always use environment variable references: `"GITHUB_TOKEN": "$GITHUB_TOKEN"`. Hardcoding in a committed file exposes the credential to all repo contributors and git history.

---

**Q52.**  
An MCP tool returns an error: "Order total exceeds refund policy of 30 days." Should this be retried?

A) Yes — retry with a shorter time window.  
B) Yes — retry with corrected parameters.  
C) No — this is a business error (policy limit hit). Non-retriable. Explain to the user and offer an alternative.  
D) No — this is a permission error. Escalate.  

**Answer: C**  
> Business errors (policy limits, business rules) are non-retriable. The agent cannot retry its way past a policy constraint. The correct response is to explain the situation and offer alternatives (e.g., store credit, escalation).

---

**Q53.**  
A coordinator agent has 12 MCP tools from three different servers. During testing, the coordinator calls tools from all servers even when the task only needs one server's tools. What is the BEST fix?

A) Rename the tools to be more distinct.  
B) Remove unused tools from the session — only load the tools relevant to the coordinator's role.  
C) Add a `pre_tool_use` hook to block the unwanted servers.  
D) Set `tool_choice` to restrict to one server.  

**Answer: B**  
> Trim unused tools — 4-5 tools max per agent is the recommended ceiling. All MCP tools load at session start; removing unused ones reduces confusion, token waste, and incorrect routing. The exam rule: scope down, don't block.

---

**Q54.**  
When does the Glob tool apply vs. the Grep tool in the Agent SDK?

A) Glob = content search inside files; Grep = file name pattern matching.  
B) Glob = file name/path pattern matching; Grep = content search inside files.  
C) Both do the same thing — use either.  
D) Glob is for directories; Grep is for single files.  

**Answer: B**  
> Glob finds files by name/path pattern (e.g., `**/*.ts`). Grep searches content inside files. The pattern: Glob first (find the right files) → Grep (find the right content) → Read only if needed.

---

## DOMAIN 5 — CONTEXT MANAGEMENT & RELIABILITY (Questions 55–60)

---

**Q55.**  
An agent has used 75% of its context window and is about to start implementing a complex feature. What is the correct action?

A) Proceed — 25% remaining is sufficient for most implementations.  
B) Switch to a larger model with a bigger context window.  
C) Run `/compact` with a directed instruction specifying what to keep, then continue.  
D) Start a fresh session immediately.  

**Answer: C**  
> At 70-80% context used and BEFORE starting implementation: run directed compaction to free space. Directed = specify what to preserve (decisions, identifiers, constraints). Don't start implementation with an already-crowded context. Fresh session (D) is for when context is already degraded.

---

**Q56.**  
A directed `/compact` command is run. Which content survives compaction?

A) All turn history and tool results.  
B) Everything in the context window equally.  
C) Memory files on disk and the case-facts block. Turn history and tool results are summarized.  
D) Only the most recent 10 turns.  

**Answer: C**  
> Compaction summarizes turn history and tool results. It leaves untouched: files on disk (memory, scratchpad, case-facts JSON). The case-facts block is designed specifically to survive compaction by being stored outside the rolling context.

---

**Q57.**  
A support agent is handling a multi-turn conversation. The case involves an order ID, a customer ID, refund amounts, and open action items. After 20 turns, these key facts keep being referenced. What is the BEST strategy to prevent them from being lost during compaction?

A) Repeat the key facts in every user message.  
B) Store key facts in a dedicated case-facts block updated by deterministic code; reference it below the system prompt.  
C) Increase `max_tokens` to preserve more context.  
D) Run `/compact` before every turn.  

**Answer: B**  
> The case-facts lever: a dedicated structured block below the system prompt, updated by code (deterministic), designed to survive compaction. It ensures critical identifiers and state always remain accessible.

---

**Q58.**  
An agent encounters a customer request that is partially covered by policy (the standard resolution applies) but the customer is very angry. What should the agent do?

A) Escalate immediately — angry customers always get a human.  
B) Apply the policy resolution — sentiment alone is not an escalation trigger.  
C) Ask the customer if they want a human.  
D) Offer a higher refund than policy allows to defuse the situation.  

**Answer: B**  
> Escalation is policy-based, not sentiment-based. Angry + policy-covered → resolve. Calm + policy gap → escalate. Sentiment analysis as an escalation trigger is an explicit anti-pattern.

---

**Q59.**  
A document extraction pipeline has 97% overall accuracy. A team member proposes sampling 3% of documents at random for quality review. What is the problem with this approach?

A) 3% is too small a sample — use 10%.  
B) Random sampling may under-represent rare document types that have low accuracy; stratified sampling (3% from EACH type) is required.  
C) There is no problem — random sampling is statistically sound.  
D) Sampling should only happen after the pipeline reaches 99% accuracy.  

**Answer: B**  
> A 97% aggregate can hide a 60% accuracy rate on a small document type. Stratified sampling takes ~3% from EACH document type to surface hidden subgroup failures. The exam concept: aggregate metrics can mask per-category failures.

---

**Q60.**  
A codebase discovery subagent is given the full documentation folder (40MB of markdown) plus the source files it needs to analyze. Its context fills up before it can analyze any source files. What is the correct architectural fix?

A) Use a larger model with a bigger context window.  
B) Use subagent isolation — send the subagent only the selected path and source location, not full documentation.  
C) Run `/compact` before launching the subagent.  
D) Split the documentation into two subagents.  

**Answer: B**  
> Subagent isolation: send only what the subagent needs for its specific task — the selected path and source location. Full docs, reasoning chains, and coordinator history should never be dumped into a subagent. A bigger model (A) doesn't solve an architectural data-flow problem.

---

## ANSWER KEY

| Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|
| 1 | C | 16 | B | 31 | D | 46 | C |
| 2 | B | 17 | C | 32 | B | 47 | B |
| 3 | B | 18 | C | 33 | C | 48 | C |
| 4 | C | 19 | B | 34 | B | 49 | C |
| 5 | C | 20 | B | 35 | C | 50 | C |
| 6 | B | 21 | C | 36 | C | 51 | B |
| 7 | B | 22 | B | 37 | C | 52 | C |
| 8 | C | 23 | C | 38 | B | 53 | B |
| 9 | C | 24 | B | 39 | B | 54 | B |
| 10 | B | 25 | A | 40 | B | 55 | C |
| 11 | B | 26 | C | 41 | B | 56 | C |
| 12 | B | 27 | B | 42 | C | 57 | B |
| 13 | B | 28 | C | 43 | A | 58 | B |
| 14 | C | 29 | C | 44 | B | 59 | B |
| 15 | B | 30 | B | 45 | C | 60 | B |

---

## SCORING

| Correct | Score (est.) | Result |
|---------|-------------|--------|
| 60 / 60 | 1000 | Pass |
| 54 / 60 | ~900 | Pass |
| 48 / 60 | ~800 | Pass |
| 43 / 60 | ~720 | Borderline pass |
| < 43 | < 720 | Fail |

**Focus areas if you missed:**
- Q1, Q9, Q17, Q34 → Hook precedence and pre vs. post semantics
- Q2, Q16, Q20 → AgentDefinition and coordinator wiring
- Q5, Q49 → Empty result ≠ error
- Q6, Q36, Q37 → Deterministic fixes vs. probabilistic fixes
- Q23, Q26 → dontAsk for CI; Hooks for zero-exception rules
- Q35, Q41 → Confidence score anti-patterns
