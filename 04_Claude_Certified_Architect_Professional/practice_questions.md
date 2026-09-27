# CCAR-P: Original Practice Questions

30 original, professional/advanced-difficulty practice questions, allocated proportionally to the exam blueprint (Domain 1: 5, Domain 2: 4, Domain 3: 6, Domain 4: 5, Domain 5: 4, Domain 6: 4, Domain 7: 2 — approximating 17/13/19/16/14/14/7% of a 30-item set). These are entirely original items written for this study kit — they are **not** from Anthropic's exam guide and are not live exam content. Each item states how many responses to select, matching the real exam's item format. Answer key and rationale are in Section 2, after all questions, so you can self-test before checking.

---

## Section 1: Questions

### Domain 1 — Solution Design & Architecture

**Q1.** A platform team is designing a system where the number and order of steps needed to resolve a request cannot be known in advance — some requests need one lookup, others need five lookups plus two corrective actions, depending on what's found. The team currently has a fixed five-step workflow graph that handles the common case but silently produces incomplete results on the harder ~15% of requests. Select the ONE change that most directly fixes this mismatch.

A. Add two more fixed steps to the workflow graph to cover more cases.
B. Replace the fixed workflow with an agentic pattern where the model plans and executes steps dynamically until the task is resolved.
C. Increase the model size used at each fixed step.
D. Run the same five-step workflow twice and merge the outputs.

**Q2.** A supervisor agent dispatches subtasks to four specialized worker agents, then synthesizes their outputs. Measured end-to-end latency is 9 seconds at p50, and the business SLA is 3 seconds. Task accuracy is excellent. Which TWO changes are most appropriate first responses? (Select 2.)

A. Replace the multi-agent design with a single well-prompted call for the subset of requests that don't actually need specialized decomposition.
B. Add a fifth worker agent to further specialize the task and improve accuracy further.
C. Parallelize the independent worker calls instead of running them sequentially, where their outputs don't depend on each other.
D. Increase the model size used by the supervisor agent to improve synthesis quality.

**Q3.** A team is deciding how to align a proposed Claude-based contract-review system with business value pillars before requesting budget approval. The system will let a 10-person legal team handle a caseload previously requiring 25 people, at slightly lower per-contract turnaround time. Which pillar is the PRIMARY value driver being described?

A. Transformation — a new capability that wasn't previously possible.
B. Efficiency — the same work is now done with materially less human effort.
C. Performance SLA — a contracted latency commitment.
D. Developer productivity — engineers are shipping code faster.

**Q4.** A candidate architecture diagram shows: request → Claude call → response. There is no logging of intermediate steps, no route to human review, and no mechanism to feed production outcomes back into future prompt or retrieval changes. Which domain-1 principle is most clearly violated?

A. Business value pillar alignment.
B. The requirement to design the full loop, including feedback loops, not just the model call.
C. Multi-agent orchestration best practice.
D. Decomposition by risk tier.

**Q5.** A team building a single enormous system prompt for a customer-service assistant has nested dozens of conditional instructions ("if the customer asks about billing AND is a premium tier AND ... then ... unless ... in which case instead ...") to handle every case in one prompt. Output quality has been degrading as more conditions are added. What is the most appropriate architectural fix?

A. Add a chain-of-thought instruction so the model reasons through the nested conditions before answering.
B. Introduce a router that classifies intent first and dispatches to a smaller, focused prompt per branch.
C. Switch to a larger model that can hold more instructions reliably.
D. Move the conditional logic into few-shot examples instead of instructions.

---

### Domain 2 — Claude Models, Prompting & Context Engineering

**Q6.** A production system handles 500,000 requests/day. Roughly 85% are simple, well-scoped classification requests; the remaining 15% require multi-step reasoning over ambiguous input. The team currently sends all requests to the same large, high-capability model. Which change most directly reduces cost without a proportional accuracy loss?

A. Reduce the context window for all requests uniformly.
B. Introduce a cheap router/classifier model that identifies the simple 85% and routes only the ambiguous 15% to the large model.
C. Apply prompt caching to every request regardless of content.
D. Switch every request to the smallest available model.

**Q7.** An architect is separating a system prompt into a "stable" block (role, policy, output schema) and a "variable" block (per-request user data), and placing the stable block first. Besides enabling prompt caching, what is a SECOND direct benefit of this separation? (Select 1.)

A. It removes the need for any output-format guardrail.
B. It makes it possible to update the shared policy/role definition once, in one place, without touching per-request logic — supporting modular prompt reuse across many call sites.
C. It guarantees the model will never hallucinate.
D. It reduces the model's context window size.

**Q8.** A team observes that few-shot examples added to fix inconsistent output formatting have not resolved a separate issue: the system occasionally fabricates numeric figures not present in any retrieved document. What should the team conclude and do?

A. The few-shot examples need to be longer; add more of them.
B. This is a distinct failure mode (hallucination/grounding), not a format issue; add retrieval grounding or explicit instructions to refuse when data isn't present, rather than adding more examples.
C. Chain-of-thought prompting is the correct fix for both issues, so add it instead.
D. This means the model has been deprecated and a newer model version must be adopted.

**Q9.** Which TWO of the following are examples of "prompt reuse strategies" as distinguished from one-off prompt engineering? (Select 2.)

A. Writing a longer, more detailed one-time prompt for a single ad hoc request.
B. Packaging a repeatable procedure and its associated resources as a Skill that can be invoked across many tasks.
C. Composing a system prompt from modular blocks (shared policy + per-tenant customization + per-task instruction) instead of maintaining near-duplicate full prompts per tenant.
D. Increasing the temperature setting for more creative single-request output.

---

### Domain 3 — Integration

**Q10.** An internal agent is configured with tools to read employee records, update employee records, issue payroll adjustments, and terminate employee accounts. The agent's actual task is limited to answering employee questions about their own leave balance. Applying least privilege, which TWO actions are most appropriate? (Select 2.)

A. Remove the update-records, payroll-adjustment, and account-termination tools entirely, since none are needed for the stated task.
B. Grant the agent a scoped, read-only credential limited to leave-balance data rather than full employee-record access.
C. Keep all four tools but add a warning message shown to the requesting employee.
D. Upgrade to a larger model so it is less likely to misuse the extra tools.

**Q11.** A team is choosing between MCP, direct API/CLI integration, and agent-to-agent communication to connect a new agent to an internal expense-reporting service that is the only external system this agent will ever need to call, with no plans to add more tools. Which mechanism best fits, and why?

A. MCP, because it is always the most secure option regardless of scale.
B. Direct API/CLI integration, because it is a small, fixed, well-known single service where dynamic tool discovery adds unneeded complexity.
C. Agent-to-agent communication, because the expense-reporting service should be modeled as an autonomous peer agent.
D. All three are equivalent; the choice makes no architectural difference.

**Q12.** A RAG system indexes a mix of long policy documents, a searchable table of product SKUs with exact part numbers, and frequently-updated pricing data. Which combination of retrieval design choices is MOST appropriate? (Select 2.)

A. Use semantic/vector search only, since it is sufficient for all data shapes including exact part-number lookup.
B. Include keyword/exact-match search alongside semantic search specifically to reliably retrieve exact part numbers that embeddings may not match precisely.
C. Define a re-indexing freshness SLA for the pricing data so stale prices don't get served as current.
D. Store the pricing data as embedded vector chunks identical in treatment to the long policy documents, since a single pipeline is simpler.

**Q13.** An agent's available tool/resource set has grown from 6 tools to over 60 as the platform has expanded. The team currently loads every tool's full definition into context on every call. Tool-selection accuracy has degraded and per-request token cost has risen substantially. What is the most appropriate architectural response?

A. Continue with monolithic context loading, since it was originally simple to reason about.
B. Move to progressive discovery: expose a lightweight index of available tools/Skills, and load full detail only for the specific tool selected for the current step.
C. Remove all guardrails to speed up tool selection.
D. Switch to a larger context window model so all 60 tool definitions fit more comfortably, with no other change.

**Q14.** A security review of an agent's configuration finds that it uses one shared, long-lived API key with broad admin permissions across every tool it calls, including tools used by other unrelated agents. Which observation best identifies the security gap, and what is the most direct fix?

A. The gap is model selection; fix it by using a more capable model.
B. The gap is a shared, over-broad credential creating a large blast radius if leaked; fix it with per-tool, scoped, and ideally short-lived credentials specific to each tool's actual needs.
C. The gap is prompt engineering; fix it by adding a refusal instruction to the system prompt.
D. There is no gap as long as logging is enabled on the shared key.

**Q15.** Which observability practice is MOST important to add first when an agentic system moves from a single model call per request to a multi-hop pipeline (retrieval → model call → tool call → model call)?

A. Average latency across all requests, computed weekly.
B. A propagated trace ID threaded through every hop, enabling reconstruction of what happened across the whole chain for any single failed request.
C. Full unredacted payload logging of every request and response by default.
D. Removing all logging to reduce storage cost.

---

### Domain 4 — Evaluation, Testing & Optimization

**Q16.** A team's evaluation suite consists only of a fixed 200-example golden set that is re-run after every prompt change. Accuracy on the golden set has stayed at 96% for months, but user complaints about unsafe/off-policy outputs on unusual inputs have been rising. What is missing from the evaluation approach?

A. Nothing; a stable golden-set score means the system is performing well.
B. A safety/security-focused evaluation using adversarial or red-team-generated test cases, which a static accuracy-only golden set does not exercise.
C. A larger golden set with more examples of the same kind.
D. Switching the golden set to LLM-as-judge scoring instead of exact match.

**Q17.** A team wants to validate a new system prompt before full rollout. Which sequence best reflects sound A/B testing practice? (Select 1.)

A. Change the prompt, model, and retrieval configuration together, deploy to 100% of traffic immediately, and monitor for complaints.
B. Change only the prompt, route a small percentage of live traffic to it as a canary, compare the full metric set (accuracy, latency, cost, safety) against the incumbent over a statistically meaningful sample, then ramp gradually.
C. Test only against the golden set, skip canary testing, and roll out fully if the golden-set score improves.
D. Change only the prompt but ramp to 100% traffic within the first hour regardless of canary results.

**Q18.** A production RAG system begins producing confident, well-formatted, but factually wrong answers. The model version, temperature setting, and observed latency are all unchanged; the only recent change was a scheduled refresh of the source document corpus. Following sound diagnostic practice, what should be investigated FIRST?

A. Whether the model's weights changed silently.
B. Whether the retrieval/indexing pipeline is returning stale or irrelevant chunks after the refresh (e.g., a broken re-index job or embedding/query mismatch).
C. Whether the temperature setting needs to be raised.
D. Whether the context window has become too small.

**Q19.** Which of the following is the LEAST appropriate sole evaluation methodology for a high-stakes medical-information system being prepared for production, and why?

A. Relying entirely on LLM-as-judge scoring with no human validation of the judge's own reliability, given the high stakes and the risk of unvalidated judge bias going undetected.
B. Combining a golden set, LLM-as-judge for scale, periodic human review to validate the judge, and adversarial test generation for safety.
C. Including a safety-focused evaluation slice specifically targeting harmful or out-of-scope medical claims.
D. Using held-out human-labeled examples as a baseline regression check.

**Q20.** A cost-optimization initiative swaps a large model for a smaller one on a customer-facing summarization feature, cutting per-request cost by 60%, and the team ships it after confirming the golden-set accuracy score is nearly unchanged. Two weeks later, hallucination-related complaints have risen sharply. What does this most likely indicate about the optimization's evaluation process?

A. The optimization was flawed from the start and should never have been attempted.
B. The golden set alone did not adequately measure hallucination/safety behavior under the new smaller model, illustrating why the full metric set (including safety), not just accuracy, must be re-evaluated after any cost optimization.
C. Hallucination complaints are unrelated to model size and must have a different cause entirely.
D. The fix is to revert to the large model permanently with no further investigation.

---

### Domain 5 — Governance, Safety & Risk Management

**Q21.** A healthcare organization is designing a Claude-based system that will process protected health information (PHI) and route insights to clinicians. Which combination of considerations is MOST directly required by HIPAA-relevant architecture, as distinct from general security best practice? (Select 2.)

A. A Business Associate Agreement covering any vendor/model provider in the data path that touches PHI.
B. Minimum-necessary access enforced through tool/agent permission scoping, not just general encryption.
C. Choosing the single largest available model for maximum accuracy.
D. Using chain-of-thought prompting for every clinical query.

**Q22.** A system automates approval of small reimbursement requests below $500 and routes only low-confidence or high-value requests above $500 to a human reviewer. What HITL pattern does this best describe, and what is its main advantage over full human review of every request?

A. Pre-execution approval gate for every request; its advantage is maximum safety with no scaling trade-off.
B. Confidence/threshold-triggered escalation; its advantage is that it scales by automating the confident majority while still routing the risky or uncertain tail to a human.
C. Post-hoc sampled review; its advantage is it requires no real-time model behavior at all.
D. Full human-in-the-loop; its advantage is zero automation risk with no scaling cost.

**Q23.** A hiring-screening system built on Claude produces noticeably different callback recommendation rates across demographic subgroups on otherwise comparable resumes. Which governance response is most appropriate?

A. Ignore the disparity if overall accuracy against a labeled set remains high.
B. Add bias-focused evaluation slices disaggregated by relevant subgroup, and require human review for this consequential decision category given legal/civil-rights implications.
C. Switch to a larger model, since bigger models are inherently less biased.
D. Remove all evaluation of the system's demographic performance to avoid legal exposure.

**Q24.** An architecture review finds that a customer-facing chatbot's guardrails consist solely of a system-prompt instruction telling the model not to discuss competitor products or make unauthorized commitments. No output validation, no independent classifier, and no tool-permission scoping exist. What is the most accurate assessment?

A. This is sufficient because system-prompt instructions are the strongest layer of guardrail available.
B. This relies on a single, comparatively weak guardrail layer (an instruction the model could be argued around); a production system should add independent output validation/classification and least-privilege tool scoping as additional, harder-to-bypass layers.
C. No further guardrails are needed since the underlying model is trained to be safe by default.
D. The fix is exclusively to increase the system prompt's length.

---

### Domain 6 — Stakeholder Communication & Lifecycle Management

**Q25.** During discovery for a new Claude-based solution, a stakeholder requests "an agentic system using the newest model" without describing the underlying business problem, expected volume, data sensitivity, or success criteria. What is the most appropriate next step?

A. Proceed to build exactly what was requested, since the stakeholder is the domain expert.
B. Conduct structured discovery to elicit the underlying business problem, measurable success criteria, expected scale, and data-sensitivity/compliance constraints before committing to any specific architecture pattern or model.
C. Select the newest, largest model immediately, since that satisfies the literal request.
D. Skip discovery and begin with agentic architecture, since that was explicitly named.

**Q26.** An architect is preparing a decision record for a choice between a RAG-based approach and a fine-tuning approach for a knowledge-grounded assistant. Which artifact best reflects sound Domain 6 documentation practice?

A. A final architecture diagram only, with no explanation of alternatives considered.
B. A record capturing what was decided, which alternatives (including fine-tuning) were considered, and the specific reasons they were rejected in favor of RAG.
C. A verbal explanation given once in a meeting with no written record.
D. Documentation limited strictly to code comments in the implementation repository.

**Q27.** A team commits to a 500ms p95 latency SLA with a client before any model, architecture, or scale analysis has been performed. During implementation, the chosen architecture (a multi-hop agentic pipeline needed for the required accuracy) cannot sustain the SLA. What does this scenario illustrate?

A. SLAs should always be set as low (fast) as possible regardless of architecture.
B. SLA commitments must be negotiated against what the chosen architecture can actually sustain at expected scale — this should happen after design trade-offs are understood, not before.
C. The engineering team is solely at fault and no process change is needed.
D. Multi-hop agentic pipelines can always meet any SLA if given enough budget.

**Q28.** Which lifecycle phase is most clearly being skipped if a team documents an architecture as fully "shipped" and considers the project complete at launch, with no defined plan for what gets watched afterward or how production feedback reaches the design team?

A. Discovery.
B. Design.
C. Monitoring and iteration.
D. Handoff.

---

### Domain 7 — Developer Productivity & Operational Enablement

**Q29.** A platform team is rolling out an AI coding assistant across the engineering organization with broad default access to production infrastructure credentials and deployment tooling. Which principle from elsewhere in the blueprint most directly applies to configuring this tool safely?

A. Prompt caching, since it primarily affects cost, not access scope.
B. Least privilege / capability-bloat analysis — the same discipline applied to customer-facing agents' tool access should apply to an AI coding assistant's access to infrastructure and deployment capabilities.
C. Chain-of-thought prompting, since it improves reasoning quality.
D. RAG chunking strategy, since it primarily concerns retrieval, not tooling.

**Q30.** An engineer uses an AI-assisted debugging tool to generate a candidate root-cause explanation and fix for a production incident. What is the most appropriate way to use this output, consistent with sound operational practice?

A. Deploy the suggested fix immediately without further verification, since the tool is generally reliable.
B. Treat the suggestion as a candidate that still requires the same verification discipline (testing, review) as any other proposed fix, rather than shipping it unverified.
C. Ignore AI-assisted tooling entirely for production incidents regardless of potential time savings.
D. Use the tool only to write documentation, never for actual debugging assistance.

---

## Section 2: Answer Key and Rationale

**Q1 — B.** An unpredictable, variable-length resolution path is exactly the case for an agentic pattern (the model plans dynamically) rather than a fixed workflow graph, which structurally cannot flex to a step count it wasn't built for. Adding more fixed steps (A) or a bigger model at fixed steps (C) doesn't address the structural mismatch; running the same graph twice (D) doesn't add missing capability.

**Q2 — A and C.** The accuracy is already excellent and the problem is purely latency against SLA, so the fix should reduce hop count/serialize-to-parallel where possible (C) and avoid paying multi-agent overhead on requests that don't need it (A). Adding a fifth worker (B) or a bigger supervisor model (D) both add cost/latency in the wrong direction without addressing the stated SLA problem.

**Q3 — B.** Doing existing work (contract review) with substantially less headcount for equivalent or better output is the textbook definition of the efficiency pillar. It is not "transformation" (A) since contract review wasn't previously impossible; it is not primarily an SLA commitment (C, though turnaround improved slightly) or developer productivity (D, no engineers are described).

**Q4 — B.** The described diagram stops at "response returned" with no logging, no HITL path, and no mechanism to route production data back into improvement — precisely the feedback-loop requirement Domain 1 calls out as part of end-to-end architecture design. It is not primarily about value-pillar alignment (A), multi-agent orchestration (C, no agents described), or risk-tier decomposition (D, no routing described at all).

**Q5 — B.** A single sprawling prompt trying to handle every branch via deep nested conditionals is a decomposition failure; the fix is to route by intent first and dispatch to smaller, focused prompts — not to add more reasoning instructions (A) or a bigger model (C) on top of the same monolithic design, and moving logic into few-shot examples (D) doesn't resolve the underlying structural problem.

**Q6 — B.** With a heavy-tailed distribution (85% simple), a cheap router that only escalates the harder 15% to the expensive model captures most of the cost savings while preserving accuracy on the cases that need it. Uniform context reduction (A) or blanket caching (C) don't address the model-cost driver, and downsizing every request (D) risks accuracy loss on the 15% that actually need the larger model.

**Q7 — B.** Separating stable and variable prompt content also enables single-point updates to shared policy/role logic across every call site — a modular-prompt-reuse benefit distinct from (though complementary to) caching. It does not eliminate the need for output guardrails (A), guarantee no hallucination (C), or shrink the context window (D).

**Q8 — B.** Fabricating unsupported numeric figures is a hallucination/grounding problem, a different failure mode from formatting inconsistency that few-shot examples address; the fix is retrieval grounding or explicit refusal instructions, not more/longer examples (A), and chain-of-thought (C) targets reasoning-step errors, not fabrication from ungrounded generation. Model deprecation (D) is an unsupported leap from the symptom described.

**Q9 — B and C.** Skills and modular prompt composition are both explicitly reuse strategies meant to be applied across many call sites/tasks. A one-time detailed prompt (A) is the opposite of reuse, and adjusting temperature (D) is a generation-parameter choice, not a reuse strategy.

**Q10 — A and B.** Removing entirely unnecessary destructive/mutation tools (A) and scoping the remaining access to only what the actual task (leave-balance queries) requires (B) both directly apply least privilege. A warning message (C) is a weak compensating control that leaves the excess capability in place, and a larger model (D) does not change authorization scope at all.

**Q11 — B.** A single, fixed, well-known service with no planned expansion is exactly the case where direct API/CLI integration is appropriate — MCP's dynamic-discovery value (A) isn't needed here and isn't "always most secure" regardless of context, and agent-to-agent communication (C) is for delegating to an autonomous peer agent, not a static internal service. The three mechanisms are not interchangeable (D).

**Q12 — B and C.** Exact part-number lookup needs keyword/exact-match retrieval alongside semantic search (vector search alone often misses precise identifiers), and frequently-changing pricing data needs an explicit freshness/re-indexing SLA to avoid serving stale prices as current. Treating all data uniformly as vector-embedded chunks (A, D) mismatches data shape to retrieval strategy in both cases.

**Q13 — B.** A tool/resource set that has grown into the dozens is exactly the threshold where progressive discovery (lightweight index, load full detail only as needed) is architecturally necessary to control both token cost and tool-selection accuracy — continuing monolithic loading (A) is the described failure itself, removing guardrails (C) is unrelated and unsafe, and a bigger context window alone (D) doesn't fix the tool-selection accuracy problem, only the token-fit problem.

**Q14 — B.** A single shared, over-broad, long-lived credential is a textbook large-blast-radius authorization gap; the direct fix is scoped, per-tool, ideally short-lived credentials matched to actual need. This is not a model-selection issue (A), not fixable by a prompt instruction alone (C), and logging (D) is a detective control that doesn't reduce the blast radius of a leaked broad credential.

**Q15 — B.** Once a request spans multiple hops, a propagated trace ID is the baseline requirement to reconstruct what happened for any single failure — without it, individual hop logs can't be correlated. Weekly average latency (A) hides tail behavior and is too coarse; full unredacted logging by default (C) creates cost/compliance exposure without being the most important first addition; removing logging (D) is the opposite of sound practice.

**Q16 — B.** A stable golden-set accuracy score says nothing about safety/security behavior on adversarial or unusual inputs; rising unsafe-output complaints despite flat accuracy is exactly the gap that dedicated adversarial/red-team evaluation is meant to catch, which this suite lacks. A larger golden set of the same kind (C) or switching scoring method (D) doesn't add the missing safety dimension, and (A) misreads a stable accuracy number as sufficient evidence of overall system health.

**Q17 — B.** Canary a single changed variable, measure the full metric set over a meaningful sample, then ramp — this isolates cause and catches regressions before full exposure. Changing multiple variables at once and deploying to 100% immediately (A), skipping canary testing entirely (C), or ramping to 100% within an hour regardless of results (D) all violate sound iterative-testing practice.

**Q18 — B.** With model version, temperature, and latency all held constant and only the document corpus having changed, the retrieval/indexing layer is the most likely and most directly implicated first place to check — not silent model drift (A), a temperature issue (C), or a context-window issue (D), none of which would be selectively triggered by a document refresh.

**Q19 — A.** Relying solely on LLM-as-judge scoring with no human validation of the judge's own reliability is the least appropriate approach precisely because judge bias or drift would go undetected in a high-stakes domain — this is a general risk of using an unvalidated judge as the sole method, distinct from combining it with other methods (B, C, D), each of which represents sound complementary practice.

**Q20 — B.** A golden-set-only re-evaluation missed a hallucination-rate regression introduced by the smaller model — illustrating precisely why the full metric set (including safety/hallucination-focused evaluation), not accuracy alone, must be re-run after any cost/model-size change. The optimization itself may still be salvageable with fixes (A overstates the failure); hallucination and model capacity are directly related here, not unrelated (C); and immediate permanent reversion without further investigation (D) skips the diagnostic step that would identify a more targeted fix.

**Q21 — A and B.** A Business Associate Agreement with any PHI-touching vendor and enforcing minimum-necessary access via tool/permission scoping are both specific, named HIPAA-relevant architectural requirements. Model size (C) and chain-of-thought prompting (D) are unrelated to HIPAA compliance specifically.

**Q22 — B.** Automating the confident majority (below-$500) while escalating only the uncertain or higher-risk tail (above $500) is confidence/threshold-triggered escalation, whose core advantage is scaling review effort to where it's actually needed rather than reviewing everything. Full pre-execution gating on every request (A) or full human review of everything (D) don't scale the way this design does, and post-hoc sampled review (C) isn't what's described here since high-value requests are reviewed before completion, not after.

**Q23 — B.** Disparate outcomes across demographic subgroups on comparable inputs is a fairness/bias signal requiring disaggregated evaluation and human review given the legally consequential nature of hiring decisions — overall accuracy alone (A) can mask subgroup disparity, model size (C) doesn't guarantee reduced bias, and removing evaluation of the disparity (D) increases rather than reduces legal and ethical risk.

**Q24 — B.** A single system-prompt instruction is a comparatively weak, arguable-around guardrail layer; sound practice adds independent output validation/classification and tool-permission scoping as harder-to-bypass layers on top. Treating the current single layer as sufficient (A) or assuming default model training obviates further guardrails (C) both understate real risk, and simply lengthening the prompt (D) doesn't add a structurally independent layer.

**Q25 — B.** A request framed only as a technology choice ("agentic," "newest model") without the underlying problem, success criteria, scale, or data sensitivity is exactly the case for structured discovery before any architecture commitment — proceeding directly to build (A, C) or committing to a named pattern before discovery (D) all skip the requirement-gathering step this domain tests.

**Q26 — B.** A decision record naming the decision, the alternatives considered (including the rejected fine-tuning approach), and the specific reasons for rejection is the documentation standard this domain expects — a diagram alone (A), a verbal-only explanation (C), or code-comment-only documentation (D) all fail to preserve a reviewable rationale trail.

**Q27 — B.** Committing to a latency SLA before understanding what the necessary architecture (here, a multi-hop pipeline required for accuracy) can actually sustain at scale is the described failure; SLA negotiation must follow, not precede, architecture/trade-off analysis. Setting SLAs unconditionally low (A) or assuming unlimited budget solves any latency problem (D) are not sound general principles, and placing sole fault on engineering with no process fix (C) misses the actual lesson.

**Q28 — C.** Treating launch as project-complete with no defined post-launch observation or feedback path is precisely a skipped monitoring-and-iteration phase — discovery (A), design (B), and handoff (D) are implied to have already occurred by the time of "launch" in this stem.

**Q29 — B.** An AI coding assistant with broad default infrastructure/deployment access is subject to the same capability-bloat and least-privilege analysis this blueprint applies to customer-facing agents' tool access — this is the cross-domain principle most directly applicable, not caching (A), chain-of-thought (C), or chunking strategy (D), none of which concern access scope.

**Q30 — B.** AI-assisted debugging output should be treated as a candidate requiring the same verification (testing, review) as any other proposed fix before deployment — deploying unverified (A) risks shipping an unvalidated change, blanket avoidance of the tooling (C) forfeits a real productivity gain for no safety benefit, and restricting the tool to documentation only (D) is an unnecessarily narrow use of a capability this domain explicitly expects teams to apply to debugging.
