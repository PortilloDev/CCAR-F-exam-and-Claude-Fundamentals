# Lecture 19: Tool Selection: Diagnosing Misrouting

**Section:** Tools and MCP  
**Duration:** ~7 min | **Lecture ID:** 58184801

---

## Transcript

When misrouting occurs despite a "well-defined" description, use this diagnostic decision tree:

**Step 1: Check the system prompt first**
The system prompt has a huge impact on tool selection. Identify if a keyword in the system prompt is over-triggering a specific tool. Example: including the word "account" in a condition causes `get_customer` to be triggered in 78% of cases — removing that word drops it to 93% for the wrong tool. Evaluate each word's impact.

**Step 2: Check for overlapping tool names/descriptions**
- BAD pair: `AnalyzeContent` + `AnalyzeDocument` — too similar, 50% wrong selection
- FIX: rename to `ExtractWebResults` + `AnalyzeInternalDocument`
- Improve descriptions with the 6-point checklist

**Step 3: Expand descriptions if still insufficient**
Longer descriptions cost more tokens (slower, more expensive). Start short, include all 6 checklist points, then expand if error rate is still high.

**Step 4: Consider splitting into dedicated agents**
If multiple overlapping tools must coexist, create separate agents for separate domains — each agent gets only tools relevant to its role.

**Step 5: Few-shot examples (last resort only)**
- Only use when you have highly ambiguous cases where two well-described tools still overlap
- Few-shot increases cost and latency significantly
- Don't jump to few-shot before completing steps 1-4

**Real example — renaming fix:**
- Before: `analyze_content` → "Analyzes content and extracts key information"
- After: `extract_web_results` → "Processes and returns information retrieved from web search and URLs"

**Evaluation is key:** Test your system prompt thoroughly. Understand the impact of each word. Monitor tool selection during runtime.
