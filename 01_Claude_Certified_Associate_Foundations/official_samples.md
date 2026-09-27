# Official Sample Questions — CCAO-F

> These are Anthropic's own published sample questions from the official exam guide — public
> pre-exam study material, not live exam content. The guide states these items "show the style and
> cognitive level of the exam" and are "not drawn from the live item bank." Reproduced verbatim
> (question text, options, correct answer, and rationale) from Section 8 of the *Claude Certified
> Associate – Foundations Exam Guide*, Version 1.0.

The guide publishes exactly three sample questions, one each for Domains 2, 3, and 6. No sample
items are published for Domains 1, 4, 5, or 7 — the guide does not claim to cover every domain with a
sample, and this file does not pad that gap with invented "official" items. (See `practice_questions.md`
for original practice items covering all seven domains.)

---

### Sample 1 · Domain 2 — Output Evaluation and Validation

An associate asks Claude to summarize a new regulation, and Claude produces a confident summary
citing a specific subsection number. Before sending the summary to the compliance team, what is the
most appropriate action?

A. Send it as-is, since Claude expressed high confidence.
B. Verify the cited subsection against the official regulation text before sharing.
C. Ask Claude to rate its own confidence and send it if the rating is high.
D. Reword the summary to sound more formal, then send it.

---

### Sample 2 · Domain 3 — Product and Model Selection

An associate needs to generate a high volume of short customer-reply drafts where speed and cost
matter more than deep reasoning. Which choice best fits the task?

A. Use the most capable, highest-cost model for every reply to maximize quality.
B. Use a faster, lower-cost model suited to straightforward, high-volume tasks.
C. Disable all product features to reduce cost.
D. Switch to a different AI platform.

---

### Sample 3 · Domain 6 — Governance, Risk, and Responsible Use

A project manager wants to upload a spreadsheet containing customer names and account numbers so
Claude can analyze trends. Organizational policy restricts sharing regulated personal data. What is the
most appropriate action?

A. Upload the file as-is, since the analysis is internal.
B. Remove or anonymize the personal identifiers before uploading, consistent with policy.
C. Upload the file but instruct Claude not to retain it.
D. Skip the analysis entirely.

---

## Answer Key and Rationale (as published)

**Sample 1: B.** Language models can fabricate specific-looking details such as citation numbers, a
hallucination. Validating factual claims, especially citations bound for a compliance audience, against
an authoritative source is the diligence step required. Self-reported confidence (A, C) is not a reliable
accuracy signal, and reformatting (D) does not address correctness.

**Sample 2: B.** Aligning model selection with task requirements means matching a faster, lower-cost
model to straightforward, high-volume work, reserving the most capable model for complex reasoning.
Always using the top model (A) wastes the cost and latency budget; disabling features (C) or switching
platforms (D) does not address the trade-off.

**Sample 3: B.** Applying data-sensitivity and privacy safeguards means redacting or anonymizing
regulated identifiers before use, so the analysis can proceed without exposing protected data.
Uploading as-is (A) violates policy; instructing the model not to retain data (C) does not satisfy the
policy control; abandoning the task (D) is unnecessary when anonymization enables it.
