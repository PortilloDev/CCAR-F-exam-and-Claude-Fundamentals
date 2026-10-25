# Lecture 17: Intro: Tools That Get Selected

**Section:** Tools and MCP  
**Duration:** ~1 min | **Lecture ID:** 58006577

---

## Transcript

Suppose you give a document assistant a supplier contract and ask which clauses renew automatically. It answers using a vendor's website instead of the uploaded file. To fix this, you need to understand how Claude chose between the available tools. Similar names, incomplete descriptions or wording in the system prompt can each contribute to a wrong choice. We'll look at these causes separately so you can improve the tool interface that caused the problem and give each agent appropriate access. We'll start with writing descriptions that explain a tool's purpose, inputs and outputs and limits. We'll then examine misrouted calls and compare the changes that address each cause. Next, we'll discuss when to split a broad tool into focused tools and when related lookups can share one interface. We'll compare Claude Code's built-in tools for searching file contents, finding files by path, editing text only when the match is unique and replacing a file. Finally, we'll explain how to select tools for each agent's role, including a narrow verification tool for a synthesis agent and the coordinator route for deeper research. After this chapter you'll be able to diagnose tool-selection problems, choose a suitable repair and define the access each agent needs.
