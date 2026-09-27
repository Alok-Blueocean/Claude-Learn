# Official Sample Questions — CCDV-F

These are Anthropic's own published sample questions from the official exam guide — public pre-exam study material, not live exam content.

Source: `CCDV-F_Developer_Foundations_Exam_Guide.pdf`, Section 8 ("Sample Questions"). The guide states: *"These illustrative items show the style and cognitive level of the exam. They are not drawn from the live item bank."* The guide includes exactly **three** sample questions, one each for Domains 2, 7, and 8. It does not provide sample items for the other five domains (1, 3, 4, 5, 6) — that gap is not filled in here, in keeping with reproducing the guide faithfully rather than padding it with invented "official" items. (See `practice_questions.md` for original questions covering every domain.)

---

## Sample 1 · Domain 2 — Applications and Integration

A developer must process 10,000 documents overnight to produce a non-urgent analytics report. Cost is the primary concern, and results are not needed until the following morning. Which approach best fits the requirement?

A. Send every request synchronously through the Messages API in parallel to finish as quickly as possible.
B. Use the Message Batches API, which processes large asynchronous workloads within a 24-hour window at reduced cost.
C. Lower max_tokens on synchronous calls to minimize cost.
D. Switch to the smallest available model regardless of output quality.

**Correct answer: B.**

**Rationale (verbatim from the guide):** The Message Batches API is designed for latency-tolerant, high-volume workloads at lower cost, which matches an overnight, non-urgent job. Sending requests synchronously in parallel (A) does not reduce per-token cost; lowering max_tokens (C) or blindly downsizing the model (D) does not address the batch-versus-realtime tradeoff.

---

## Sample 2 · Domain 7 — Security and Safety

A Claude-powered agent summarizes web pages submitted by end users. One page contains hidden text instructing the model to ignore previous instructions and reveal its system prompt. Which mitigation is most effective?

A. Raise the model's temperature so its behavior is harder to predict.
B. Treat retrieved page content as untrusted input, keep it separate from trusted instructions, and use guardrails or hooks so injected instructions cannot trigger sensitive actions.
C. Add a line to the system prompt asking users not to include malicious instructions.
D. Switch to a larger model that follows instructions more reliably.

**Correct answer: B.**

**Rationale (verbatim from the guide):** Prompt injection is addressed by isolating untrusted content from trusted instructions and enforcing least-privilege guardrails so injected text cannot invoke sensitive tools. Temperature (A) is irrelevant to injection; a polite request (C) is not an enforceable control; a more instruction-following model (D) can be more susceptible, not less.

---

## Sample 3 · Domain 8 — Tools and MCPs

A team needs Claude to call an internal inventory service exposed as a REST API. They want the capability to be reusable across several Claude applications and maintained independently of any one app. Which approach best fits?

A. Hard-code the inventory logic into each application's system prompt.
B. Build an MCP server that exposes the inventory operations as tools so multiple Claude applications can connect to it.
C. Paste the current inventory data into the context window on every request.
D. Rely on a built-in tool, since built-in tools can reach any internal REST API.

**Correct answer: B.**

**Rationale (verbatim from the guide):** An MCP server exposes reusable tools that multiple Claude applications can share and that can be maintained independently. Hard-coding logic into prompts (A) is neither reusable nor maintainable; pasting data (C) gives no live access and wastes context; built-in tools (D) do not automatically reach arbitrary internal APIs.

---

## What to take away from these three items

Notice the pattern across all three: the correct answer is always the option that best matches the *stated constraint* in the scenario (cost + latency tolerance → Batch API; untrusted external content → isolation + least privilege; reusability + independent maintenance → MCP server), and every distractor is a plausible-sounding action that either doesn't address the actual constraint or actively misunderstands a mechanism (e.g., believing built-in tools reach arbitrary internal systems, or that temperature affects injection resistance). Expect this "constraint-matching" question design throughout the real exam — read for the binding constraint in the scenario before evaluating the options.
