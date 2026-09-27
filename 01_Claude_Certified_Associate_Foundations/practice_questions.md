# Practice Questions — CCAO-F

32 original practice items, written to match the style, difficulty, and cognitive level of the official
sample questions (see `official_samples.md`). These are **not** copies or rewordings of the official
samples — they are new scenarios built from the same blueprint objectives. Distribution is
proportional to domain weight:

| Domain | Weight | Items here |
|---|---|---|
| 1. Prompting and Task Execution | 14% | 4 (Q1–Q4) |
| 2. Output Evaluation and Validation | 21% | 7 (Q5–Q11) |
| 3. Product and Model Selection | 12% | 4 (Q12–Q15) |
| 4. Workflow Integration and Solution Design | 16% | 5 (Q16–Q20) |
| 5. Configuration and Knowledge Management | 12% | 4 (Q21–Q24) |
| 6. Governance, Risk, and Responsible Use | 15% | 5 (Q25–Q29) |
| 7. Troubleshooting and Optimization | 10% | 3 (Q30–Q32) |

Each item states how many responses to select, matching the real exam's item format. Answer key
with rationale follows the questions.

---

## Domain 1: Prompting and Task Execution

**Q1.** A benefits coordinator asks Claude, "Write something about our new PTO policy." The output is
vague and doesn't match the tone the coordinator wanted. What is the most likely root cause? (Select 1)

A. Claude cannot write HR communications.
B. The prompt did not specify audience, tone, format, or the policy details to include.
C. The task should have been decomposed into ten separate prompts.
D. The coordinator should have used a lower-cost model.

**Q2.** A consultant needs a 40-page vendor-evaluation report restructured into a client-ready
deliverable. Which approach best reflects effective task decomposition? (Select 1)

A. Paste the entire report and ask Claude to "make it better" in one request.
B. Ask Claude to propose a section outline first, get sign-off on the outline, then draft one section at
   a time.
C. Ask ten different people to each rewrite one section independently with no shared instructions.
D. Ask Claude to shorten the report to one paragraph regardless of content.

**Q3.** After reviewing a first draft, a marketing manager says the tone is "close but too stiff." Which
follow-up instruction is most likely to produce a useful revision? (Select 1)

A. "Try again."
B. "Make it better."
C. "Keep the same structure and facts, but rewrite in a warmer, more conversational tone — contractions are fine, avoid passive voice."
D. "Use a different model."

**Q4.** Which task type most requires the prompt to explicitly state whether Claude should rely only on
supplied material versus gather current outside information? (Select 1)

A. Brainstorming internal team names.
B. Researching a fast-moving competitive or regulatory topic.
C. Reformatting a fixed internal memo.
D. Drafting a thank-you note from a fixed template.

---

## Domain 2: Output Evaluation and Validation

**Q5.** Claude summarizes a 60-page vendor contract and includes a specific dollar penalty figure for
late delivery. The associate cannot find that figure anywhere in the contract. What does this most
likely indicate? (Select 1)

A. The contract's formatting confused the summarizer, but the figure is safe to keep.
B. A hallucination — the figure should be verified against the contract text or removed before use.
C. Claude is being overly cautious and probably rounded correctly.
D. The associate should ask Claude to re-state its confidence level and trust whichever number is repeated.

**Q6.** Which two of the following are reliable techniques for catching hallucinated content in a
Claude-generated summary? (Select 2)

A. Asking Claude for a direct quote and location for each factual claim, then checking it against the source.
B. Trusting any claim stated in a confident, fluent tone.
C. Cross-checking specific numbers, names, or citations against the original source document.
D. Asking Claude how confident it is on a 1–10 scale and accepting anything rated 8+.

**Q7.** An output draft contains a total in the executive summary that doesn't match the sum of the
line items in the table two paragraphs later. What should the associate do? (Select 1)

A. Ship it — minor arithmetic differences don't matter in an executive summary.
B. Flag and resolve the inconsistency before the document goes out; internal contradictions are a low-cost, high-value signal to check.
C. Delete the table since the summary number is probably right.
D. Ask a colleague to guess which number is correct.

**Q8.** Which scenario most clearly requires human review before an output is used? (Select 1)

A. An internal brainstorm list of possible team lunch spots.
B. A first-draft outline for a personal to-do list.
C. A customer-facing legal disclosure drafted with Claude's help.
D. A rough internal summary of notes from a meeting the associate personally attended.

**Q9.** A recruiter asks Claude to draft candidate rejection language and then asks it to summarize
"why candidates from School X tend to underperform in interviews" based on the recruiter's own
notes. What is the primary responsible-use concern with the second request? (Select 1)

A. It's outside Claude's context window.
B. It risks encoding and reinforcing bias into a people-facing decision process based on limited, possibly unrepresentative notes.
C. It should be routed to the fastest, cheapest model.
D. It requires a Project with a connector to Google Drive.
E. It is a request Claude Architects must handle instead.

**Q10.** A single analysis needs to reach both a technical engineering team and a non-technical
executive sponsor. What is the best practice? (Select 1)

A. Send the exact same technical draft to both audiences to save time.
B. Produce two adapted versions from the same underlying facts — one detailed/technical, one summarized for decision-making — without changing the substance.
C. Only write for the executive sponsor since they approve budget.
D. Ask the engineering team to translate the executive version themselves.

**Q11.** An associate needs to hand a colleague a list of 40 vendor names with pricing tiers that will be
imported into a spreadsheet. Which output format is most appropriate? (Select 1)

A. A long prose paragraph.
B. A structured table or CSV-style output with consistent columns.
C. A single-sentence summary.
D. An Artifact containing only a title and no data.

---

## Domain 3: Product and Model Selection

**Q12.** A team runs the same weekly report using the same tone, structure, and three reference
documents every time. Which product feature best fits this recurring need? (Select 1)

A. A brand-new plain chat conversation each week, re-explaining context from scratch.
B. A Project configured with standing instructions and the reference documents as knowledge sources.
C. Switching models each week at random.
D. An Artifact with no reusable configuration attached.

**Q13.** A financial analyst needs deep, nuanced reasoning over an ambiguous, high-stakes multi-year
forecast with significant judgment calls. Cost and speed are secondary to getting it right. Which
approach best fits? (Select 1)

A. The fastest, lowest-cost model tier, run many times in parallel.
B. The highest-capability model tier suited to complex reasoning, even at higher cost/latency.
C. Disabling all product features to reduce ambiguity.
D. Splitting the task across five unrelated conversations with no shared context.

**Q14.** A long-running conversation has drifted across five unrelated topics over several days and
responses have become less focused. What is the most appropriate response? (Select 1)

A. Keep going in the same thread indefinitely; length doesn't matter.
B. Start a new, well-scoped conversation, optionally carrying forward a short summary of the relevant prior decisions.
C. Switch to a lower-cost model to save money in the same thread.
D. Delete the Project entirely.

**Q15.** Which two factors should most directly drive the choice between a faster/cheaper model tier
and the most capable model tier for a given task? (Select 2)

A. The complexity/stakes of the reasoning required.
B. The visual theme of the chat interface.
C. The acceptable cost and latency for the task's volume.
D. The number of browser tabs the associate has open.

---

## Domain 4: Workflow Integration and Solution Design

**Q16.** An operations lead wants to apply Claude to a multi-step intake process. Which step of the
current process is the best first candidate for AI augmentation? (Select 1)

A. The step requiring a manager's final sign-off and accountability.
B. A repetitive, rules-based triage/categorization step with a clear, describable pattern.
C. The step that requires in-person relationship context unavailable to Claude.
D. The step that is already fully automated by other software.

**Q17.** A team wants Claude to help redesign, not just augment, a workflow. Which consideration
should most influence whether they choose redesign over augmentation? (Select 1)

A. Redesign is always strictly better and lower-risk, so it should always be chosen.
B. Redesign captures more value but requires more organizational change management, risk tolerance, and governance oversight than augmentation.
C. Redesign requires no human review at any stage.
D. Redesign and augmentation are identical in practice.

**Q18.** A stakeholder asks a project manager to "just tell me Claude will make this process perfect."
What is the most appropriate, calibrated response? (Select 1)

A. Agree, since it keeps the stakeholder happy.
B. Explain realistic benefits (faster drafts, consistency) alongside real limitations (need for review, no guarantee of factual accuracy without verification, no organizational authority).
C. Say Claude can do nothing useful here to avoid overpromising.
D. Avoid answering and change the subject.

**Q19.** Which use of Claude best supports the "research, planning, and process optimization" objective
of Domain 4? (Select 1)

A. Using Claude to synthesize background material and draft a first-pass project plan for a human to refine and approve.
B. Letting Claude make the final go/no-go decision on a vendor contract with no review.
C. Using Claude only to check spelling in a finished plan.
D. Avoiding Claude entirely for planning work since it's a "soft skill."

**Q20.** A team wants to compare a "Claude-augmented" version of their monthly reporting workflow
against the old manual version before rolling it out company-wide. Which comparison approach is
most sound? (Select 1)

A. Assume the new workflow is faster because it feels faster.
B. Measure actual before/after time spent, error rate, or revision cycles on comparable reports.
C. Skip measurement since qualitative impressions are sufficient.
D. Only ask the most enthusiastic team member for their opinion.

---

## Domain 5: Configuration and Knowledge Management

**Q21.** A Project's knowledge base contains two versions of the same pricing sheet — one from last
quarter and one current. Outputs have started citing outdated prices. What is the most appropriate
fix? (Select 1)

A. Add a third pricing sheet for good measure.
B. Remove or replace the outdated pricing sheet so only the current, correct version remains as a knowledge source.
C. Tell users to mentally ignore old prices when they see them.
D. Switch to a different model tier.

**Q22.** Which instruction, added to a Project's custom instructions, best reflects an effective
system-level instruction? (Select 1)

A. "Be helpful."
B. "Always respond as if writing for a non-technical retail-store audience, keep replies under 150 words, and flag explicitly when a question falls outside the uploaded knowledge base."
C. "Answer everything, always guess if unsure."
D. "Ignore all uploaded documents."

**Q23.** A team wants Claude to work from live, up-to-date files in a shared drive rather than manually
re-uploading documents each week. Which capability addresses this? (Select 1)

A. A one-time file upload repeated manually every Monday.
B. A connector (e.g., to Google Drive) scoped to the relevant shared folder.
C. Increasing the model tier.
D. Disabling the Project's custom instructions.

**Q24.** Six months after a Project was configured, the underlying company policy it references has
changed twice. What is the correct practice? (Select 1)

A. Leave the Project as originally configured since it worked at launch.
B. Periodically review and update the Project's knowledge sources and instructions to reflect current policy.
C. Delete the Project since configurations can't be changed.
D. Only fix it if a customer complains.

---

## Domain 6: Governance, Risk, and Responsible Use

**Q25.** A support lead wants to paste raw customer chat logs containing full names, emails, and
payment card fragments into a general-purpose conversation to look for common complaint themes.
What is the most appropriate action? (Select 1)

A. Paste the logs as-is since the goal is internal analysis.
B. Strip or mask the regulated personal and payment data before analysis, consistent with data-sensitivity practice.
C. Proceed, but ask Claude nicely not to remember it.
D. Cancel the analysis project entirely; there's no safe way to ever do this.

**Q26.** Which of the following is most clearly an appropriate use case for an Associate-level user
without further escalation? (Select 1)

A. Drafting a first-pass internal memo that a human will review before sending.
B. Making a final, unreviewed automated hiring rejection decision at scale.
C. Building a custom enterprise-wide agentic integration into core HR systems.
D. Overriding organizational data-sharing policy because a task feels low-risk.

**Q27.** An organization's internal AI policy prohibits uploading unredacted health records to any
general AI tool, even though a proposed analysis "seems harmless." What should the associate do?
(Select 1)

A. Follow internal policy and not upload unredacted records, regardless of personal risk judgment.
B. Upload the records since the associate personally judges the risk to be low.
C. Ask a colleague outside the compliance function to approve an exception informally.
D. Proceed and disclose it only if someone asks later.

**Q28.** Which two statements best reflect responsible-use judgment about accountability? (Select 2)

A. The human who uses Claude's output remains accountable for that output.
B. "The AI generated it" is a sufficient defense for an inaccurate deliverable that shipped without review.
C. Fluent, confident phrasing is not proof of accuracy and shouldn't be over-trusted.
D. Once content passes through Claude, no further human judgment is required.

**Q29.** A hiring manager wants Claude to help write structured interview questions but is unsure
whether to disclose AI assistance to candidates. What is the most appropriate approach? (Select 1)

A. Never disclose AI involvement under any circumstances.
B. Check organizational disclosure norms/policy for the context, since transparency expectations vary by use case and jurisdiction.
C. Assume disclosure is never necessary for internal drafting tools.
D. Let each hiring manager decide privately with no consistent policy.

---

## Domain 7: Troubleshooting and Optimization

**Q30.** A prompt asking Claude to "review this 30-page proposal" repeatedly returns generic,
surface-level feedback. What is the most likely fix? (Select 1)

A. Repeat the exact same prompt several more times.
B. Add specific review criteria (e.g., "assess pricing competitiveness, timeline realism, and risk section completeness against these three requirements") instead of an open-ended "review this."
C. Switch to a lower-cost model tier.
D. Give up on using Claude for proposal review.

**Q31.** A team notices that outputs are inconsistent every time they ask for a "status update" because
each person phrases the request differently. What is the most effective long-term fix? (Select 1)

A. Ask everyone to stop requesting status updates.
B. Build and share a reusable prompt template with clearly labeled fields (project name, period, audience, sections required).
C. Increase the model tier only.
D. Accept the inconsistency as unavoidable.

**Q32.** A workflow currently sends every routine, high-volume ticket-classification request to the
most capable (and most expensive) model tier, and the team wants to reduce cost without hurting
quality where it matters. What is the best optimization? (Select 1)

A. Route routine, high-volume classification to a faster/cheaper tier and reserve the most capable tier for genuinely complex edge cases.
B. Switch everything to the cheapest tier with no regard for task complexity.
C. Stop using Claude for this workflow entirely.
D. Keep everything on the most expensive tier since cost doesn't matter.

---

## Answer Key and Rationale

**Q1: B.** Vague, off-target output almost always traces back to missing audience/tone/format/content specifics in the prompt, not a capability gap or a model-tier issue.

**Q2: B.** Sequential decomposition — outline first, sign-off, then section-by-section drafting — prevents a large deliverable from being built around the wrong structure.

**Q3: C.** Specific, actionable feedback (what to keep, what to change, concrete tone guidance) drives useful revision; vague feedback ("try again," "make it better") gives the model nothing new to act on.

**Q4: B.** Fast-moving competitive/regulatory topics can be stale in a model's existing knowledge; the prompt should specify whether current, externally-gathered information is needed.

**Q5: B.** A specific figure that cannot be found in the source document is a textbook hallucination — a plausible, confident-sounding fabrication that must be verified or removed.

**Q6: A, C.** Requesting quotable, locatable claims and independently cross-checking specifics against the source are reliable checks; confident tone and self-reported confidence scores are not.

**Q7: B.** An internal arithmetic inconsistency is a free, low-cost signal of a likely error and should be resolved before the document is used, not ignored or guessed at.

**Q8: C.** External, legal/compliance-facing content is exactly the high-stakes, externally-consequential case that requires human review before release; the other options are low-stakes/internal/personal.

**Q9: B.** Using limited personal notes to generalize about a school or group risks encoding bias into a people-facing decision process — a core ethical-implications concern, not a technical or model-tier issue.

**Q10: B.** Re-expressing the same underlying facts for different audiences (technical vs. executive) without changing substance is the correct adapt-for-audience practice; sending one-size-fits-all content or skipping an audience is not.

**Q11: B.** Data destined for a spreadsheet import needs consistent, structured fields — a table/CSV-style format — not prose or a title-only artifact.

**Q12: B.** Recurring work sharing the same tone/structure/reference material is precisely what a Project's standing instructions and knowledge sources are for; re-explaining context weekly in plain chat wastes effort.

**Q13: B.** When stakes and ambiguity are high and cost/speed are secondary, the most capable model tier is the appropriate choice, even at higher cost/latency.

**Q14: B.** A conversation that has drifted across many topics benefits from a fresh, well-scoped start, optionally carrying forward only the relevant summary — not indefinite continuation in an overloaded thread.

**Q15: A, C.** Model-tier choice should be driven by the complexity/stakes of the reasoning needed and the cost/latency budget for the task's volume — not cosmetic or unrelated factors.

**Q16: B.** Repetitive, rules-based, clearly-patterned steps are the best automation-augmentation candidates; steps requiring accountability, relationship context, or that are already automated are poor or moot candidates.

**Q17: B.** Redesign captures more value than simple augmentation but requires more change management and governance — it is not categorically risk-free or always the default choice.

**Q18: B.** Calibrated stakeholder communication states real benefits and real limitations honestly, rather than overpromising, dismissing the tool outright, or avoiding the question.

**Q19: A.** Using Claude to accelerate background synthesis and a first-pass plan for human refinement matches the "research, planning, and process optimization" objective; letting it make unreviewed final decisions does not.

**Q20: B.** Sound before/after comparison requires actual measurement (time, error rate, revisions), not assumption or a single opinion.

**Q21: B.** A stale, duplicate knowledge source actively producing wrong answers should be removed/replaced, not supplemented or worked around informally.

**Q22: B.** A good system-level instruction is specific about audience, tone, length, and what to do when material is missing; vague ("be helpful") or harmful ("always guess," "ignore documents") instructions are not.

**Q23: B.** A scoped connector to the live source (e.g., a specific Drive folder) keeps Claude working from current files without manual re-upload; increasing model tier or disabling instructions doesn't address the staleness problem.

**Q24: B.** Configuration is living and must be reviewed/updated as the policies or source material it depends on change — it is not a one-time setup task.

**Q25: B.** Regulated personal and payment data should be masked/stripped before analysis in a general-purpose conversation, consistent with data-sensitivity practice; asking the model to "not remember" doesn't satisfy a policy control.

**Q26: A.** A reviewed first-pass draft is a clearly appropriate Associate-level use case; unreviewed high-stakes automated decisions, enterprise-system integration builds, and policy overrides exceed Associate scope or violate governance.

**Q27: A.** Organizational policy governs regardless of an individual's personal risk assessment; informal exceptions and quiet non-disclosure are not appropriate substitutes for following policy.

**Q28: A, C.** The human using the output remains accountable, and fluent phrasing isn't proof of accuracy; claiming "the AI did it" as a defense, or assuming no further human judgment is needed, are the incorrect framings.

**Q29: B.** Disclosure norms vary by context and jurisdiction, so the correct action is to check applicable organizational policy rather than adopt a blanket always/never rule or leave it to individual discretion.

**Q30: B.** Generic feedback on an open-ended "review this" request is fixed by supplying specific review criteria, not by repeating the same vague prompt or changing model tier.

**Q31: B.** A shared, reusable template with labeled fields removes phrasing-driven inconsistency at its source; this is a durable fix, unlike a one-off model-tier change.

**Q32: A.** Routing routine, high-volume, low-complexity work to a faster/cheaper tier while reserving the top tier for genuinely complex cases reduces cost without sacrificing quality where it matters — the core cost/speed/quality alignment principle from Domain 3 applied as an optimization.
