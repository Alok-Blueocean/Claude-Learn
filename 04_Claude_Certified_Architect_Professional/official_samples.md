# CCAR-P: Official Sample Questions

Anthropic's own published sample questions — public pre-exam study material, not live exam content. Reproduced verbatim from Section 8 of `CCAR-P_Architect_Professional_Exam_Guide.pdf` (Version 1.0), including the guide's own answer key and rationale from the same section.

The guide itself states: "These illustrative items show the style and cognitive level of the exam. They are not drawn from the live item bank." There are exactly **three** official sample questions in the guide — this is a genuinely sparse set (fewer than half the official samples CCAR-F provides, if you are comparing across the two exams), so treat these as a style/difficulty calibration, not as broad content coverage. Use `practice_questions.md` and `scenario_qa.md` in this kit for volume.

---

## Sample 1 · Domain 3 — Integration

> A team exposes a customer-support agent that can read tickets, draft replies, issue refunds, and delete user accounts. Support staff only ever need to read tickets and draft replies. Applying least-privilege principles, which change best reduces risk?

**A.** Add logging to the refund and delete tools so misuse can be audited later.
**B.** Remove the refund and delete tools from the agent's configuration entirely.
**C.** Keep all tools but add a confirmation prompt before refunds and deletions.
**D.** Replace the agent with a larger model that follows instructions more reliably.

**Correct answer: B**

**Guide's rationale:** "Least privilege means removing capabilities the role does not require, eliminating the attack surface rather than monitoring or guarding it. Logging (A) and confirmations (C) are detective/compensating controls, not removal of unnecessary privilege; model size (D) is unrelated to authorization scope."

---

## Sample 2 · Domain 2 — Models, Prompting & Context

> An application sends the same 8,000-token system prompt and policy document on every request, followed by a short, varying user message. Latency and cost are both concerns. Which optimization most directly addresses both?

**A.** Truncate the policy document to the first 1,000 tokens.
**B.** Switch to the smallest available model regardless of task fit.
**C.** Place the static system prompt and policy before the dynamic content and enable prompt caching.
**D.** Move the policy document into a few-shot example block.

**Correct answer: C**

**Guide's rationale:** "Ordering stable content first and enabling prompt caching lets repeated prefixes be reused, reducing both time-to-first-token and per-request cost without discarding required context. Truncation (A) loses needed policy; downsizing blindly (B) risks quality; relocating to few-shot (D) does not create a cacheable, reusable prefix."

---

## Sample 3 · Domain 4 — Evaluation & Optimization

> A RAG system suddenly returns confident but incorrect answers after a document refresh, while latency and model version are unchanged. What is the most likely first place to investigate?

**A.** The model weights have silently changed.
**B.** The retrieval/indexing step is returning irrelevant or stale chunks.
**C.** The temperature setting is too low.
**D.** The context window has shrunk.

**Correct answer: B**

**Guide's rationale:** "Confident-but-wrong answers following a document refresh, with model and latency unchanged, point to retrieval feeding the model poor context, for example a broken re-index or mismatched embeddings. The other options would not be triggered specifically by a document refresh."

---

## Pattern to notice across all three

Every one of these three official samples has the same anatomy: a short factual situation that pins down what changed and what didn't (staff's actual needs; latency and cost both named as concerns; model version and latency explicitly unchanged), followed by four options where the wrong three are each individually plausible engineering actions — just not the one that matches the specific constraint stated in the stem. None of the wrong options are careless or absurd; they're each a real technique that is simply not the right fix for *this* stated situation. This is the calibration signal the guide is giving you about the live exam's difficulty: read the stem for what is explicitly fixed/unchanged and what is explicitly a concern, and pick the option that addresses exactly that, not the option that sounds like generally good practice in the abstract.
