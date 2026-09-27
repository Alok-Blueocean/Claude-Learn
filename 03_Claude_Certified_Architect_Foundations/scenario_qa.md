# Additional Architecture Scenarios (Original)

12 original architecture scenarios, beyond the official 6 in `official_scenarios.md`, covering real architecture decisions the official scenario bank doesn't exercise directly — migrating a monolithic prompt to a multi-agent system, PII handling in a RAG pipeline, fine-tuning vs. prompting, multi-region failover, and more. Each entry states the situation, then walks through what you'd decide and why, naming which of the 5 CCAR-F domains the reasoning draws on. These are written in the same "what would you decide and why" depth as the official scenarios, for study purposes — they are not exam content.

---

## Scenario A: Migrating a Monolithic Prompt to a Multi-Agent System

**Situation:** Your team has a single, very long system prompt (11,000 tokens) driving one Claude instance that handles contract review end-to-end: extracting clauses, flagging risk, checking against a playbook, and drafting redlines. It works, but it's slow, the prompt is nearly impossible to safely edit without regressing an unrelated section, and quality has plateaued. Should you move to a multi-agent architecture, and how would you do it safely?

**What you'd decide and why:**
- **Diagnose before restructuring.** A monolithic prompt handling four distinct responsibilities (extraction, risk-flagging, playbook comparison, drafting) is a strong candidate for decomposition, but "it's slow and hard to edit" doesn't automatically justify a multi-agent rewrite — first confirm the four responsibilities have genuinely different tool needs, different failure modes, or benefit from parallelism. Here they do: clause extraction and risk-flagging can run over independent chunks of the same document in parallel, and drafting genuinely depends on the other three finishing first.
- **Choose a coordinator-subagent architecture, not a peer-to-peer one** (Domain 1, Task Statement 1.2): a coordinator owns the sequencing (extraction and risk-flagging in parallel → playbook comparison → drafting), and all inter-stage handoffs flow through it for observability and consistent error handling.
- **Split tools by role** (Domain 2, Task Statement 2.3): the extraction subagent gets only document-parsing tools; the playbook-comparison subagent gets only playbook-lookup tools; neither needs drafting tools, and the drafting subagent doesn't need raw document-parsing access if it's handed structured clause data.
- **Pass explicit, structured context between stages** (Domain 1, Task Statement 1.3): the extraction subagent's output (clauses with page/section metadata) goes directly into the risk-flagging and playbook-comparison subagents' prompts — not "the subagent will remember," since subagents don't inherit context automatically.
- **Migrate incrementally, not as a single rewrite.** Use plan mode (Domain 3, Task Statement 3.4) to design the decomposition and validate it against a held-out set of contracts before cutting the old monolithic prompt over, since a full-system rewrite risks regressing quality across all four responsibilities simultaneously with no isolated point of comparison.
- **Domains exercised:** 1 (Agentic Architecture & Orchestration), 2 (Tool Design & MCP Integration), 3 (Claude Code Configuration & Workflows, via plan-mode-driven migration design).

---

## Scenario B: PII in a RAG Pipeline for Internal HR Documents

**Situation:** You're building a retrieval-augmented internal HR assistant. The knowledge base includes performance reviews, compensation bands, and medical-accommodation records. Employees should be able to ask general HR-policy questions freely, but the system must never surface another employee's PII (salary, medical details, review content) in a retrieved chunk, even indirectly through a paraphrase.

**What you'd decide and why:**
- **This is a data-boundary problem, not a prompting problem — solve it before the retrieval step, not after.** Instructing the model in its system prompt to "never reveal other employees' PII" is exactly the probabilistic-compliance pattern the exam warns against for anything with real consequences (Domain 1, Task Statement 1.4/1.5) — a prompt-only safeguard has a non-zero failure rate, and a leaked salary or medical detail is not an acceptable rare failure.
- **Enforce access scoping programmatically at the retrieval layer**, before any chunk ever reaches the model's context: the retrieval tool itself should filter candidate documents by the requesting employee's own authorization scope (their own records, plus genuinely public policy documents), so PII-bearing documents belonging to other employees are never retrievable in the first place, regardless of what the prompt says.
- **Design the retrieval tool's error response deliberately** (Domain 2, Task Statement 2.2): a query that would have matched a document outside the requester's scope should not silently return zero results indistinguishable from "no matching policy exists" — return a structured, explicit "restricted" signal so the agent can respond appropriately ("that information isn't available to you") instead of the coordinator/agent guessing why nothing came back.
- **Add a hook as a second, independent layer** (Domain 1, Task Statement 1.5): a `PostToolUse`-style interception on retrieval results that scans for known PII patterns (SSNs, other-employee identifiers) as a backstop, even though the primary control is the retrieval-scoping itself — defense in depth, not reliance on either layer alone.
- **This is explicitly an area where "prompt harder" is the wrong answer** — the exam guide's own distinction between programmatic enforcement and prompt-based guidance for anything with real-world compliance stakes applies directly here.
- **Domains exercised:** 1 (Agentic Architecture & Orchestration — enforcement patterns), 2 (Tool Design & MCP Integration — tool/error design), 5 (Context Management & Reliability — what enters context in the first place).

---

## Scenario C: Fine-Tuning vs. Prompting for a Domain-Specific Insurance-Claims Classifier

**Situation:** A team wants Claude to classify incoming insurance claims into 40 fine-grained subcategories used by their claims-routing system, and a stakeholder proposes fine-tuning a custom model on 15,000 labeled historical claims to "guarantee" accuracy on this specific taxonomy.

**What you'd decide and why:**
- **Fine-tuning and training custom models are explicitly out of scope for this exam's tested content** (see `tutorial.md`'s Out-of-Scope appendix) — but the *architecture decision* of prompting vs. an alternative is squarely a real design question, and the reasoning below stays within tested territory: strong prompt engineering plus structured output, not a training-pipeline discussion.
- **Try the higher-leverage, lower-cost path first**: a `tool_use` schema with the 40 subcategories as an enum (plus an `"other" + detail` fallback for genuine novel cases, and an `"unclear"` value for ambiguous claims — Domain 4, Task Statement 4.3), paired with few-shot examples covering the categories that are semantically closest to each other and most often confused (Domain 4, Task Statement 4.2). This is often sufficient for exactly the kind of ambiguous-boundary classification a 40-way taxonomy creates.
- **Validate with stratified, per-category accuracy measurement before concluding prompting "isn't accurate enough"** (Domain 5, Task Statement 5.5): an aggregate accuracy figure can look fine while 3 of the 40 categories — the ones fine-tuning was actually being proposed to fix — perform badly. Measure per-category, not just overall, before deciding the simpler approach has failed.
- **Route low-confidence or genuinely ambiguous classifications to human review** (Domain 5, Task Statement 5.5) rather than accepting either "the model is always right" or "we need a bespoke trained model" as the only two options — confidence-calibrated review is a middle path the exam tests directly.
- **The architecturally sound answer here is not "fine-tune" vs. "prompt harder" as a binary** — it's: exhaust schema design, few-shot examples, and confidence-routed human review first, and treat "needs a custom-trained model" as a conclusion you reach only after measuring where prompting-based structured output actually falls short, not a default reached for convenience.
- **Domains exercised:** 4 (Prompt Engineering & Structured Output), 5 (Context Management & Reliability — confidence calibration).

---

## Scenario D: Designing for Multi-Region Failover

**Situation:** Your customer-facing agent currently calls the Claude API from a single region. Leadership wants resilience against a regional outage without over-engineering a system that, today, has no compliance requirement to keep customer data in a specific region.

**What you'd decide and why:**
- **Note what's out of scope first.** Specific cloud-provider configurations, infrastructure/networking, and MCP server hosting/deployment are explicitly out of scope for this exam (`tutorial.md` appendix) — so the tested part of this scenario isn't "how do you configure AWS multi-region routing," it's the **application-architecture reliability patterns** layered on top, which are in scope.
- **Design the agent's error-handling and retry logic to treat a regional API outage as a transient, structured error** (Domain 2, Task Statement 2.2), not a silent failure or an unhandled exception — the agent-facing architecture should already distinguish `transient` (retry-worthy) failures from other categories, and a regional outage is the same failure shape as any other transient dependency failure, just at a different layer.
- **For any multi-step or multi-agent workflow in flight during a regional failover, design for graceful resumption, not silent data loss** (Domain 1, Task Statement 1.7; Domain 5, Task Statement 5.4): structured state exports (a manifest of what each agent had completed) mean a coordinator resuming after a failover event can reload state and continue, rather than restarting a long-running multi-agent workflow from scratch.
- **Don't let a resilience initiative silently turn into a prompt-compliance problem**: if failover ever means a request gets replayed against a second region mid-conversation, make sure conversation history and any session-scoped state travels with it — a customer support agent that "forgets" the first 20 turns of a dispute because of an invisible regional switch is a context-management regression (Domain 5, Task Statement 5.1), not just an infrastructure detail.
- **Domains exercised:** 2 (Tool Design & MCP Integration — error categorization), 5 (Context Management & Reliability — state persistence and resumption across a failure boundary), with an explicit acknowledgment of what infrastructure-level content is out of scope.

---

## Scenario E: A CI Code-Review Bot That Developers Have Started Ignoring

**Situation:** Your Claude-powered PR review bot has been live for six months. Metrics show developers now merge PRs without reading its comments roughly 70% of the time, up from 15% at launch. No single category has an extreme false-positive rate, but the bot's comment volume per PR has crept up over time as more checks were added.

**What you'd decide and why:**
- **This is a signal-to-noise problem, and the exam's guidance on explicit criteria applies at the aggregate level, not just per-category** (Domain 4, Task Statement 4.1): even if no single category is egregiously wrong, a bot that comments on everything trains developers to stop reading, the same way one very-high-false-positive category erodes trust in the guide's single-category example — just distributed across many categories instead of concentrated in one.
- **Audit per-category value, not just per-category accuracy**: measure not just "is this finding correct" but "does a developer act on it" — a technically-correct but consistently-ignored category (e.g., minor style nits that duplicate what a linter already reports) is adding noise even when it isn't technically a false positive.
- **Apply the same "temporarily disable, then rework" pattern used for high-false-positive categories** (Domain 4, Task Statement 4.1) to categories that are technically accurate but low-value — the goal is restoring signal density, and removing low-value volume is a legitimate lever even without a strict "false positive" framing.
- **Consider whether review architecture, not just criteria, is contributing**: if PRs have grown larger over the same period, check whether a single-pass review over increasingly large diffs is producing the kind of superficial, high-volume commentary that comes from attention dilution (Domain 4, Task Statement 4.6) — the fix there is splitting into focused per-file plus integration passes, which naturally tends to produce fewer, better-targeted findings per file rather than a long undifferentiated list.
- **Domains exercised:** 3 (Claude Code Configuration & Workflows — CI integration health), 4 (Prompt Engineering & Structured Output — explicit criteria and multi-pass review).

---

## Scenario F: Choosing Between a Single Powerful Agent and a Multi-Agent Pipeline for Customer Onboarding

**Situation:** A fintech company wants an agent to guide new customers through account opening: identity verification, risk-tier assignment, document collection, and initial product recommendations. A team debates whether this should be one agent with all the necessary tools, or a coordinator with specialized subagents.

**What you'd decide and why:**
- **The deciding question is tool count and role separation, not "multi-agent is inherently better."** The exam is explicit that too many tools on one agent (e.g., 18 instead of 4-5) degrades tool-selection reliability (Domain 2, Task Statement 2.3) — if this single agent would need identity-verification tools, document-OCR tools, a risk-scoring service, and a product catalog all at once, that's a strong signal toward decomposition regardless of any abstract architectural preference.
- **Identity verification is the clearest case for programmatic enforcement, not agent judgment** (Domain 1, Task Statement 1.4): risk-tier assignment and product recommendations should be **blocked programmatically** until identity verification has returned a positive result — this is a compliance-critical ordering constraint, not a preference to express in a system prompt.
- **Decompose by role, with the coordinator owning sequencing**: an identity-verification subagent (narrow toolset, hard gate), a document-collection subagent, and a recommendation subagent, coordinated so that recommendation only runs after risk-tier assignment completes — mirroring the hub-and-spoke pattern (Domain 1, Task Statement 1.2) rather than one agent juggling every tool and every ordering constraint itself via prompt instructions alone.
- **Structure the human-handoff path for verification failures or ambiguous risk signals** (Domain 1, Task Statement 1.4; Domain 5, Task Statement 5.2): a structured handoff summary (what identity signals were checked, what didn't match, recommended next step) for a compliance reviewer, since this is a domain where an escalation without full context is a real business risk, not just an inconvenience.
- **Domains exercised:** 1 (Agentic Architecture & Orchestration), 2 (Tool Design & MCP Integration), 5 (Context Management & Reliability — escalation).

---

## Scenario G: Recovering From an MCP Server That Silently Started Returning Stale Data

**Situation:** An internal inventory MCP server had a caching bug for three weeks: it kept returning inventory counts from before a warehouse migration, without ever raising an error. Your fulfillment agent used these stale counts to confirm orders that couldn't actually be filled, and nobody noticed until customer complaints spiked.

**What you'd decide and why:**
- **This is fundamentally an error-design gap, not a caching bug in isolation** — the caching bug was the trigger, but the reason it went undetected for three weeks is that the tool's response shape gave the agent (and any monitoring) no way to distinguish "fresh, correct data" from "stale data returned with full confidence" (Domain 2, Task Statement 2.2). A tool that can silently serve stale data needs a way to signal staleness, not just a value.
- **Add explicit freshness/provenance metadata to the tool's response schema**: a `data_as_of` timestamp alongside the inventory count, so the agent (or a downstream check) can reason about whether the data is recent enough to act on, rather than treating every successful response as equally current — this is the same principle tested for multi-source synthesis (Domain 5, Task Statement 5.6: temporal data needs explicit dates), applied to a live operational tool rather than a research citation.
- **Distinguish this from a legitimate "valid empty/unchanged result"** (Domain 2, Task Statement 2.2; Domain 5, Task Statement 5.3): the server wasn't returning an access failure, and it wasn't returning a genuinely valid "no change" either — it was returning a confidently wrong value with no signal at all, which is a strictly worse failure mode than either of the two the exam's error-handling guidance normally contrasts, and the fix is the same one applied more broadly: never let "successful response" alone imply "trustworthy response."
- **At the agent-architecture level**, consider a `PostToolUse` hook (Domain 1, Task Statement 1.5) that checks a returned `data_as_of` field against a staleness threshold and blocks order confirmation — turning "the data looked fine" into a programmatically enforced guarantee rather than trusting each response at face value.
- **Domains exercised:** 2 (Tool Design & MCP Integration — error/response design), 1 (Agentic Architecture & Orchestration — hooks as a backstop), 5 (Context Management & Reliability — provenance).

---

## Scenario H: A Documentation-Generation Agent That Contradicts Itself Across a Large Repository

**Situation:** An agent tasked with generating API documentation for a 300-endpoint service produces docs where endpoints in the `auth/` module are described as requiring a bearer token, while functionally identical endpoints in `admin/` (which also require the same bearer token, per the actual code) are documented as requiring no authentication at all.

**What you'd decide and why:**
- **Diagnose this as attention dilution across a single large pass, the same underlying mechanism as the exam's multi-file code-review scenario** (Domain 1, Task Statement 1.6; Domain 4, Task Statement 4.6), just applied to documentation generation instead of review: processing 300 endpoints in one continuous pass produces inconsistent depth and, here, outright contradictory conclusions about functionally identical patterns depending on where in the pass they fall.
- **Restructure as a two-phase decomposition**: a per-module pass that documents each module's endpoints locally (consistent depth per module), followed by a dedicated cross-module consistency pass that specifically checks for exactly this kind of contradiction — endpoints sharing an auth pattern being documented inconsistently — rather than trusting a single pass to self-correct.
- **Add few-shot examples anchoring the authentication-detection judgment specifically** (Domain 4, Task Statement 4.2): a couple of examples showing "how to recognize a bearer-token requirement in code, even when the enforcing middleware is applied at the module level rather than per-endpoint" would directly address why `admin/`'s shared, module-level auth check was missed — this is exactly the kind of ambiguous-pattern generalization few-shot examples are suited for.
- **This is not primarily a context-window-size problem** — a bigger context window lets the model see more endpoints at once, but doesn't fix the attention-quality degradation that causes inconsistent conclusions about superficially different code that behaves identically; the fix is decomposition and few-shot pattern-anchoring, not scale.
- **Domains exercised:** 1 (Agentic Architecture & Orchestration — task decomposition), 4 (Prompt Engineering & Structured Output — few-shot prompting, multi-pass review).

---

## Scenario I: Replacing a Slow, Bespoke Internal Tool with a Community MCP Server

**Situation:** Your team built a custom MCP server wrapping your internal Confluence-like wiki two years ago. It's become a maintenance burden, and a well-maintained community MCP server for the same underlying wiki platform now exists. An engineer proposes replacing the custom server, but another argues "our custom one has business-specific logic baked in."

**What you'd decide and why:**
- **The exam's default guidance is to prefer existing, well-maintained community MCP servers for standard integrations, reserving custom development for genuinely team-specific workflows** (Domain 2, Task Statement 2.4) — so the right first step isn't "keep the custom server because it's custom," it's identifying exactly which parts of the custom server's logic are genuinely business-specific versus generic wiki access that the community server already covers.
- **Separate the concerns rather than treating this as all-or-nothing**: use the community server for standard operations (search, page retrieval, page creation) and, if truly necessary, keep a much smaller custom layer (or a small set of custom tools) only for the genuinely business-specific logic (e.g., a custom `find_policy_by_department` tool that encodes internal document-organization conventions the generic server has no way to know about) — this is the same "purpose-specific tool with a defined contract" principle used for splitting overloaded generic tools (Domain 2, Task Statement 2.1), applied to deciding what still deserves to be custom.
- **Before finalizing, check the community server's tool descriptions against your agent's needs** (Domain 2, Task Statement 2.4): if its descriptions under-explain its own capabilities relative to your old custom tool, the agent may under-use it and keep preferring less-capable built-in fallbacks — the fix there is enhancing the community server's tool descriptions in your own configuration layer, not writing a replacement server from scratch.
- **Domains exercised:** 2 (Tool Design & MCP Integration).

---

## Scenario J: Structured Extraction From Scanned, Handwritten Medical Intake Forms

**Situation:** A healthcare intake system extracts structured fields (patient-reported symptoms, medication list, allergy list) from scanned, sometimes handwritten intake forms. Some forms are missing fields entirely (left blank), some have illegible handwriting in a field, and the allergy list format varies wildly (comma-separated, one per line, "NKDA" for "no known drug allergies," or left blank meaning "not asked").

**What you'd decide and why:**
- **Schema design has to encode three distinct "no value" meanings, not collapse them into one null** (Domain 4, Task Statement 4.3): a blank field (never asked), an explicit "NKDA"/none-reported, and illegible handwriting are three semantically different situations that a downstream clinical system needs to treat differently — one nullable field can't carry that distinction. Model this with an enum status field per section (e.g., `allergy_status: "reported" | "none_reported" | "not_asked" | "illegible"`) alongside the actual extracted list, rather than a single nullable `allergies` field that can't tell "the patient has no allergies" from "we don't know."
- **Few-shot examples should specifically cover the varied allergy-list formats** (Domain 4, Task Statement 4.2) — comma-separated, one-per-line, and the "NKDA" convention — since this is exactly the kind of format variety few-shot examples generalize well against, more reliably than a prose instruction trying to enumerate every format upfront.
- **For illegible handwriting specifically, retry is the wrong tool** (Domain 4, Task Statement 4.4): if a field is genuinely illegible in the source image, no amount of re-prompting manufactures a value that was never legible — the correct behavior is flagging it (`illegible`) for human review, not retrying the extraction hoping for a different guess.
- **Route to human review based on both model confidence and source-document ambiguity independently** (Domain 5, Task Statement 5.5): a confidently-extracted value from a form with a genuinely ambiguous/contradictory allergy section should still route to review, since high model confidence doesn't resolve a source document that is itself unclear — the two signals are independent triggers, not substitutes for each other.
- **Domains exercised:** 4 (Prompt Engineering & Structured Output), 5 (Context Management & Reliability — human review routing).

---

## Scenario K: A Developer-Productivity Agent That Keeps Re-Exploring the Same Codebase Structure

**Situation:** Engineers use a Claude Code-based agent to answer ad hoc questions about a large legacy monorepo throughout the day ("where does order cancellation get triggered from?", "what calls into the pricing engine?"). Each new question starts a fresh session, and the agent re-derives the monorepo's high-level module structure from scratch every single time, burning several minutes and a meaningful number of tokens before it even gets to the actual question.

**What you'd decide and why:**
- **This is a repeated-cost problem the exam's context-management guidance addresses directly**: rather than re-deriving high-level structure every session, persist a scratchpad-style summary of the monorepo's module map (top-level modules, key entry points, where major subsystems like pricing and order-processing live) that a new session can load upfront (Domain 5, Task Statement 5.4) — this converts a repeated exploration cost into a one-time investment that's refreshed periodically rather than redone on every question.
- **This is also a CLAUDE.md-scoping decision, not just a runtime one** (Domain 3, Task Statement 3.1): a project-level `CLAUDE.md` (or an `@import`ed architecture-overview file) describing the monorepo's module layout gives *every* session — not just a manually-loaded scratchpad — this context for free at session start, without depending on each engineer remembering to load anything.
- **Use `Grep`-first, `Read`-to-follow incremental exploration for anything not already captured in the persisted map** (Domain 2, Task Statement 2.5), rather than reverting to reading files broadly whenever the static map doesn't have the answer — the persisted map handles the common "where does X live" questions; incremental grep-then-read exploration handles the genuinely novel ones.
- **Keep the persisted map current deliberately**: since a stale module map that no longer reflects a since-refactored subsystem is worse than no map (a confidently wrong answer vs. an honestly effortful one), treat it as something to periodically regenerate or spot-check, not a write-once artifact — directly analogous to the exam's warning about resuming sessions with stale tool results (Domain 1, Task Statement 1.7).
- **Domains exercised:** 3 (Claude Code Configuration & Workflows — CLAUDE.md), 5 (Context Management & Reliability — persisted state across sessions), 2 (Tool Design & MCP Integration — built-in tool selection for incremental exploration).

---

## Scenario L: Deciding Whether an Escalation Threshold Should Be a Dollar Amount or a Judgment Call

**Situation:** Your customer support agent currently escalates any refund request over $500 to a human (a hard programmatic threshold). A product manager asks whether this threshold should instead be "escalate whenever the situation seems complicated," arguing the fixed dollar amount is "too rigid" and sometimes escalates simple, clearly-justified large refunds unnecessarily while letting smaller but genuinely ambiguous cases through untouched.

**What you'd decide and why:**
- **These are two different problems, and the fix should not collapse them into one control.** The $500 threshold is a **compliance/business-risk boundary** — the kind of rule the exam explicitly treats as needing deterministic, programmatic enforcement (a hook blocking `process_refund` above the threshold) precisely because it's a rule with real financial consequences, not a judgment call (Domain 1, Task Statement 1.4/1.5). Replacing it with "seems complicated" reintroduces exactly the probabilistic-compliance risk the hook exists to eliminate — a plainly justified $50,000 refund and a fraudulent $50,000 refund would now be escalated or not based on the model's subjective read of "complicated," which is not an acceptable substitute for a hard financial control.
- **The PM's actual complaint is about escalation *calibration* for ambiguous smaller cases, which is a separate, legitimate problem** — and the right fix for *that* is the tested pattern from Domain 5, Task Statement 5.2: explicit escalation criteria with few-shot examples distinguishing "straightforward, resolve autonomously" from "policy gap or genuine ambiguity, escalate" — independent of dollar amount.
- **The correct architecture keeps both controls, layered, not merged**: a hard programmatic dollar-threshold gate (deterministic, for the compliance boundary) **and** a judgment-based escalation-criteria layer below that threshold (for ambiguity, policy gaps, and customer preference, per Domain 5, Task Statement 5.2) — a large, clearly-justified refund still escalates (that's the point of the hard gate; a human confirms and releases it quickly), while a small but genuinely ambiguous case now also escalates, because the *judgment* layer catches what the dollar threshold was never designed to catch.
- **The lesson generalizes**: "this deterministic rule is imperfect for judgment-based edge cases" is not evidence the rule should become a judgment call — it's usually evidence you need a second, judgment-based layer *in addition to*, not *instead of*, the deterministic one.
- **Domains exercised:** 1 (Agentic Architecture & Orchestration — hooks/enforcement), 5 (Context Management & Reliability — escalation criteria).
