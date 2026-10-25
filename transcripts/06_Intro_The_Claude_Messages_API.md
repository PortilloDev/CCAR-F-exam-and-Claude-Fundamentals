# Lecture 6: Intro: The Claude Messages API

**Section:** Introduction to the CCAR-F exam and Claude Fundamentals  
**Duration:** 1 min | **Lecture ID:** 58006571

---

## Transcript

When you build an assistant with the Claude API, your application has to manage the conversation. Suppose a follow-up request no longer follows the writing rules you supplied earlier. Before changing the prompt, you need to check whether those instructions were included in the new request. We'll use this example to explain what Claude receives on each call and what comes back in the response. Understanding that exchange will help you diagnose missing context and incomplete answers. We'll start with the request and the conversation history it needs. We'll explain why the API does not automatically retain earlier calls, and how your application supplies the context needed for a follow-up. We'll include both previous messages and any instructions that still apply. Then we'll examine the response format, including text and tool-use blocks, and explain what the stop reason tells your code to do next. We'll distinguish a response that is finished from one requesting a tool or reaching its output limit. Finally, we'll look at setting that output limit and recognising a truncated answer. After this chapter, you'll be able to construct a follow-up request, inspect the response and decide whether the application can use the result or needs another step.
