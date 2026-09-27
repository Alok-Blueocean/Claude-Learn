# Claude Certified Architect — Foundations (CCAR-F) — Study Guide

This is a study guide for the **Claude Certified Architect – Foundations** certification (exam code **CCAR-F**, Version 1.0, effective July 2026). It teaches the architecture judgment the exam actually tests: how to design multi-component systems with Claude, choose between agentic architecture patterns, design tool/MCP interfaces, configure Claude Code for teams, engineer prompts for reliable structured output, and manage context and reliability at the system level.

The exam is scenario-built: 60 items, 120 minutes, 4 scenarios drawn at random from a bank of 6 (see `official_scenarios.md` for all 6 in full), passing score 720/1000. Read this guide alongside `official_scenarios.md` (the highest-value file in this kit — the real exam draws 4 of those 6 scenarios verbatim), `official_samples.md`, `practice_questions.md`, and `scenario_qa.md`.

## Exam Blueprint

| Domain | Content Domain | Weight |
|---|---|---|
| 1 | Agentic Architecture & Orchestration | 27% |
| 2 | Tool Design & MCP Integration | 18% |
| 3 | Claude Code Configuration & Workflows | 20% |
| 4 | Prompt Engineering & Structured Output | 20% |
| 5 | Context Management & Reliability | 15% |
| | **Total** | **100%** |

Weights reflect the relative importance of each domain to competent performance, as determined by Anthropic's job task analysis. Section-level percent-correct is reported on your score but does **not** determine pass/fail — only your total scaled score (100–1,000, cut score 720) does that.

The intended candidate has 6+ months of hands-on experience building with the Claude Agent SDK, Claude Code, MCP, and the Claude API — this guide assumes you've built at least a toy version of each pattern below, not just read about it.

---

## Domain 1: Agentic Architecture & Orchestration (27%)

This is the largest domain, and it's the one the exam scenarios lean on most — 3 of the 6 official scenarios (Customer Support Resolution Agent, Multi-Agent Research System, Developer Productivity with Claude) list it as a primary domain. The core architectural question throughout is: **how much of the decision-making do you delegate to the model, versus enforce deterministically in code?**

### 1.1 The Agentic Loop

An agentic loop is the control-flow pattern underneath every Claude-powered agent: send a request, inspect what Claude wants to do next, act on it, and feed the result back in.

```
loop:
    response = client.messages.create(model=..., messages=history, tools=tools)
    history.append(response)                      # assistant turn joins history
    if response.stop_reason == "tool_use":
        for block in response.content:
            if block.type == "tool_use":
                result = execute_tool(block.name, block.input)
                history.append(tool_result(block.id, result))
        continue                                    # loop again — model sees new info
    elif response.stop_reason == "end_turn":
        return response                              # done — no more actions requested
```

The two structural facts to internalize:

- **Termination is driven by `stop_reason`**, not by anything you infer from the assistant's text. `"tool_use"` means keep going; `"end_turn"` means stop. This is the *only* reliable signal.
- **Tool results must be appended to conversation history** before the next iteration — that's how the model "sees" what happened and reasons about the next step. Skipping this turns the loop into repeated, context-blind requests.

**Anti-patterns the exam calls out explicitly** (these show up as wrong-but-tempting answer choices):

| Anti-pattern | Why it fails |
|---|---|
| Parsing assistant natural-language text ("I'm done now") to decide when to stop | Text is not a reliable, structured termination signal — the model can phrase completion in unlimited ways, and `stop_reason` already exists for exactly this purpose |
| Setting an arbitrary iteration cap as the *primary* stopping mechanism | Caps are a safety net, not a design — a legitimate multi-step task gets cut off, and a broken task silently "succeeds" at the cap |
| Checking for the presence of assistant text content as a completion indicator | Claude can emit explanatory text alongside a tool call; text presence is unrelated to whether the turn is actually done |

The underlying distinction the exam tests repeatedly: **model-driven decision-making** (Claude reasons about which tool to call next, given context) vs. **pre-configured decision trees** (you hard-code the sequence). Good agent architecture leans on the former for judgment calls and the latter (via hooks/prerequisites — see 1.4/1.5) only where compliance must be deterministic.

### 1.2 Coordinator–Subagent Orchestration

The dominant multi-agent pattern tested is **hub-and-spoke**: one coordinator agent owns all inter-subagent communication, error handling, and information routing. Subagents don't talk to each other directly.

```
                 ┌──────────────┐
   user query -> │  Coordinator  │ -> aggregated, cited result
                 └──┬───┬───┬───┘
           ┌────────┘   │   └────────┐
     ┌─────▼────┐  ┌────▼─────┐ ┌────▼─────┐
     │  Search  │  │ Analyze  │ │ Synthesize│
     │ subagent │  │ subagent │ │  subagent │
     └──────────┘  └──────────┘ └──────────┘
```

Key architectural facts:

- **Subagents have isolated context.** They do not automatically inherit the coordinator's conversation history — any context a subagent needs must be explicitly included in its prompt (see 1.3).
- **The coordinator decides which subagents to invoke**, based on query complexity — it should not always route every query through the full pipeline. A simple factual query doesn't need the document-analysis and synthesis stages; a broad research topic does.
- **The single biggest failure mode is overly narrow task decomposition.** If the coordinator decomposes "impact of AI on creative industries" into only "AI in digital art," "AI in graphic design," and "AI in photography," every subagent will execute *correctly* — and the system will still fail, because music, writing, and film were never assigned to anyone. This is the diagnostic pattern behind official Sample Question 7 (see `official_scenarios.md`): when subagents all report success but coverage is missing, look at the coordinator's decomposition, not the subagents.
- **Iterative refinement**: a mature coordinator evaluates synthesis output for gaps, re-delegates to search/analysis with *targeted* follow-up queries, and re-invokes synthesis — rather than treating the pipeline as strictly one-shot.
- **Route everything through the coordinator**, even when it would be technically possible for subagents to call each other, because this is what gives you observability, consistent error handling, and controlled information flow. (Section 1.3 below shows the one narrow, deliberate exception: scoped cross-role tools for high-frequency needs.)

### 1.3 Subagent Invocation, Context Passing, and Spawning

- **The `Task` tool is the spawning mechanism.** A coordinator's `allowedTools` must include `"Task"` or it cannot invoke subagents at all — a common configuration bug.
- **`AgentDefinition`** configures each subagent type: its description (used for routing/selection), its system prompt, and its tool restrictions.
- **Context must be explicit.** Because subagents don't inherit parent context, you pass prior findings *directly in the prompt* — e.g., the web-search subagent's raw results and the document-analysis subagent's summaries go straight into the synthesis subagent's prompt text, not "the subagent will remember what happened."
- **Separate content from metadata using structured formats.** When passing findings between agents, keep source URLs, document names, and page numbers as structured fields alongside the content — collapsing everything into unstructured prose is how attribution gets lost by the time it reaches a report (this recurs in Domain 5.6).
- **Parallel spawning**: emit multiple `Task` tool calls in a *single* coordinator response to run subagents concurrently, rather than sequential turns. This is a straightforward and large latency win (official Sample Question 9's scenario reports a 40% latency increase specifically from avoidable round-trips).
- **Prompt subagents on goals and quality criteria, not procedures.** "Research the regulatory landscape for X and cite primary sources" adapts better than a rigid numbered checklist, because the subagent can apply judgment to what it discovers.
- **`fork_session`** creates an independent branch from a shared analysis baseline — useful for exploring two divergent approaches (e.g., two refactoring strategies) starting from identical prior context, without one branch's exploration polluting the other.

### 1.4 Enforcement and Handoff Patterns

This is the exam's sharpest recurring distinction: **programmatic enforcement vs. prompt-based guidance.**

| | Prompt-based guidance | Programmatic enforcement (hooks, prerequisite gates) |
|---|---|---|
| Mechanism | System prompt instructions ("always verify identity first") | Code that blocks a tool call unless a precondition is met |
| Compliance | Probabilistic — has a non-zero failure rate | Deterministic — guaranteed |
| Right for | Style, tone, non-critical sequencing preferences | Identity verification before financial operations, refund thresholds, any business rule where a miss has real cost |

Concretely: if `process_refund` should never run before `get_customer` has returned a verified customer ID, don't just *tell* the model to check first — **block the call programmatically** until the prerequisite has actually completed. Prompt instructions alone will occasionally be skipped, and "occasionally" is unacceptable when the failure mode is an incorrect refund to the wrong account. (This is exactly the reasoning behind official Sample Question 1 in `official_scenarios.md`.)

Two more patterns worth internalizing:

- **Decomposing multi-concern requests.** A single customer message can raise several distinct issues (a billing question *and* a shipping complaint). Break it into distinct items, investigate each — in parallel where possible, using shared context — then synthesize one unified resolution, rather than answering only the first concern noticed.
- **Structured handoff summaries.** When escalating to a human agent who has no access to the conversation transcript, compile a structured summary: customer ID, root cause, refund amount (if any), and a recommended action. A prose recap is worse than a structured one because a human skimming a queue needs the same fields every time.

### 1.5 Hooks for Interception and Normalization

Agent SDK hooks let you intercept the loop from *outside* the model's own reasoning — this is the concrete mechanism for turning "programmatic enforcement" (1.4) into working code.

- **`PostToolUse`** intercepts a tool's *result* before the model sees it — the natural place to normalize heterogeneous data. If three different MCP tools return dates as Unix timestamps, ISO 8601, and a numeric status code respectively, a `PostToolUse` hook can normalize all of them into one consistent shape before the model has to reason about them.
- **Tool-call interception hooks** intercept an *outgoing* call before it executes, to enforce a business rule — e.g., blocking any `process_refund` call above $500 and redirecting the flow to human escalation instead of letting the model attempt it.
- **Choose hooks over prompt instructions whenever the business rule requires a guarantee**, not a probability. This is the same principle as 1.4, implemented as actual code rather than left as an architectural preference.

### 1.6 Task Decomposition Strategies

Two decomposition patterns, and the exam wants you to pick the right one per situation, not treat one as universally superior:

| Pattern | When to use | Example |
|---|---|---|
| **Prompt chaining** (fixed sequential pipeline) | The workflow is predictable and multi-aspect | Review each file individually, *then* run one dedicated cross-file integration pass |
| **Dynamic/adaptive decomposition** | The task is open-ended and what's discovered changes what's needed next | "Add comprehensive tests to a legacy codebase": first map structure, identify high-impact areas, *then* build a prioritized plan that adapts as dependencies surface |

A specific, frequently-tested instance: **splitting large code reviews into per-file local-analysis passes plus a separate cross-file integration pass**, rather than one pass over everything. A single pass over many files at once causes *attention dilution* — inconsistent depth per file and even contradictory findings (flagging a pattern as a bug in file A while approving identical code in file B of the same PR). See official Sample Question 12 in `official_scenarios.md` for the canonical version of this failure and fix.

### 1.7 Session State, Resumption, and Forking

| Mechanism | What it does | When to use it |
|---|---|---|
| `--resume <session-name>` | Continues a specific named prior conversation | Prior context is mostly still valid; you're picking up an ongoing investigation |
| `fork_session` | Branches an independent copy from a shared baseline | Comparing two divergent approaches (e.g., two testing strategies) from identical starting context |
| Starting fresh + injected structured summary | New session, but you inject a written recap of what mattered | Prior tool results are now **stale** (e.g., the files have since changed) |

The key judgment call: **resuming with stale tool results is a reliability risk.** If code has changed since a session's earlier exploration, don't just resume and hope the model "remembers correctly" — either inform the resumed session explicitly about what changed (for targeted re-analysis of just the delta) or start a new session with a structured summary rather than relying on old, now-inaccurate tool output baked into history.

---

## Domain 2: Tool Design & MCP Integration (18%)

Every scenario that involves backend integration (Customer Support Resolution Agent, Multi-Agent Research System, Developer Productivity with Claude) leans on this domain. The throughline: **tool descriptions are the primary interface the model uses to make decisions — treat them like an API contract, not a comment.**

### 2.1 Designing Effective Tool Interfaces

Tool descriptions are the *primary mechanism* an LLM uses for tool selection. A minimal description ("Retrieves customer information") gives the model nothing to differentiate that tool from a similarly-named neighbor — this is precisely how `analyze_content` vs. `analyze_document` end up misrouted, or why an agent calls `get_customer` when a user says "check my order #12345" instead of calling `lookup_order`.

A well-written tool description states, at minimum:
- Input formats it accepts
- Example queries that should route to it
- Edge cases and explicit boundaries ("use X when..., not when...")
- How it differs from similarly-named or similarly-scoped tools

Two remediation patterns, in order of effort:

1. **Improve descriptions first.** This is a low-effort, high-leverage first step — before reaching for few-shot examples or a custom routing layer. (Official Sample Question 2 makes exactly this point: description quality beats adding few-shot overhead or building a bespoke routing classifier, when the root cause is that the descriptions themselves are inadequate.)
2. **Split or rename tools when functional overlap is structural**, not just a wording problem — e.g., splitting a generic `analyze_document` into `extract_data_points`, `summarize_content`, and `verify_claim_against_source`, each with a tight, non-overlapping contract; or renaming `analyze_content` to `extract_web_results` so its name alone stops inviting confusion with a document-analysis tool.

Also check your **system prompt** for keyword-sensitive instructions that can override otherwise-good tool descriptions — a system prompt that says "always check content first" can bias tool selection toward a tool with "content" in its name regardless of how well-scoped the tools actually are.

### 2.2 Structured Error Responses for MCP Tools

MCP communicates tool failure via the **`isError` flag**. What you put in that error payload determines whether the agent can make a *good* recovery decision or just fails blind.

| Error category | Retryable? | Example |
|---|---|---|
| Transient | Usually yes | Timeout, service temporarily unavailable |
| Validation | No (fix input first) | Malformed input, missing required field |
| Business rule | No | Policy violation (e.g., refund exceeds threshold) |
| Permission | No | Caller lacks access |

A generic `"Operation failed"` string is a design failure: the agent can't tell a worth-retrying timeout from a request that will *never* succeed no matter how many times it's retried. Structured error metadata should include an `errorCategory`, an `isRetryable` boolean, and a human-readable, customer-safe description (for business-rule errors specifically, so the agent can explain the refusal without regurgitating internal policy IDs).

Two more distinctions worth internalizing:

- **Local recovery vs. propagation.** A subagent should attempt recovery for transient failures itself, and propagate to the coordinator only what it genuinely cannot resolve — along with partial results and what was attempted, so the coordinator isn't recovering blind.
- **Access failure ≠ valid empty result.** A failed connection needing a retry decision is not the same thing as a successful query that legitimately found nothing. Collapsing these into one "no results" signal removes information the coordinator needs.

### 2.3 Distributing Tools and Configuring Tool Choice

**Fewer tools per agent, scoped to role, is a reliability decision, not just a cleanliness preference.** Giving one agent 18 tools instead of 4–5 measurably degrades tool-selection reliability, because every additional plausible tool increases decision complexity at each turn. A synthesis agent given web-search tools "just in case" will occasionally misuse them — attempting searches that are actually the search agent's job, muddying the separation of concerns the whole architecture depends on.

The resolution is **scoped tool access with narrow, deliberate cross-role exceptions**: give each subagent only what its role needs, and where a genuinely high-frequency cross-role need exists (e.g., a synthesis agent needing to fact-check a date 85% of the time via a simple lookup), give it one narrow, purpose-built tool (`verify_fact`) rather than the full toolkit of the specialist role — routing the remaining, rarer, complex cases through the coordinator as before. (This exact tradeoff is official Sample Question 9's scenario.)

**`tool_choice` configuration:**

| Value | Behavior | Use when |
|---|---|---|
| `"auto"` | Model may call a tool *or* return plain text | Default, open-ended reasoning |
| `"any"` | Model must call *some* tool, but chooses which | You need guaranteed tool use but the right tool depends on context (e.g., unknown document type across multiple extraction schemas) |
| `{"type": "tool", "name": "..."}` (forced) | Model must call this specific tool | You need a specific step to run first (e.g., forcing `extract_metadata` before enrichment tools), then handle later steps in follow-up turns |

### 2.4 Integrating MCP Servers

| Scope | File | Use for |
|---|---|---|
| Project | `.mcp.json` | Shared team tooling, checked into version control |
| User | `~/.claude.json` | Personal/experimental servers, not shared with teammates |

- **Environment variable expansion** (e.g., `${GITHUB_TOKEN}`) in `.mcp.json` keeps credentials out of version control while still letting the config file itself be shared.
- **All configured MCP servers' tools are discovered at connection time and available simultaneously** — a project-scoped server and a personally-added experimental one both show up to the agent at once; this is a discovery/availability fact, not a "pick one" choice.
- **MCP resources** expose content catalogs (issue summaries, documentation hierarchies, database schemas) so the agent has visibility into what's available *without* spending exploratory tool calls just to find out what exists.
- **Prefer existing community MCP servers for standard integrations** (e.g., Jira) — reserve custom server development for genuinely team-specific workflows. Building a bespoke Jira integration when a maintained community server already exists is wasted effort with an ongoing maintenance cost.
- **Enhance, don't just accept, third-party MCP tool descriptions.** If an MCP tool's shipped description under-explains its own capability, the agent will under-use it and default back to a built-in tool like `Grep` — even when the MCP tool is strictly more capable for the task. Improving the description is often the fix, not replacing the tool.

### 2.5 Built-in Tools (Read, Write, Edit, Bash, Grep, Glob)

| Tool | Purpose | Selection cue |
|---|---|---|
| `Grep` | Content search | "Find all callers of `X`," "locate this error message" — searching *inside* file contents |
| `Glob` | Path pattern matching | "Find all files matching `**/*.test.tsx`" — searching *file names/paths*, not contents |
| `Read` / `Write` | Whole-file operations | Loading full context, or a full-file rewrite |
| `Edit` | Targeted modification via unique text match | Small, surgical changes with an unambiguous anchor string |

**When `Edit` fails because the anchor text isn't unique in the file, fall back to `Read` + `Write`** — reading the whole file and writing back a modified full version is the reliable path when precise, unique-match editing isn't available.

Build codebase understanding **incrementally**, not by reading everything upfront: start with `Grep` to locate entry points, then `Read` to follow imports and trace flow. For tracing a function's usage through re-export/wrapper modules, first enumerate all the exported names, then `Grep` for each name across the codebase — a single grep for the original name will miss call sites that only ever see the re-exported name.

---

## Domain 3: Claude Code Configuration & Workflows (20%)

Four of the six official scenarios touch this domain (Code Generation with Claude Code, Developer Productivity with Claude, Claude Code for Continuous Integration, and indirectly Developer Productivity's MCP integration). This domain is about **configuration architecture for a team**, not individual command syntax.

### 3.1 CLAUDE.md Hierarchy and Modular Organization

| Level | Location | Scope |
|---|---|---|
| User | `~/.claude/CLAUDE.md` | Applies only to that individual — **not shared with teammates via version control** |
| Project | `.claude/CLAUDE.md` or root `CLAUDE.md` | Shared, version-controlled, applies to everyone working in the repo |
| Directory | Subdirectory `CLAUDE.md` files | Applies to that subtree |

A classic diagnostic scenario: a new team member "isn't following the team's conventions," and the root cause is that the conventions live in *that person's* user-level config from a previous project, not in the project-level file every teammate actually gets. Fixing this means moving the instructions to project scope, not repeating them louder.

- **`@import`** references external files to keep a CLAUDE.md modular — e.g., each package's CLAUDE.md `@import`s only the standards files relevant to that package's maintainers, instead of one monolithic file everyone has to read past.
- **`.claude/rules/`** is the alternative to one giant CLAUDE.md — split into topic-specific files (`testing.md`, `api-conventions.md`, `deployment.md`) for maintainability.
- **`/memory`** verifies which memory files are actually loaded in a session — the diagnostic tool for "why is behavior inconsistent across sessions."

### 3.2 Custom Slash Commands and Skills

| | `.claude/commands/` (project) | `~/.claude/commands/` (user) | `.claude/skills/` |
|---|---|---|---|
| Shared via version control? | Yes | No — personal only | Yes (if project-scoped) |
| Format | Command file | Command file | `SKILL.md` with frontmatter |
| Use for | Team-wide slash commands (e.g., `/review`) | Individual habits/workflows | Task-specific workflows invoked on demand |

**Skill frontmatter options worth knowing cold:**
- `context: fork` — runs the skill in an isolated sub-agent context, so verbose or exploratory output (codebase analysis, brainstorming alternatives) doesn't pollute the main conversation.
- `allowed-tools` — restricts which tools the skill can use while running (e.g., limiting a skill to file-write operations only, to prevent it from taking destructive actions outside its intended scope).
- `argument-hint` — prompts the developer for required parameters when they invoke the skill without arguments.

Personal customization: create a personal variant under `~/.claude/skills/` with a different name, rather than editing the shared project skill, when you want to experiment without affecting teammates.

**Skills vs. CLAUDE.md**: skills are for on-demand, task-specific workflows; CLAUDE.md is for always-loaded, universal standards. If something needs to apply to *every* interaction, it belongs in CLAUDE.md — if it's a workflow you invoke for a specific job, it's a skill.

### 3.3 Path-Specific Rules

`.claude/rules/` files carry **YAML frontmatter with a `paths` field of glob patterns**, so a rule loads only when you're editing a matching file — e.g., `paths: ["terraform/**/*"]` or `paths: ["**/*.test.tsx"]`.

This beats directory-level CLAUDE.md files specifically when a convention applies to files **by type, scattered across many directories** — e.g., every `*.test.tsx` file sitting right next to the component it tests, rather than grouped in one `tests/` folder. A directory-level CLAUDE.md can't reach across the whole tree that way; a glob-pattern rule can.

### 3.4 Plan Mode vs. Direct Execution

| Signal | Plan mode | Direct execution |
|---|---|---|
| Scope | Large-scale, multi-file (e.g., 45+ files), architectural | Single-file, well-understood |
| Ambiguity | Multiple valid approaches; a real design decision | One clear fix, clear stack trace |
| Risk of rework | High if you commit to changes before exploring | Low |
| Example | Monolith → microservices restructuring; choosing between integration approaches with different infra implications | Adding one validation check to one function; a single-file bug fix |

Plan mode exists to let you **explore and design safely before committing to changes** — this is the whole value proposition, and it's why "start with direct execution and let the boundaries emerge as you go" is the wrong call for genuinely architectural work: rework after the fact costs more than planning up front would have. Conversely, plan mode is *overhead*, not safety, on a task where the scope and approach are already unambiguous.

The **`Explore` subagent** is the tool for isolating a verbose discovery phase (e.g., an initial large-codebase survey) so its raw output doesn't consume the main conversation's context — the main session gets a summary back, not the full trace.

Plan mode and direct execution **combine**: plan the investigation and approach, then execute the agreed plan directly.

### 3.5 Iterative Refinement Techniques

| Technique | When it helps |
|---|---|
| 2–3 concrete input/output examples | Prose descriptions of a transformation are being interpreted inconsistently |
| Test-driven iteration (write tests first, iterate against failures) | You want objective, repeatable feedback rather than re-describing the requirement in prose each round |
| The "interview pattern" (have Claude ask clarifying questions before implementing) | Unfamiliar domain, where design considerations (cache invalidation, failure modes) aren't obvious upfront |
| Single detailed message covering all issues | The issues **interact** — fixing one changes the right fix for another |
| Sequential fixes, one at a time | The issues are **independent** |

A specific, testable instance: providing an explicit input/output pair (e.g., "a null value in this field should map to X, not crash the migration") is more reliable than restating "handle nulls gracefully" in different words.

### 3.6 Integrating Claude Code into CI/CD

- **`-p` / `--print`** runs Claude Code non-interactively — without it, a CI job hangs waiting for input it will never receive. This is the fix whenever a pipeline "hangs indefinitely."
- **`--output-format json`** with **`--json-schema`** produces machine-parseable structured findings — necessary for automatically posting inline PR comments rather than dumping unstructured text into a log.
- **CLAUDE.md carries project context into CI** — testing standards, fixture conventions, and review criteria that a CI-invoked Claude Code instance would otherwise have no way to know.
- **Session context isolation matters for review quality.** The same session that generated code is *worse* at reviewing its own changes than an independent instance would be — the generating session retains its own reasoning and is less likely to question decisions it just made. Run code review as a fresh, independent instance, not a continuation of the generation session (this is the CI/CD-specific instance of the general self-review-vs-independent-review principle in Domain 4.6).
- When re-running a review after new commits, **include prior findings in context and instruct Claude to report only new or unaddressed issues** — otherwise every re-run re-reports everything, and developers start ignoring the bot.
- Provide existing test files in context for test-generation tasks, so generated tests don't duplicate scenarios the suite already covers.

---

## Domain 4: Prompt Engineering & Structured Output (20%)

Two scenarios anchor this domain directly (Claude Code for Continuous Integration, Structured Data Extraction), but the techniques generalize across every scenario that needs consistent, machine-usable output.

### 4.1 Explicit Criteria to Reduce False Positives

The central finding: **vague conservatism instructions don't work.** "Be conservative," "only report high-confidence findings," and similar phrasing fail to actually improve precision, because they give the model no concrete boundary to apply — it's still making the same judgment call, just with a hedge word attached.

What *does* work: **explicit, categorical criteria** that state which issues to report and which to skip — e.g., "flag a comment only when the claimed behavior contradicts the code's actual behavior" (specific, checkable) instead of "check that comments are accurate" (a restatement of the goal, not a criterion). Define severity levels the same way: concrete code examples per severity level, not adjectives.

Practical remediation when a category has a high false-positive rate and is actively eroding developer trust: **temporarily disable that category** while you rework its criteria, rather than letting it keep burning trust in the categories that *are* accurate — high false-positive rates in one category measurably reduce confidence in the tool's correct findings elsewhere.

### 4.2 Few-Shot Prompting for Consistency

Few-shot examples are the most effective lever when **detailed prose instructions alone produce inconsistent output** — 2–4 targeted examples, chosen for ambiguous or edge-case scenarios, showing not just the desired output format but *why* one action was correct over a plausible alternative (the reasoning, not just the label).

Where this matters most:
- **Ambiguous-case handling** — e.g., which tool to select for a genuinely ambiguous request, or how to judge branch-level test coverage gaps.
- **Consistent output format** — demonstrating the exact fields expected (location, issue, severity, suggested fix) so every output has the same shape.
- **Reducing hallucination in extraction** — showing correct handling of informal measurements or varied document structures (inline citations vs. a bibliography, a methodology section vs. details embedded inline) teaches the model to generalize the *pattern*, not just match the literal examples given.

The generalization claim is worth being precise about: good few-shot examples let the model apply the *same underlying judgment* to novel patterns it wasn't shown — that's what separates few-shot prompting from simple pattern-matching against a fixed example set.

### 4.3 Structured Output via Tool Use and JSON Schemas

**`tool_use` with a JSON schema is the most reliable path to guaranteed schema-compliant output** — it eliminates JSON *syntax* errors entirely, because the model is constrained to the schema at the API level rather than asked to produce well-formed JSON as free text.

| `tool_choice` | Guarantee |
|---|---|
| `"auto"` | Model may return text instead of calling a tool — **no** structured-output guarantee |
| `"any"` | Model must call *a* tool — use when multiple extraction schemas exist and you don't yet know the document type |
| Forced (`{"type": "tool", "name": "extract_metadata"}`) | Model must call *this* tool — use to sequence a specific extraction before enrichment steps |

**What tool-use schemas do *not* fix: semantic correctness.** A syntactically valid, schema-compliant JSON object can still have line items that don't sum to the stated total, or a value placed in the wrong field. Schema compliance and semantic correctness are two separate problems — the first is solved by `tool_use`; the second needs validation logic (4.4).

Schema design choices that matter in practice:
- **Optional/nullable fields** for information source documents may simply not contain — a required field the model can't find will get a *fabricated* value to satisfy the schema, which is worse than a null.
- **`enum` + "other" + detail-string pattern** for extensible categories, so novel categories aren't forced into an ill-fitting existing enum value; add `"unclear"` explicitly for genuinely ambiguous cases rather than making the model guess.
- **Format normalization rules stated in the prompt**, alongside the strict schema, to handle inconsistent source formatting the schema alone can't paper over.

### 4.4 Validation, Retry, and Feedback Loops

**Retry-with-error-feedback**: on a validation failure, the follow-up request should include the original document, the failed extraction, *and* the specific validation error — not just "try again." This gives the model something concrete to correct against.

**Know when retry will and won't help:**

| Failure type | Will retry help? |
|---|---|
| Format/structural mismatch (wrong shape, malformed value) | Yes — the model can self-correct given the specific error |
| Information is simply absent from the provided source | No — no amount of retrying manufactures data that was never in the document |

Two structural design patterns for catching semantic errors specifically:
- **Self-correction validation flows** — e.g., extract both `calculated_total` and `stated_total` so a discrepancy is *detectable*, and add a `conflict_detected` boolean when source data itself is internally inconsistent.
- **`detected_pattern` fields on findings** — tagging *which* code construct triggered a given finding turns "developers keep dismissing this category" from an anecdote into an analyzable pattern you can actually fix.

### 4.5 Batch Processing Strategy

| | Synchronous (real-time) API | Message Batches API |
|---|---|---|
| Cost | Full price | ~50% savings |
| Latency | Immediate, request-scoped | Up to 24-hour processing window, **no guaranteed SLA** |
| Blocking workflows (pre-merge checks) | Correct fit | **Wrong fit** — no latency guarantee means it can't back a blocking gate |
| Latency-tolerant workloads (overnight reports, weekly audits) | Unnecessarily expensive | Correct fit |
| Multi-turn tool calling mid-request | Supported | **Not supported** — a batch request can't execute a tool mid-flight and continue reasoning on the result |
| Correlating requests to responses | N/A (synchronous) | `custom_id` field |

The exam explicitly tests **matching the API to the workflow's actual latency tolerance**, not just chasing the 50% discount everywhere: a blocking pre-merge check that developers wait on cannot be moved to batch just because it's cheaper, because "up to 24 hours, no SLA" would mean merges sometimes silently hang overnight. (See official Sample Question 11 in `official_scenarios.md`.)

Operational details worth knowing: calculate submission frequency against your SLA (e.g., 4-hour submission windows to guarantee a 30-hour SLA against a 24-hour worst-case batch window); on partial failure, resubmit only the failed documents (identified by `custom_id`) with whatever fix is needed (e.g., chunking documents that exceeded context limits) rather than resubmitting the whole batch; and refine your prompt against a small sample *before* submitting a large batch, since a systematic prompt problem discovered only after a 10,000-document batch is a very expensive way to learn it.

### 4.6 Multi-Instance and Multi-Pass Review Architectures

**Self-review is structurally weaker than independent review**, because a model retains the reasoning context from generating something and is measurably less likely to question its own prior decisions within the same session — this holds even with explicit "please double check your work" instructions or extended thinking; the fix is architectural (a second, independent instance without that reasoning history), not a better self-review prompt.

**Multi-pass review for large changesets**: split into focused per-file passes for local issues, plus one dedicated cross-file integration pass for things like data-flow consistency across files — this is the same attention-dilution fix as Domain 1.6, applied specifically to review architecture. A single pass over many files at once measurably degrades depth and consistency.

A useful refinement: have the model self-report a confidence score alongside each finding during a verification pass, so downstream routing (auto-accept vs. route to a human) can be calibrated rather than uniform.

---

## Domain 5: Context Management & Reliability (15%)

The smallest domain by weight, but it's the connective tissue across every scenario — every one of the 6 official scenarios lists Context Management & Reliability or its close relative as a primary domain except Code Generation with Claude Code (which still relies on it implicitly). The throughline: **long-running agentic systems degrade in specific, predictable ways, and the fixes are structural, not "just add more context."**

### 5.1 Preserving Critical Information Across Long Interactions

Two decay mechanisms to know cold:

- **Progressive summarization risk**: repeatedly condensing a conversation tends to blur exactly the details that matter most operationally — numeric values, percentages, dates, and customer-stated expectations get smoothed into vague paraphrase. The fix is a **persistent "case facts" block** — extracted transactional facts (amounts, dates, order numbers, statuses) carried in every prompt *outside* the summarized history, so they can't be lossy-compressed away.
- **"Lost in the middle"**: models reliably attend to the beginning and end of a long input, but can miss findings buried in the middle. The mitigating pattern is **position-aware structuring** — put key-findings summaries at the *start* of an aggregated input, and use explicit section headers for detailed results, rather than one long undifferentiated blob.

Also watch token cost disproportionate to relevance: a verbose tool result (e.g., 40+ fields from an order lookup when only 5 matter for the task at hand) should be **trimmed to relevant fields before it accumulates in context** — otherwise a handful of genuinely useful facts get buried in noise that also costs real tokens across every subsequent turn.

For multi-agent systems specifically: require subagents to emit **structured outputs with explicit metadata** (source, date, methodological context) rather than verbose prose, whenever the downstream agent has a limited context budget — the downstream agent needs facts and citations, not the upstream agent's full reasoning trace.

### 5.2 Escalation and Ambiguity Resolution

| Trigger | Correct response |
|---|---|
| Customer explicitly asks for a human | Escalate **immediately** — do not attempt investigation first |
| Straightforward issue, customer hasn't asked for a human | Acknowledge and offer to resolve; escalate only if they reiterate the preference |
| Policy is silent or ambiguous on the specific request (e.g., competitor price-matching when policy only covers own-site adjustments) | Escalate — this is a policy *gap*, not a "hard case" |
| Multiple customer records match the query | Ask for an additional identifier — **do not** heuristically pick one |

Two commonly-tempting but wrong signals: **sentiment-based escalation** (frustration doesn't correlate with actual case complexity — a calm customer can have a genuinely policy-ambiguous case, and a frustrated one can have a trivially resolvable issue) and **self-reported confidence scores** (an agent that's wrong about a hard case is often *also* wrong about how confident it should be — miscalibration doesn't announce itself). Both fail for the same underlying reason: they're proxies for complexity that don't actually track complexity. The reliable fix is **explicit escalation criteria with few-shot examples** showing the boundary between "resolve autonomously" and "escalate" (see official Sample Question 3 in `official_scenarios.md`).

### 5.3 Error Propagation in Multi-Agent Systems

Three anti-patterns to recognize immediately as wrong answers:
1. **Generic error status** ("search unavailable") — hides context the coordinator needs to make a real recovery decision.
2. **Silently suppressing errors** (returning an empty result marked as success) — masks a real failure as a legitimate "no matches," corrupting downstream synthesis.
3. **Terminating the entire workflow on a single subagent failure** — throws away everything that *did* succeed over one component's transient failure.

The correct pattern: **structured error context** — failure type, what was actually attempted, any partial results obtained, and potential alternative approaches — propagated only for what a subagent genuinely could not resolve locally (see 2.2 and 1.4/1.5 for the same principle at the tool and hook layers). The coordinator can then proceed with partial results and explicitly annotate the final output with coverage gaps, rather than either pretending nothing went wrong or discarding a partially-successful research pass.

### 5.4 Context Management in Large Codebase Exploration

Extended exploration sessions degrade in a specific, recognizable way: the model starts giving inconsistent answers and referencing "typical patterns" in the abstract rather than the specific classes it actually discovered earlier in the same session — a sign context has degraded past the point of reliable recall.

Countermeasures:
- **Scratchpad files** that persist key findings across context boundaries — the agent writes down what it found, and references the file for later questions instead of relying on in-context memory of a now-distant discovery.
- **Subagent delegation for verbose discovery** — spawn a subagent to investigate a specific question ("find all test files," "trace refund-flow dependencies") while the main agent stays at the coordination level, receiving a summary rather than the full trace.
- **Structured state exports for crash recovery** — each agent exports its state to a known location; on resume, the coordinator loads a manifest and injects the relevant state back into agent prompts, instead of restarting exploration from zero.
- **`/compact`** reduces context usage mid-session when it's filled with verbose discovery output that's no longer needed at full fidelity.

### 5.5 Human Review Workflows and Confidence Calibration

The single most important caution here: **an aggregate accuracy number can hide a real problem.** 97% overall accuracy can still mean one document type or one field is performing badly, simply outweighed in the average by everything else performing well. Validate **by document type and by field**, not only in aggregate, before you reduce human review based on an overall number.

Practical design:
- **Stratified random sampling** of *high-confidence* extractions specifically — not just low-confidence ones — because this is how you catch novel error patterns and measure the true error rate of the cases you'd otherwise stop checking entirely.
- **Field-level confidence scores**, calibrated against a labeled validation set, to route review attention where it's actually needed rather than uniformly.
- Route to human review when confidence is low **or** the source document itself is ambiguous/contradictory — the second condition matters independently of model confidence, because a confidently-wrong extraction from contradictory source data is still wrong.

### 5.6 Provenance and Uncertainty in Multi-Source Synthesis

Source attribution is lost the same way progressive summarization loses numeric detail (5.1): compressing findings without preserving a **claim → source mapping** is how a synthesis report ends up with unattributed assertions nobody can trace back to a document.

The fix is structural, not a synthesis-prompt instruction: require every subagent to output **structured claim-source mappings** (source URL/document name, the relevant excerpt, and — critically — a publication or data-collection date) that the synthesis step is required to preserve and merge, rather than paraphrase away.

For genuinely **conflicting statistics from credible sources**: annotate the conflict with both values and their sources, rather than arbitrarily picking one — the report should have an explicit section distinguishing well-established findings from contested ones. Dates matter specifically because a temporal difference (an older stat vs. a newer one) can otherwise be misread as a contradiction when it's actually just two snapshots in time. Finally, match the rendering to the content type in the final report — financial data as a table, news as prose, technical findings as structured lists — rather than flattening every content type into the same uniform shape for convenience.

---

## Appendix: In-Scope and Out-of-Scope Topics (from the Exam Guide)

Reproduced accurately from the official exam guide's appendix (Section 17). Anything not in the in-scope list, or explicitly listed as out-of-scope, should not be assumed testable.

### Technologies and Concepts That May Appear

- **Claude Agent SDK** — agent definitions, agentic loops, `stop_reason` handling, hooks (`PostToolUse`, tool call interception), subagent spawning via the `Task` tool, `allowedTools` configuration
- **Model Context Protocol (MCP)** — MCP servers, MCP tools, MCP resources, `isError` flag, tool descriptions, tool distribution, `.mcp.json` configuration, environment variable expansion
- **Claude Code** — CLAUDE.md configuration hierarchy (user/project/directory), `.claude/rules/` with YAML frontmatter path-scoping, `.claude/commands/` for slash commands, `.claude/skills/` with SKILL.md frontmatter (`context: fork`, `allowed-tools`, `argument-hint`), plan mode, direct execution, `/memory` command, `/compact`, `--resume`, `fork_session`, Explore subagent
- **Claude Code CLI** — `-p` / `--print` flag for non-interactive mode, `--output-format json`, `--json-schema` for structured CI output
- **Claude API** — `tool_use` with JSON schemas, `tool_choice` options (`"auto"`, `"any"`, forced tool selection), `stop_reason` values (`"tool_use"`, `"end_turn"`), `max_tokens`, system prompts
- **Message Batches API** — 50% cost savings, up to 24-hour processing window, `custom_id` for request/response correlation, polling for completion, no multi-turn tool calling support
- **JSON Schema** — required vs. optional fields, enum types, nullable fields, "other" + detail string patterns, strict mode for syntax error elimination
- **Pydantic** — schema validation, semantic validation errors, validation-retry loops
- **Built-in tools** — Read, Write, Edit, Bash, Grep, Glob — their purposes and selection criteria
- **Few-shot prompting** — targeted examples for ambiguous scenarios, format demonstration, generalization to novel patterns
- **Prompt chaining** — sequential task decomposition into focused passes
- **Context window management** — token budgets, progressive summarization, lost-in-the-middle effects, context extraction, scratchpad files
- **Session management** — session resumption, `fork_session`, named sessions, session context isolation
- **Confidence scoring** — field-level confidence, calibration with labeled validation sets, stratified sampling for error rate measurement

### In-Scope Topics (Explicitly Tested)

- Agentic loop implementation: control flow based on `stop_reason`, tool result handling, loop termination conditions
- Multi-agent orchestration: coordinator-subagent patterns, task decomposition, parallel subagent execution, iterative refinement loops
- Subagent context management: explicit context passing, structured state persistence, crash recovery using manifests
- Tool interface design: writing effective tool descriptions, splitting vs. consolidating tools, tool naming to reduce ambiguity
- MCP tool and resource design: resources for content catalogs, tools for actions, description quality for adoption
- MCP server configuration: project vs. user scope, environment variable expansion, multi-server simultaneous access
- Error handling and propagation: structured error responses, transient vs. business vs. permission errors, local recovery before escalation
- Escalation decision-making: explicit criteria, honoring customer preferences, policy gap identification
- CLAUDE.md configuration: hierarchy (user/project/directory), `@import` patterns, `.claude/rules/` with glob patterns
- Custom commands and skills: project vs. user scope, `context: fork`, `allowed-tools`, `argument-hint` frontmatter
- Plan mode vs. direct execution: complexity assessment, architectural decisions, single-file changes
- Iterative refinement: input/output examples, test-driven iteration, interview pattern, sequential vs. parallel issue resolution
- Structured output via `tool_use`: schema design, `tool_choice` configuration, nullable fields to prevent hallucination
- Few-shot prompting: ambiguous scenario targeting, format consistency, false positive reduction
- Batch processing: Message Batches API appropriateness, latency tolerance assessment, failure handling by `custom_id`
- Context window optimization: trimming verbose tool outputs, structured fact extraction, position-aware input ordering
- Human review workflows: confidence calibration, stratified sampling, accuracy segmentation by document type and field
- Information provenance: claim-source mappings, temporal data handling, conflict annotation, coverage gap reporting

### Out-of-Scope Topics (Will NOT Appear on the Exam)

- Fine-tuning Claude models or training custom models
- Claude API authentication, billing, or account management
- Detailed implementation of specific programming languages or frameworks (beyond what's needed for tool and schema configuration)
- Deploying or hosting MCP servers (infrastructure, networking, container orchestration)
- Claude's internal architecture, training process, or model weights
- Constitutional AI, RLHF, or safety training methodologies
- Embedding models or vector database implementation details
- Computer use (browser automation, desktop interaction)
- Vision/image analysis capabilities
- Streaming API implementation or server-sent events
- Rate limiting, quotas, or API pricing calculations
- OAuth, API key rotation, or authentication protocol details
- Specific cloud provider configurations (AWS, GCP, Azure)
- Performance benchmarking or model comparison metrics
- Prompt caching implementation details (beyond knowing it exists)
- Token counting algorithms or tokenization specifics

---

## Where to Go Next in This Kit

- **`official_scenarios.md`** — all 6 named scenarios in full, with their sample questions and rationale. Read this next; 4 of these 6 are the actual exam.
- **`official_samples.md`** — cross-reference of every official sample question and where it lives.
- **`practice_questions.md`** — 30 original scenario-anchored practice questions across all 5 domains, weighted by blueprint percentage, with an answer key and rationale.
- **`scenario_qa.md`** — 12 additional original architecture scenarios covering decisions the official 6 don't exercise (PII in RAG, fine-tuning vs. prompting, multi-region failover, and more).
