# Lecture 12: Intro: The Tool-Use Loop

**Section:** Tools and MCP  
**Duration:** ~1 min | **Lecture ID:** 58006575

---

## Transcript

An assistant on an operations dashboard needs to answer questions about current subscription usage. Claude can interpret the question, but the current number of available seats is stored in your database. You need a way for the model to request that information and for your application to return it. In this chapter, we'll explain how tool use connects those responsibilities. You'll follow a request through the model's decision, the application's execution and the response to the user. We'll start with what happens when Claude requests a tool, and which parts of the process your code must carry out. Then we'll define a tool by giving it a name, a description and an input schema. We'll explain how the description helps Claude select the tool and how the schema defines its arguments. Next, we'll walk through the loop: execute the action, return its result in a user message, match it to the original call by identifier and continue while tools are requested. We'll also compare allowing free tool choice, requiring a tool call and forcing a particular tool. After this chapter you will be able to implement this exchange and use the stop reason to distinguish a tool request from a completed turn.
