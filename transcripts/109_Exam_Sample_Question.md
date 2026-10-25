# Lecture 109: Exam: Sample Question

**Section:** Exam Readiness and Practice Tests  
**Lecture ID:** 58217305

---

## Transcript

### Sample question walkthrough

**Scenario:**  
Your pipeline extracts product lines from supplier delivery notes to a tool whose JSON schema **requires** batch number on every line. Some suppliers leave the batch number blank, and all those lines came back with a batch number anyway — well-formed and matching nothing the supplier shipped. Every response passes schema validation. Quality assurance can chase a missing batch number by hand, but the wrong one hides stock from recall.

**What is the most effective change?**

A. Add a line to the system prompt telling the model never to invent a batch number.  
B. Make the batch number nullable in the schema and describe null as the answer when a line has none.  
C. Check each batch number against the supplier's record and retry with error when it fails.  
D. Route delivery notes to a larger model and enable extended thinking for the extraction call.

---

### Analysis framework (apply to every exam question)

| Component | This question |
|-----------|--------------|
| **Setting** | Delivery note extraction via a tool with a required batch_number field |
| **Symptom** | Lines without a batch number came back with one (invented) |
| **Constraint** | Missing batch number = acceptable (QA can chase it). Wrong batch number = critical (hides recall stock) |
| **Ask** | Most effective single change |

---

### Why each option fails or works

| Option | Verdict | Reason |
|--------|---------|--------|
| A — system prompt instruction | ❌ Distractor | Probabilistic; prompt instructions can be overridden; schema still forces a value |
| **B — nullable schema** | ✅ **Correct** | Removes the pressure to invent; Claude returns null when no batch number exists; schema is honest |
| C — retry with supplier check | ❌ Wrong cause | Addresses verification after the fact, not the root cause (schema forces invention) |
| D — larger model + extended thinking | ❌ Wrong cause | Model capability is not the issue; schema design is |

---

### Exam reasoning rule

**Identify the constraint first.** Here: "wrong one hides stock from recall" → invented values are the critical risk. The schema forces a value → schema design is the root cause → fix the schema (option B).
