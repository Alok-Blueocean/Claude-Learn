# Practice Questions (Original)

30 original, scenario-anchored practice questions covering all 5 CCAR-F domains, weighted proportionally to the exam blueprint. These are **not** official Anthropic exam content — they are written for this study kit to reinforce the same knowledge/skill bullets tested on the real exam, using new scenarios not drawn from the official exam guide. Some items are multiple-response ("select two") to mirror the real exam's item format, which always states how many responses to select.

**Distribution** (target ≈ blueprint weight): Domain 1 — 8 questions (27%) · Domain 2 — 5 questions (18%) · Domain 3 — 6 questions (20%) · Domain 4 — 6 questions (20%) · Domain 5 — 5 questions (15%)

## Answer Key (Quick Reference)

| Q | Answer | Q | Answer | Q | Answer |
|---|---|---|---|---|---|
| 1 | B | 11 | B | 21 | C |
| 2 | A, C | 12 | A | 22 | B |
| 3 | C | 13 | B | 23 | A |
| 4 | B | 14 | D | 24 | B, D |
| 5 | A | 15 | A | 25 | C |
| 6 | D | 16 | C | 26 | A |
| 7 | B | 17 | B | 27 | B |
| 8 | A, D | 18 | A | 28 | C |
| 9 | C | 19 | B | 29 | A |
| 10 | A | 20 | A | 30 | B, C |

Full rationale for every item follows below, grouped by domain.

---

## Domain 1: Agentic Architecture & Orchestration (8 questions)

### Question 1

You've implemented an agentic loop for an inventory-management agent. During code review, a teammate proposes this termination check: "stop the loop once the assistant's response contains the phrase 'task complete' anywhere in its text." What is the strongest objection to this design?

A. It will run too many iterations before terminating, wasting tokens.
B. Text content is not a structured signal; the loop should terminate based on `stop_reason` being `"end_turn"`, since the model can complete a task without using that exact phrase, or use the phrase while still intending to call another tool.
C. The phrase "task complete" should be replaced with a JSON flag the model outputs in a text block.
D. This approach is correct, but it should also check for `"tool_use"` as a secondary signal.

**Correct Answer: B.** Parsing natural-language text for a completion signal is exactly the anti-pattern the SDK's `stop_reason` field exists to eliminate — a model can produce any phrasing, in any position, independent of whether it's actually finished acting. `stop_reason` (`"tool_use"` vs. `"end_turn"`) is the only structurally reliable termination signal. Option C still relies on parsing text output instead of `stop_reason`. Option D misunderstands the values — `"tool_use"` means *continue*, not terminate.

### Question 2 (select two)

A coordinator agent for a legal-document review pipeline delegates to a "clause extraction" subagent and a "risk scoring" subagent. Which **two** of the following are consistent with the hub-and-spoke coordinator pattern this exam tests?

A. The risk-scoring subagent never communicates directly with the clause-extraction subagent — it receives clause data only via the coordinator's prompt to it.
B. The clause-extraction and risk-scoring subagents share a single conversation history so each can see the other's reasoning automatically.
C. The coordinator decides whether to invoke the risk-scoring subagent at all, based on whether the clause-extraction output indicates any high-risk clause types are present.
D. Each subagent independently decides, without coordinator involvement, which other subagent to hand off to next.

**Correct Answer: A, C.** Hub-and-spoke means all inter-subagent communication and routing decisions flow through the coordinator, and the coordinator dynamically decides which subagents a given query actually needs rather than always invoking the full pipeline. B is wrong because subagents have isolated context and do not automatically share history. D is wrong because it describes subagent-to-subagent routing, which breaks the coordinator's role as the single point of observability and control.

### Question 3

Your multi-agent onboarding-documentation generator has a coordinator that decomposes "document our authentication system" into: "document the login endpoint" and "document the password-reset endpoint." The generated docs are technically accurate but completely omit OAuth SSO and session-token refresh, both of which are part of the authentication system. What is the most likely root cause, and what's the fix?

A. The synthesis subagent needs instructions to flag missing sections in its output.
B. The document-analysis subagent's relevance filtering is too strict and is excluding SSO-related source files.
C. The coordinator's task decomposition is too narrow — it should decompose "authentication system" into a set of subtasks that actually spans all its known components (login, password reset, SSO, session/token refresh) before delegating.
D. The report-generation subagent should be given a broader style template that includes an SSO section by default.

**Correct Answer: C.** This mirrors the "narrow decomposition" failure mode: subagents did exactly what they were assigned and produced correct output — the problem is that the coordinator's decomposition never assigned anyone to cover SSO or token refresh at all. The fix is at the decomposition step, not by patching downstream subagents or templates to compensate for an incomplete task breakdown.

### Question 4

A coordinator's `allowedTools` configuration for a research assistant does not include `"Task"`. What is the direct architectural consequence?

A. The coordinator can spawn subagents but they will run with restricted permissions.
B. The coordinator cannot invoke any subagents at all, since `Task` is the mechanism the Agent SDK uses to spawn them.
C. Subagents will run but will not receive any context from the coordinator's prompt.
D. The coordinator will fall back to sequential prompt chaining instead of subagent delegation automatically.

**Correct Answer: B.** The `Task` tool is the spawning mechanism for subagents in the Agent SDK, and a coordinator's `allowedTools` must include it for subagent invocation to be possible at all — this is a hard requirement, not a soft restriction. There is no automatic fallback behavior (D), and the failure is total rather than partial (A, C).

### Question 5

A synthesis subagent in a market-research pipeline needs a competitor's Q3 revenue figure that the web-search subagent already found two steps earlier, but the synthesis subagent's prompt only says "combine the earlier findings into a report." The synthesis subagent hallucinates a plausible-sounding but incorrect revenue figure. What is the most direct fix?

A. Include the web-search subagent's actual findings (the specific revenue figure and its source) directly in the text of the synthesis subagent's prompt, since subagents do not automatically inherit the coordinator's or another subagent's context.
B. Give the synthesis subagent a `web_search` tool so it can look the figure up itself if it's missing.
C. Add a validation step that checks the synthesis output's numbers against a knowledge base after generation.
D. Instruct the synthesis subagent's system prompt to "always double-check figures against provided data."

**Correct Answer: A.** The root cause is that the actual findings were never explicitly placed in the synthesis subagent's prompt — subagent context isolation means "the earlier findings" is not something the subagent can see unless it's spelled out. B over-provisions a specialized agent with a tool outside its role (see Domain 2, Task Statement 2.3) instead of fixing the real problem: missing context. C and D are after-the-fact mitigations for a problem that a correct prompt design prevents outright.

### Question 6

An engineering team is deciding how to break down "modernize our test suite from an old assertion library to the new one, across 200 files with varying test patterns." Which task decomposition strategy is most appropriate, and why?

A. Fixed sequential prompt chaining: process files 1 through 200 in a hard-coded order, one per turn.
B. A single pass that attempts to rewrite all 200 files' tests in one very long response.
C. Dynamic, adaptive decomposition: first survey the codebase to characterize the different test patterns present, identify the highest-impact/most-common patterns, then build a prioritized migration plan that adapts as new patterns are discovered.
D. Skip decomposition — since it's "just" a library swap, direct execution with a single find-and-replace instruction is sufficient.

**Correct Answer: C.** This is an open-ended investigation task where what's needed depends on what's discovered (different files will use different assertion patterns needing different rewrites) — exactly the case for adaptive decomposition rather than a fixed pipeline. A assumes the work is uniform and predictable, which "varying test patterns" explicitly contradicts. B risks the same attention-dilution failure mode tested elsewhere (one pass over too much at once). D underestimates the task; "varying test patterns" signals real complexity a naive find-and-replace will get wrong.

### Question 7

A developer is mid-way through a multi-hour Claude Code session investigating a performance regression across a large codebase. They step away, and by the time they return, several of the files that were analyzed earlier in the session have since been modified by a teammate's merged PR. What is the most reliable way to continue?

A. Use `--resume` with the session name and continue immediately, since the analysis and conversation are still mostly valid.
B. Inform the resumed session specifically about which files changed since the earlier analysis, so it re-analyzes only the affected parts rather than trusting stale tool results or requiring a full re-exploration.
C. Start a completely new session from scratch and re-run the entire investigation, discarding all prior findings.
D. Use `fork_session` to branch a new investigation path from the current baseline.

**Correct Answer: B.** The core risk called out for session resumption is relying on stale tool results after underlying files have changed. The reliable pattern is to resume but explicitly tell the agent what changed, enabling targeted re-analysis — not blind resumption (A, which risks reasoning over outdated file contents) and not a wasteful full restart (C, which discards everything that's still valid). `fork_session` (D) is for exploring divergent approaches from a shared baseline, not for handling stale context after external file changes.

### Question 8 (select two)

A payments-integration agent uses a `PostToolUse` hook. Which **two** of the following are appropriate uses of this hook, consistent with how the exam tests hook patterns?

A. Normalizing three different payment-gateway MCP tools' differently-formatted transaction timestamps (Unix epoch, ISO 8601, and a custom `MM/DD/YYYY HH:mm` string) into one consistent format before the model reasons over them.
B. Asking the model to double-check its own refund calculation via an extra reasoning step.
C. Blocking any outgoing `issue_refund` call above $1,000 and redirecting to a human-approval workflow.
D. Intercepting a tool's *result* to strip out sensitive account fields not needed for the current task before they enter context.

**Correct Answer: A, D.** `PostToolUse` hooks intercept **tool results** for transformation before the model processes them — data normalization (A) and trimming irrelevant/sensitive fields (D) are both textbook uses. B is not a hook at all — it's a prompt-based self-review instruction, which is exactly the weaker, probabilistic approach hooks are meant to replace for guaranteed compliance. C is a real, valid interception pattern, but it intercepts an **outgoing tool call** (a different hook point), not a tool *result* — so it doesn't fit "appropriate uses of `PostToolUse`" as asked.

---

## Domain 2: Tool Design & MCP Integration (5 questions)

### Question 9

Your agent has two tools: `fetch_ticket` ("Gets ticket information") and `fetch_ticket_history` ("Gets ticket history information"). Logs show the agent frequently calls `fetch_ticket` when it actually needs the audit trail of status changes, and vice versa. What is the most effective fix, applied first?

A. Merge both into one `fetch_ticket_full` tool that always returns everything.
B. Add five few-shot examples per tool showing correct selection.
C. Rewrite both descriptions to state exactly what each returns, give a concrete example query each should handle, and explicitly state the boundary ("use `fetch_ticket_history` for status-change audit trails; use `fetch_ticket` for current ticket state — not history").
D. Rename `fetch_ticket_history` to `fetch_ticket_v2` to signal it's the newer, more complete tool.

**Correct Answer: C.** As with the official exam's tool-description guidance, the first and highest-leverage fix for ambiguous, near-identical tool descriptions is to make the descriptions themselves precise and mutually exclusive — including an explicit boundary statement. A is a valid alternative architecture but skips straight to consolidation before trying the cheaper fix. B adds token overhead without addressing the actual gap (the descriptions give the model nothing to differentiate on). D actively makes tool selection worse by implying a version relationship that has nothing to do with the tools' actual purposes.

### Question 10

An MCP tool for a hotel-booking agent currently returns `{"error": "failed"}` on every failure — whether the third-party booking API timed out, the requested room type doesn't exist, or the request lacked required permissions. What structured error design would most improve the agent's ability to recover appropriately?

A. Return `errorCategory` (e.g., `transient`, `validation`, `permission`), an `isRetryable` boolean, and a human-readable description for every failure.
B. Return the raw HTTP status code from the underlying booking API.
C. Return `{"error": "failed", "severity": "high"}` for all failures so the agent knows to escalate.
D. Suppress the error and return an empty availability list instead, so the conversation continues smoothly.

**Correct Answer: A.** This is the canonical structured-error pattern: category + retryability + human-readable detail lets the agent distinguish a worth-retrying timeout from a validation error it should reformulate from a permission failure it should escalate — none of which a uniform "failed" status supports. B exposes an implementation detail (HTTP codes) without the semantic categories the agent actually needs to act on. C still collapses every distinct failure mode into one signal. D is the "silently suppress errors as success" anti-pattern — a real failure gets mislabeled as a legitimate empty result.

### Question 11

A synthesis agent in a due-diligence pipeline is given full read/write access to the same document-repository tools the research subagent uses, "in case it needs to pull something up." Post-launch, you observe the synthesis agent occasionally attempting its own exploratory document searches instead of relying on the research subagent's findings, sometimes contradicting the research subagent's own conclusions. What principle was violated, and what's the fix?

A. This is expected and desirable — more tool access always improves output quality.
B. Giving an agent tools outside its specialization increases the likelihood of misuse; the synthesis agent should be scoped to only the tools its role needs (with, at most, one narrow cross-role tool for a specific high-frequency need), not the full document-repository toolkit.
C. The fix is to give the research subagent write access to the synthesis agent's report instead.
D. The fix is to increase the synthesis agent's context window so it doesn't need to re-search.

**Correct Answer: B.** This is the tool-distribution principle tested directly: broad, unscoped tool access — even to a capable, related toolkit — measurably increases the odds of an agent using tools outside its intended role, undermining the separation of concerns the architecture depends on. The fix is scoped access per role, not more access (A) or an unrelated architecture change (C, D).

### Question 12

Your team is deciding where to configure two MCP servers: one is a shared Jira integration every engineer on the team needs, using a token stored as an environment variable; the other is an experimental internal server one engineer is prototyping alone. How should these be configured?

A. The Jira server in project-scoped `.mcp.json` with `${JIRA_TOKEN}` environment variable expansion (version-controlled, shared); the experimental server in that engineer's user-scoped `~/.claude.json` (personal, not shared).
B. Both in project-scoped `.mcp.json`, since MCP servers should always be shared for consistency.
C. Both in user-scoped `~/.claude.json`, since credentials should never be checked into a repository under any circumstances.
D. The Jira server in `~/.claude.json` for security, and the experimental server in `.mcp.json` so the team can review it.

**Correct Answer: A.** This is the standard project-vs-user MCP scoping split: shared team tooling belongs in version-controlled `.mcp.json`, with environment variable expansion keeping the actual secret value out of the repo while the *reference* to it (`${JIRA_TOKEN}`) is safely shared; personal/experimental servers belong in the individual's user-scoped config so they don't affect teammates. B and D invert or collapse this distinction incorrectly. C is overly conservative — env var expansion specifically solves the "credentials in a repo" concern for shared servers.

### Question 13

A developer needs to find every file in a large repository whose filename ends in `.config.ts`, and separately needs to find every place in the codebase that calls a function named `computeShippingCost`. Which built-in tool should be used for each task, respectively?

A. `Grep` for finding the filenames; `Glob` for finding the function callers.
B. `Glob` for finding the filenames; `Grep` for finding the function callers.
C. `Read` for both, since both require inspecting file contents.
D. `Bash` with a custom shell pipeline for both, since built-in tools can't do either reliably.

**Correct Answer: B.** `Glob` is for file **path/name** pattern matching (e.g., `**/*.config.ts`); `Grep` is for searching file **contents** for a pattern (e.g., a function name used as a call). Reversing them (A) reverses their actual purposes. C and D both ignore the purpose-built tools in favor of much less efficient alternatives.

---

## Domain 3: Claude Code Configuration & Workflows (6 questions)

### Question 14

Two engineers on the same team report different Claude Code behavior when working in the same repository: one gets consistent enforcement of the team's commit-message format, the other doesn't. Investigation shows the first engineer has the convention in their personal `~/.claude/CLAUDE.md`. What's the correct fix?

A. Tell the second engineer to manually copy the same text into their own `~/.claude/CLAUDE.md`.
B. Add the convention to a `.claude/skills/` skill instead, so it's invoked on demand.
C. Leave it as-is; personal configuration differences are expected and acceptable for team conventions.
D. Move the convention into the project-level CLAUDE.md (`.claude/CLAUDE.md` or root `CLAUDE.md`) so it is version-controlled and applies to every team member automatically.

**Correct Answer: D.** This is the canonical CLAUDE.md hierarchy diagnostic: user-level settings apply only to that individual and are never shared via version control, so any instruction meant for the whole team belongs at project level. A "fixes" only one more individual and doesn't scale or stay in sync. B misapplies skills (on-demand, task-specific) to what is actually an always-applicable universal standard — exactly what CLAUDE.md is for. C accepts an inconsistency the architecture has a direct fix for.

### Question 15

A team wants a `/deploy-checklist` command available to every developer automatically when they clone the infrastructure repo, and separately, one senior engineer wants a personal `/my-notes` command for their own workflow that shouldn't affect anyone else. Where should each be created?

A. `/deploy-checklist` in `.claude/commands/` (project-scoped, version-controlled); `/my-notes` in `~/.claude/commands/` (user-scoped, personal).
B. Both in `.claude/commands/`, and instruct the senior engineer to prefix their command with their initials.
C. Both in `~/.claude/commands/`, then have the senior engineer share their config file with the team via Slack.
D. `/deploy-checklist` in the root `CLAUDE.md`; `/my-notes` in `.claude/commands/`.

**Correct Answer: A.** Project-scoped commands in `.claude/commands/` are shared via version control automatically to everyone who clones/pulls; user-scoped commands in `~/.claude/commands/` are personal and not shared. B pollutes shared, version-controlled space with a personal command. C manually replicates what project scoping already does automatically, and introduces sync drift. D confuses CLAUDE.md (context/instructions) with slash command definitions, which live in `.claude/commands/`, not CLAUDE.md.

### Question 16

Your organization's monorepo has Terraform files under `infra/**`, Python services under `services/**`, and frontend code under `apps/web/**`, each with different conventions. You want Terraform-specific conventions to load automatically only when Claude is editing Terraform files, regardless of which subdirectory of `infra/` they're in. What is the best mechanism?

A. A single root CLAUDE.md with a "Terraform" section, trusting Claude to apply it only when relevant.
B. A directory-level CLAUDE.md placed at the root of `infra/`.
C. A `.claude/rules/terraform.md` file with YAML frontmatter `paths: ["infra/**/*.tf"]`, so it loads only for matching files regardless of which subdirectory they're in.
D. A custom slash command `/terraform-rules` that developers must remember to invoke before editing Terraform files.

**Correct Answer: C.** Path-specific rules with glob-pattern frontmatter are designed for exactly this: conditional loading by file pattern, robust to files being spread across an arbitrary subdirectory structure. A relies on inference across a large mixed file, which is unreliable. B would work reasonably for this specific case since `infra/` is one directory, but is less robust than a glob pattern if Terraform files ever appear outside `infra/`, and the exam favors path-scoped rules as the more maintainable, general solution over directory-bound files for exactly this kind of type-based (not location-based) convention. D requires manual invocation, contradicting the goal of automatic loading.

### Question 17

You've asked Claude Code to write a data-transformation function based on a two-sentence prose description, and across three attempts it interprets an ambiguous edge case (empty input arrays) differently each time. What is the most effective next step?

A. Repeat the same prose description a fourth time, adding the word "precisely."
B. Provide 2-3 concrete input/output examples, including at least one showing the expected behavior for an empty input array specifically.
C. Switch to a completely different phrasing of the same requirement in hopes it lands differently.
D. Accept whichever of the three attempts looks closest and manually patch the edge case afterward.

**Correct Answer: B.** Concrete input/output examples are the most effective way to communicate an expected transformation when prose descriptions are being interpreted inconsistently — showing, rather than re-describing, resolves exactly this kind of edge-case ambiguity (here, empty-array handling). A and C both keep relying on prose, which has already been shown not to work. D papers over the underlying prompt-design problem rather than fixing it.

### Question 18

Your CI pipeline invokes `claude -p "Review this PR for correctness bugs" --output-format json --json-schema review_schema.json`, and a bot posts inline PR comments from the JSON output. After a new commit is pushed and the review re-runs, developers complain the bot re-posts comments on issues they already fixed in earlier commits, alongside genuinely new ones. What should change?

A. Include the prior review's findings in context on re-run, and instruct Claude to report only new or still-unaddressed issues.
B. Remove `--json-schema` so the output is less rigidly structured and Claude has more freedom to notice what changed.
C. Only run the review once per PR, on the final commit before merge.
D. Increase `max_tokens` so Claude can see more of the diff at once.

**Correct Answer: A.** This is the documented fix for redundant re-review noise: give the re-run visibility into what was already flagged, with an explicit instruction to report only new/unaddressed items — otherwise every re-run re-derives and re-reports the same findings from scratch. B removes a feature (schema-enforced structured output) that isn't the actual problem. C loses the value of fast per-commit feedback. D addresses a token-budget concern unrelated to the duplicate-comment symptom described.

### Question 19

A large, multi-file feature branch needs review, and your CI system currently has Claude Code run in the *same session* that generated the original implementation, with an added instruction: "now critically review your own changes for bugs." Reviewers still report the bot rarely flags anything beyond style nits, even when real bugs are later found by a human. What's the most likely explanation and best fix?

A. The instruction wording is too soft; strengthen it to "you MUST find at least one bug."
B. Self-review from the same session that generated the code retains the original reasoning context, making it structurally less likely to question its own prior decisions — even with an explicit self-critique instruction; use an independent review instance instead, without that reasoning history.
C. The model needs extended thinking enabled to catch its own bugs.
D. The generation and review should both happen in the same message to reduce latency.

**Correct Answer: B.** This is the self-review-vs-independent-review principle: the issue is structural (shared reasoning context suppresses self-doubt), not a matter of instruction phrasing or thinking budget — extended thinking and stronger self-critique wording are both explicitly called out as insufficient fixes. A forced quota ("find at least one bug") risks manufacturing false positives rather than fixing the underlying blind spot. D doesn't address quality at all.

---

## Domain 4: Prompt Engineering & Structured Output (6 questions)

### Question 20

Your automated code-review prompt currently instructs: "Only flag issues you are highly confident about, and be conservative about false positives." Developers still report a high false-positive rate in one specific category (unused-variable warnings misfiring on intentionally-unused destructured values). What is the most effective fix?

A. Replace the vague conservatism instruction with an explicit, checkable criterion specific to that category — e.g., "flag a variable as unused only if it is never referenced anywhere in the function body, including in destructuring patterns used purely for skipping positional values" — with a concrete code example of the pattern to exclude.
B. Strengthen the wording to "be extremely conservative, only report near-certain issues."
C. Remove the unused-variable check entirely and never re-add it.
D. Tell the model to lower its confidence threshold for this category specifically, without further detail.

**Correct Answer: A.** Vague instructions like "be conservative" or "only report high-confidence findings" don't reliably improve precision — they don't give the model a concrete, checkable boundary. Explicit categorical criteria plus a concrete example of what to exclude is the fix that actually works. B is the same failed approach with more emphasis. C is a valid short-term move to protect trust in the *interim*, but it isn't a fix — the guide's pattern is to temporarily disable while reworking criteria, not abandon the category permanently, which C does. D just restates "be less conservative" without giving a criterion either.

### Question 21

An extraction prompt handles invoices with citation styles that vary by vendor: some cite line-item taxes inline ("$40.00 (incl. 8% tax)"), others in a separate footer table. The prompt currently describes this variation in one paragraph of prose, and extraction accuracy on the footer-table style is far lower than on the inline style. What's the most effective fix?

A. Rewrite the paragraph to be longer and more detailed.
B. Increase the model's `max_tokens` so it has more room to reason.
C. Add 2-4 few-shot examples that explicitly demonstrate correct extraction from both the inline-citation style and the footer-table style, so the model can generalize the underlying pattern rather than defaulting to whichever style is more common in its training distribution.
D. Split this into two separate extraction tools, one per vendor citation style, and require a human to pick which one applies before calling either.

**Correct Answer: C.** This is squarely a few-shot-prompting fix: demonstrating both structural variants directly addresses inconsistent handling of "varied document structures" in a way that more prose (A) or more token budget (B) does not — few-shot examples are specifically effective here because they let the model generalize the underlying judgment to structural variants, not just match the literal examples. D adds unnecessary manual overhead and doesn't scale to vendors not yet seen.

### Question 22

You're designing a `tool_use` schema to extract structured data from customer support tickets, including a `resolution_type` field. Some tickets are resolved in ways that don't fit any of your five predefined categories. What schema design choice best prevents both fabricated categorization and silent data loss?

A. Make `resolution_type` a required field restricted to exactly the five existing enum values.
B. Add an `"other"` enum value paired with a free-text `resolution_type_detail` field, and include `"unclear"` as an explicit enum option for genuinely ambiguous cases.
C. Make `resolution_type` entirely optional with no enum constraint at all, accepting any string.
D. Keep the five-value enum but instruct the model in the prompt to "pick the closest match" when none really fit.

**Correct Answer: B.** The `"other" + detail string` pattern is the standard way to keep a schema both structured (queryable, consistent categories) and extensible (doesn't force a bad fit or force fabrication) — and an explicit `"unclear"` value gives the model a legitimate way to signal genuine ambiguity instead of guessing. A forces every case into a fixed set, guaranteeing the model fabricates a fit for the ones that don't. C sacrifices structure entirely, defeating the purpose of a schema. D still forces a bad-fit guess, just less visibly.

### Question 23

A document-extraction pipeline returns a JSON object that passes schema validation (all required fields present, correct types) but the extracted `line_items` don't sum to the extracted `total_amount`. What does this demonstrate, and what's the appropriate response?

A. Schema validation via `tool_use` eliminates syntax errors but not semantic errors; add a self-correction check that separately computes a `calculated_total` from line items and flags a discrepancy against the extracted `stated_total` for review or retry.
B. This indicates the JSON schema itself is broken and needs stricter typing.
C. Switch `tool_choice` from forced selection to `"auto"` to give the model more flexibility.
D. This is expected and does not need to be addressed, since the schema validated successfully.

**Correct Answer: A.** This is the textbook illustration that `tool_use`/schema validation guarantees structural correctness, not semantic correctness — a numerically inconsistent result can still be perfectly valid JSON. The fix is a dedicated semantic check (comparing a computed total to the stated one), which is exactly the self-correction validation flow pattern tested. B misdiagnoses a semantic issue as a typing issue. C loosens the wrong control and doesn't address the actual gap. D ignores a real data-quality problem just because it happened to pass an unrelated check.

### Question 24 (select two)

A team runs two Claude-powered workflows: (1) a blocking API-contract validation check that must finish before a deploy pipeline proceeds, and (2) a weekly summary of API usage trends generated for Monday-morning review. Which **two** statements correctly apply Message Batches API guidance to this situation?

A. Both workflows should move to the Message Batches API for the 50% cost savings, since cost reduction should always be prioritized over latency.
B. The weekly usage-trends summary is an excellent fit for the Message Batches API, since it is non-blocking and latency-tolerant (a report needed for Monday morning, submitted well in advance, easily tolerates up to a 24-hour processing window).
C. If the weekly summary workflow needs the model to call tools mid-request and reason over the results before finishing, that specific requirement cannot be met by the Message Batches API regardless of its latency tolerance.
D. The blocking API-contract validation check is also a good fit for the Message Batches API, since deploy pipelines can simply wait for batch completion.

**Correct Answer: B, C.** The weekly report is a canonical batch-appropriate workload (B). Separately, and correctly, the Batches API's lack of mid-request multi-turn tool-calling support is an independent constraint that applies regardless of latency tolerance (C) — a workflow that needs multi-turn tool use within one logical request can't move to batch even if it's otherwise latency-tolerant. A wrongly claims cost always trumps latency needs. D is the direct anti-pattern the exam tests: batch has no latency SLA, making it wrong for a blocking gate no matter how convenient "just wait" sounds.

### Question 25

A 22-file refactoring PR is reviewed by Claude Code in a single pass. The review flags a null-check omission as critical in `orders/utils.ts` but does not flag the identical omission in `billing/utils.ts`, reviewed later in the same pass. What is the best architectural fix?

A. Ask the model to review the PR twice and manually reconcile the two sets of findings.
B. Reduce the PR size requirement to 5 files maximum going forward, rejecting all larger PRs.
C. Split the review into a per-file pass for local issues (each file analyzed with full, undiluted attention) followed by a separate cross-file integration pass for any cross-cutting concerns, rather than one pass over all 22 files at once.
D. Add an instruction telling the model to "be consistent across all files in this PR."

**Correct Answer: C.** This is attention dilution across a large single pass, and the fix tested throughout the guide is architectural: split into focused per-file passes (consistent depth per file) plus one dedicated integration pass, not a policy change on PR size (B, which burdens developers instead of fixing the review architecture) or a bare consistency instruction (D, which doesn't address the root cause of diluted attention). A adds cost without fixing the underlying mechanism causing the inconsistency.

---

## Domain 5: Context Management & Reliability (5 questions)

### Question 26

A customer support agent's conversation has been running for 40 turns and includes several progressive summarization steps. The customer originally stated "I was charged $214.50 on March 3rd for an order I never received," but by turn 35, the agent's working summary just says "customer has a billing dispute about an undelivered order." The agent then can't correctly process a refund because it no longer has the exact amount or date. What's the architectural fix?

A. Increase the context window so summarization is never needed.
B. Extract transactional facts (amount, date, order status) into a persistent "case facts" block included in every prompt, kept outside the summarized/compressible conversation history.
C. Instruct the summarization step to "preserve important details."
D. Ask the customer to repeat the amount and date again at turn 35.

**Correct Answer: B.** This is the exact progressive-summarization failure mode tested: numeric/transactional specifics get smoothed away by repeated compression. The fix is structural — a separate, non-summarized "case facts" layer that persists the exact figures — not a bigger context window (A, which doesn't stop summarization logic from still discarding detail if triggered) or a vague instruction to the summarizer (C, which is the same "be careful" anti-pattern that doesn't reliably work elsewhere in prompting). D is a workaround that degrades customer experience and doesn't fix the underlying system.

### Question 27

A customer explicitly writes: "I don't want to keep explaining this to a bot, please connect me to a person." The issue itself (a straightforward shipping-address correction) is well within the agent's normal autonomous-resolution capability. What should the agent do?

A. Attempt to resolve the shipping-address correction first, since it's simple, then escalate only if the customer is still unsatisfied afterward.
B. Escalate to a human immediately, honoring the customer's explicit request, without first attempting to investigate or resolve the issue itself.
C. Ask the customer to confirm whether they still want a human, given that the issue looks simple to resolve.
D. Decline to escalate since the issue doesn't meet the complexity bar for escalation.

**Correct Answer: B.** An explicit customer request for a human is one of the appropriate escalation triggers, independent of how "simple" the underlying issue is from the system's point of view — this is different from the case where the issue is straightforward *and the customer hasn't asked for a human*, where offering to resolve first (as in A) is correct. Here, the customer has already made the request explicit, so re-litigating it (C) or overriding it based on the system's own complexity judgment (D) both fail to honor a clear, stated preference.

### Question 28

In a three-subagent research pipeline, the analytics subagent's tool call to an internal data warehouse fails with a connection timeout on one of five planned queries. The subagent returns `{"status": "success", "results": []}` for that query, since — technically — no rows were returned. The coordinator proceeds as if all five data points were successfully gathered, and the final report presents an incomplete dataset as complete. What went wrong?

A. The report-generation subagent should have caught this and added a disclaimer.
B. The coordinator should have run a fifth verification pass to catch the gap.
C. The subagent conflated an access failure (a timeout, which needed a retry/escalation decision) with a valid empty result (a successful query that legitimately found nothing) — these must be reported as distinguishable outcomes so the coordinator can act appropriately on the actual failure.
D. This is acceptable behavior since an empty result set is technically a valid response shape.

**Correct Answer: C.** This is the access-failure-vs.-valid-empty-result distinction tested directly: silently reporting a timeout as a successful empty result is an anti-pattern that hides a real failure from the coordinator, preventing any recovery (retry, alternate query, or an explicit coverage-gap annotation in the final report). A and B propose fixes downstream of where the actual information loss occurred. D accepts an error-masking anti-pattern as fine.

### Question 29

A developer spawns a subagent to answer "which modules in this codebase directly call the legacy `LegacyAuthService`?" during a long refactoring session, so the main agent's context doesn't fill up with the full grep/read trace needed to answer that question. What is this an example of, and why is it the recommended pattern for large codebase exploration?

A. Subagent delegation for verbose discovery — isolating a exploratory, tool-call-heavy investigation in a subagent that returns a summary, so the main agent preserves capacity for high-level coordination rather than accumulating the full raw trace.
B. Session forking, since the developer wants to compare two different investigation strategies.
C. This is unnecessary overhead; the main agent should always do its own exploration directly for accuracy.
D. This only helps with latency, not context management.

**Correct Answer: A.** This is precisely the subagent-delegation-for-context-management pattern: verbose, tool-call-heavy discovery is isolated in a subagent so only a distilled summary re-enters the main agent's context, preventing the context degradation ("inconsistent answers," "typical patterns" hand-waving) that comes from an extended session accumulating too much raw exploratory output. B misidentifies the scenario — there's no divergent-branch comparison here, just delegation of one investigative sub-question. C ignores the documented context-degradation risk of very long single-session exploration. D understates the benefit — the primary value here is context preservation, not (only) speed.

### Question 30 (select two)

A multi-source competitive-intelligence report cites two industry analyst firms whose published market-share figures for the same product category differ by several percentage points, both from reports published within the same month. Which **two** actions are consistent with the exam's guidance on handling conflicting statistics from credible sources?

A. Arbitrarily use whichever figure appears in the more recently published source, discarding the other without comment.
B. Annotate the conflict explicitly in the report, presenting both figures with their respective source attribution rather than silently picking one.
C. Structure the report with a section that separates well-established, broadly-corroborated findings from contested or single-source ones, rather than presenting all findings with uniform confidence.
D. Omit both figures entirely from the report to avoid the appearance of inconsistency.

**Correct Answer: B, C.** The tested guidance is explicit: when credible sources conflict, annotate both values with attribution rather than arbitrarily resolving the conflict (B), and structure the report to distinguish well-established findings from contested ones (C) — both preserve information and let the reader (or downstream system) see the actual state of the evidence. A silently discards a data point without disclosing the conflict existed. D throws away real information the report's reader may need, instead of surfacing the disagreement transparently.
