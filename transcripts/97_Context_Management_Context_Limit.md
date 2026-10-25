# Lecture 97: Context Management: Context Limit

**Section:** Context and Reliability  
**Lecture ID:** 58217269

---

## Transcript

### Four ground truths

| Fact | Detail |
|------|--------|
| Context window is always limited | Most Anthropic models: ~1M tokens; Haiku: 200K tokens |
| Messages API is stateless | No memory across calls — everything must be re-sent each request |
| Loss in the middle | Focus is highest at start and end; middle content can be missed |
| More context = higher cost | All prior tokens count as input tokens each request |

**Prompt caching** reduces cost and latency but does not eliminate the fundamental limit.

---

### What's inside a context window (per API request)

```
System prompt + tool definitions + schemas
+ Attached documents / tool results
+ Conversation history so far
+ Reserved space for model reply
= Total context window
```

If the total exceeds the context limit → overflow; the model cannot reply.

---

### The summarization trap

Example — customer support conversation:
- **Original:** discount amount, double-charge amount, return window, refund-to-card preference.
- **After auto-compaction:** "promotional pricing discussed, customer disputed charge, return timing mentioned, customer frustrated."

**Problem:** the summary is true but loses the hard facts. The agent can no longer issue the refund without re-interviewing the customer or re-running tool calls.

---

### Key principle

Always keep **exact identifiers and amounts** outside the compaction path. Summaries are for prose context; hard facts need a separate, protected section.
