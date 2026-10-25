# Lecture 91: Scale & Review Architecture: Batch vs. Synchronous

**Section:** Prompt Engineering and Structured Output  
**Lecture ID:** 58217255

---

## Transcript

### Decision: batch or synchronous?

**Use batch when:**
- Volume: tens of thousands of documents
- Latency-tolerant: overnight security scans, weekly technical debt reports
- Results not needed urgently
- Recurring jobs with relaxed SLAs

**Use synchronous when:**
- PR pre-merge check (developer waiting)
- Real-time feedback during development
- User waiting in a live process queue

---

### Time budget for batch jobs

```
Preparation (first run may take ~4h)
+ Batch processing (up to 24h)
+ Downstream processing (your pipeline consuming results)
= Total time budget
```

If your SLA is 30 hours: subtract 24h for the batch → only 6h remain for everything else. Very tight.

**Hard limit:** 24 hours is a promise (max), not a guarantee of speed. Some items may fail and require resubmission → another 24h window.

---

### Summary table

| Dimension | Batch API | Synchronous |
|-----------|-----------|-------------|
| Cost | Up to 50% cheaper | Full price |
| Latency | Up to 24h | Seconds |
| Tool calls | Single forced tool only | Full multi-turn loop |
| Best for | Large volume, flexible SLA | Interactive, real-time, immediate |
| Order | Non-deterministic | Sequential |
