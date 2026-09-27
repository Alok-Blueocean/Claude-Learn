# Official Exam Scenarios (CCAR-F)

This is the single most important file in this study kit. The CCAR-F exam presents **60 items framed by 4 scenarios, drawn at random from the fixed bank of 6 scenarios below** (per the official exam guide, Section 5). All 6 scenario descriptions, their primary domains, and every official sample question tied to them are reproduced in full here, exactly as published in the exam guide (`CCAR-F_Architect_Foundations_Exam_Guide.pdf`).

Because only 4 of 6 appear on any given sitting, you should be fluent in **all 6** — you don't get to pick which four you're tested on.

---

## Scenario 1: Customer Support Resolution Agent

> You are building a customer support resolution agent using the Claude Agent SDK. The agent handles high-ambiguity requests like returns, billing disputes, and account issues. It has access to your backend systems through custom Model Context Protocol (MCP) tools (`get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`). Your target is 80%+ first-contact resolution while knowing when to escalate.

**Primary domains:** Agentic Architecture & Orchestration, Tool Design & MCP Integration, Context Management & Reliability

**What this scenario tends to probe:** programmatic enforcement vs. prompt-based compliance for financial operations (Domain 1, Task Statement 1.4/1.5), tool description quality to prevent misrouting between similar tools (Domain 2, Task Statement 2.1), and escalation calibration — knowing when to hand off vs. resolve autonomously (Domain 5, Task Statement 5.2).

### Question 1

Production data shows that in 12% of cases, your agent skips `get_customer` entirely and calls `lookup_order` using only the customer's stated name, occasionally leading to misidentified accounts and incorrect refunds. What change would most effectively address this reliability issue?

**A.** Add a programmatic prerequisite that blocks `lookup_order` and `process_refund` calls until `get_customer` has returned a verified customer ID.
**B.** Enhance the system prompt to state that customer verification via `get_customer` is mandatory before any order operations.
**C.** Add few-shot examples showing the agent always calling `get_customer` first, even when customers volunteer order details.
**D.** Implement a routing classifier that analyzes each request and enables only the subset of tools appropriate for that request type.

**Correct Answer: A.** When a specific tool sequence is required for critical business logic (like verifying customer identity before processing refunds), programmatic enforcement provides deterministic guarantees that prompt-based approaches cannot. Options B and C rely on probabilistic LLM compliance, which is insufficient when errors have financial consequences. Option D addresses tool availability rather than tool ordering, which is not the actual problem.

### Question 2

Production logs show the agent frequently calls `get_customer` when users ask about orders (e.g., "check my order #12345"), instead of calling `lookup_order`. Both tools have minimal descriptions ("Retrieves customer information" / "Retrieves order details") and accept similar identifier formats. What's the most effective first step to improve tool selection reliability?

**A.** Add few-shot examples to the system prompt demonstrating correct tool selection patterns, with 5-8 examples showing order-related queries routing to `lookup_order`.
**B.** Expand each tool's description to include input formats it handles, example queries, edge cases, and boundaries explaining when to use it versus similar tools.
**C.** Implement a routing layer that parses user input before each turn and pre-selects the appropriate tool based on detected keywords and identifier patterns.
**D.** Consolidate both tools into a single `lookup_entity` tool that accepts any identifier and internally determines which backend to query.

**Correct Answer: B.** Tool descriptions are the primary mechanism LLMs use for tool selection. When descriptions are minimal, models lack the context to differentiate between similar tools. Option B directly addresses this root cause with a low-effort, high-leverage fix. Few-shot examples (A) add token overhead without fixing the underlying issue. A routing layer (C) is over-engineered and bypasses the LLM's natural language understanding. Consolidating tools (D) is a valid architectural choice but requires more effort than a "first step" warrants when the immediate problem is inadequate descriptions.

### Question 3

Your agent achieves 55% first-contact resolution, well below the 80% target. Logs show it escalates straightforward cases (standard damage replacements with photo evidence) while attempting to autonomously handle complex situations requiring policy exceptions. What's the most effective way to improve escalation calibration?

**A.** Add explicit escalation criteria to your system prompt with few-shot examples demonstrating when to escalate versus resolve autonomously.
**B.** Have the agent self-report a confidence score (1-10) before each response and automatically route requests to humans when confidence falls below a threshold.
**C.** Deploy a separate classifier model trained on historical tickets to predict which requests need escalation before the main agent begins processing.
**D.** Implement sentiment analysis to detect customer frustration levels and automatically escalate when negative sentiment exceeds a threshold.

**Correct Answer: A.** Adding explicit escalation criteria with few-shot examples directly addresses the root cause: unclear decision boundaries. This is the proportionate first response before adding infrastructure. Option B fails because LLM self-reported confidence is poorly calibrated — the agent is already incorrectly confident on hard cases. Option C is over-engineered, requiring labeled data and ML infrastructure when prompt optimization hasn't been tried. Option D solves a different problem entirely; sentiment doesn't correlate with case complexity, which is the actual issue.

---

## Scenario 2: Code Generation with Claude Code

> You are using Claude Code to accelerate software development. Your team uses it for code generation, refactoring, debugging, and documentation. You need to integrate it into your development workflow with custom slash commands, CLAUDE.md configurations, and understand when to use plan mode vs direct execution.

**Primary domains:** Claude Code Configuration & Workflows, Context Management & Reliability

**What this scenario tends to probe:** correct scoping of slash commands and configuration files (project vs. user), plan mode vs. direct execution judgment on tasks of varying architectural weight, and path-specific rule design for codebases with heterogeneous conventions (Domain 3, Task Statements 3.1–3.4).

### Question 4

You want to create a custom `/review` slash command that runs your team's standard code review checklist. This command should be available to every developer when they clone or pull the repository. Where should you create this command file?

**A.** In the `.claude/commands/` directory in the project repository
**B.** In `~/.claude/commands/` in each developer's home directory
**C.** In the `CLAUDE.md` file at the project root
**D.** In a `.claude/config.json` file with a commands array

**Correct Answer: A.** Project-scoped custom slash commands should be stored in the `.claude/commands/` directory within the repository. These commands are version-controlled and automatically available to all developers when they clone or pull the repo. Option B (`~/.claude/commands/`) is for personal commands that aren't shared via version control. Option C (`CLAUDE.md`) is for project instructions and context, not command definitions. Option D describes a configuration mechanism that doesn't exist in Claude Code.

### Question 5

You've been assigned to restructure the team's monolithic application into microservices. This will involve changes across dozens of files and requires decisions about service boundaries and module dependencies. Which approach should you take?

**A.** Enter plan mode to explore the codebase, understand dependencies, and design an implementation approach before making changes.
**B.** Start with direct execution and make changes incrementally, letting the implementation reveal the natural service boundaries.
**C.** Use direct execution with comprehensive upfront instructions detailing exactly how each service should be structured.
**D.** Begin in direct execution mode and only switch to plan mode if you encounter unexpected complexity during implementation.

**Correct Answer: A.** Plan mode is designed for complex tasks involving large-scale changes, multiple valid approaches, and architectural decisions — exactly what monolith-to-microservices restructuring requires. It enables safe codebase exploration and design before committing to changes. Option B risks costly rework when dependencies are discovered late. Option C assumes you already know the right structure without exploring the code. Option D ignores that the complexity is already stated in the requirements, not something that might emerge later.

### Question 6

Your codebase has distinct areas with different coding conventions: React components use functional style with hooks, API handlers use async/await with specific error handling, and database models follow a repository pattern. Test files are spread throughout the codebase alongside the code they test (e.g., `Button.test.tsx` next to `Button.tsx`), and you want all tests to follow the same conventions regardless of location. What's the most maintainable way to ensure Claude automatically applies the correct conventions when generating code?

**A.** Create rule files in `.claude/rules/` with YAML frontmatter specifying glob patterns to conditionally apply conventions based on file paths
**B.** Consolidate all conventions in the root `CLAUDE.md` file under headers for each area, relying on Claude to infer which section applies
**C.** Create skills in `.claude/skills/` for each code type that include the relevant conventions in their SKILL.md files
**D.** Place a separate CLAUDE.md file in each subdirectory containing that area's specific conventions

**Correct Answer: A.** Option A is correct because `.claude/rules/` with glob patterns (e.g., `**/*.test.tsx`) allows conventions to be automatically applied based on file paths regardless of directory location, essential for test files spread throughout the codebase. Option B relies on inference rather than explicit matching, making it unreliable. Option C requires manual skill invocation or relies on Claude choosing to load them, contradicting the need for deterministic "automatic" application based on file paths. Option D can't easily handle files spread across many directories since CLAUDE.md files are directory-bound.

---

## Scenario 3: Multi-Agent Research System

> You are building a multi-agent research system using the Claude Agent SDK. A coordinator agent delegates to specialized subagents: one searches the web, one analyzes documents, one synthesizes findings, and one generates reports. The system researches topics and produces comprehensive, cited reports.

**Primary domains:** Agentic Architecture & Orchestration, Tool Design & MCP Integration, Context Management & Reliability

**What this scenario tends to probe:** diagnosing coordinator-level decomposition failures vs. subagent-level failures, structured error propagation for intelligent coordinator recovery, and scoped cross-role tool provisioning to reduce round-trip latency (Domain 1, Task Statements 1.2–1.3; Domain 5, Task Statement 5.3).

### Question 7

After running the system on the topic "impact of AI on creative industries," you observe that each subagent completes successfully: the web search agent finds relevant articles, the document analysis agent summarizes papers correctly, and the synthesis agent produces coherent output. However, the final reports cover only visual arts, completely missing music, writing, and film production. When you examine the coordinator's logs, you see it decomposed the topic into three subtasks: "AI in digital art creation," "AI in graphic design," and "AI in photography." What is the most likely root cause?

**A.** The synthesis agent lacks instructions for identifying coverage gaps in the findings it receives from other agents.
**B.** The coordinator agent's task decomposition is too narrow, resulting in subagent assignments that don't cover all relevant domains of the topic.
**C.** The web search agent's queries are not comprehensive enough and need to be expanded to cover more creative industry sectors.
**D.** The document analysis agent is filtering out sources related to non-visual creative industries due to overly restrictive relevance criteria.

**Correct Answer: B.** The coordinator's logs reveal the root cause directly: it decomposed "creative industries" into only visual arts subtasks (digital art, graphic design, photography), completely omitting music, writing, and film. The subagents executed their assigned tasks correctly — the problem is what they were assigned. Options A, C, and D incorrectly blame downstream agents that are working correctly within their assigned scope.

### Question 8

The web search subagent times out while researching a complex topic. You need to design how this failure information flows back to the coordinator agent. Which error propagation approach best enables intelligent recovery?

**A.** Return structured error context to the coordinator including the failure type, the attempted query, any partial results, and potential alternative approaches.
**B.** Implement automatic retry logic with exponential backoff within the subagent, returning a generic "search unavailable" status only after all retries are exhausted.
**C.** Catch the timeout within the subagent and return an empty result set marked as successful.
**D.** Propagate the timeout exception directly to a top-level handler that terminates the entire research workflow.

**Correct Answer: A.** Structured error context gives the coordinator the information it needs to make intelligent recovery decisions — whether to retry with a modified query, try an alternative approach, or proceed with partial results. Option B's generic status hides valuable context from the coordinator, preventing informed decisions. Option C suppresses the error by marking failure as success, which prevents any recovery and risks incomplete research outputs. Option D terminates the entire workflow unnecessarily when recovery strategies could succeed.

### Question 9

During testing, you observe that the synthesis agent frequently needs to verify specific claims while combining findings. Currently, when verification is needed, the synthesis agent returns control to the coordinator, which invokes the web search agent, then re-invokes synthesis with results. This adds 2-3 round trips per task and increases latency by 40%. Your evaluation shows that 85% of these verifications are simple fact-checks (dates, names, statistics) while 15% require deeper investigation. What's the most effective approach to reduce overhead while maintaining system reliability?

**A.** Give the synthesis agent a scoped `verify_fact` tool for simple lookups, while complex verifications continue delegating to the web search agent through the coordinator.
**B.** Have the synthesis agent accumulate all verification needs and return them as a batch to the coordinator at the end of its pass, which then sends them all to the web search agent at once.
**C.** Give the synthesis agent access to all web search tools so it can handle any verification need directly without round-trips through the coordinator.
**D.** Have the web search agent proactively cache extra context around each source during initial research, anticipating what the synthesis agent might need to verify.

**Correct Answer: A.** Option A applies the principle of least privilege by giving the synthesis agent only what it needs for the 85% common case (simple fact verification) while preserving the existing coordination pattern for complex cases. Option B's batching approach creates blocking dependencies since synthesis steps may depend on earlier verified facts. Option C over-provisions the synthesis agent, violating separation of concerns. Option D relies on speculative caching that cannot reliably predict what the synthesis agent will need to verify.

---

## Scenario 4: Developer Productivity with Claude

> You are building developer productivity tools using the Claude Agent SDK. The agent helps engineers explore unfamiliar codebases, understand legacy systems, generate boilerplate code, and automate repetitive tasks. It uses the built-in tools (Read, Write, Bash, Grep, Glob) and integrates with Model Context Protocol (MCP) servers.

**Primary domains:** Tool Design & MCP Integration, Claude Code Configuration & Workflows, Agentic Architecture & Orchestration

**What this scenario tends to probe:** correct selection among built-in tools (Grep vs. Glob, Read/Write vs. Edit), incremental codebase-understanding strategy vs. reading everything upfront, MCP server integration alongside built-in tools, and using subagent delegation plus scratchpad files to manage context in long exploration sessions (Domain 2, Task Statement 2.5; Domain 5, Task Statement 5.4).

**Note on official sample questions:** the published exam guide does not include worked sample questions specifically tied to this scenario (its 12 published samples cover Scenarios 1, 2, 3, and 5 only). The knowledge and skills this scenario draws on — built-in tool selection, incremental exploration, MCP server scoping, scratchpad-based context management — are covered in depth in `tutorial.md` (Domain 2, Task Statement 2.5 and Domain 5, Task Statement 5.4), and exercised with original questions in `practice_questions.md` and `scenario_qa.md`.

---

## Scenario 5: Claude Code for Continuous Integration

> You are integrating Claude Code into your Continuous Integration/Continuous Deployment (CI/CD) pipeline. The system runs automated code reviews, generates test cases, and provides feedback on pull requests. You need to design prompts that provide actionable feedback and minimize false positives.

**Primary domains:** Claude Code Configuration & Workflows, Prompt Engineering & Structured Output

**What this scenario tends to probe:** running Claude Code non-interactively in a pipeline, matching synchronous vs. batch API to workflow latency requirements, and multi-pass review architecture to avoid attention dilution across large, multi-file PRs (Domain 3, Task Statement 3.6; Domain 4, Task Statements 4.5–4.6).

### Question 10

Your pipeline script runs `claude "Analyze this pull request for security issues"` but the job hangs indefinitely. Logs indicate Claude Code is waiting for interactive input. What's the correct approach to run Claude Code in an automated pipeline?

**A.** Add the `-p` flag: `claude -p "Analyze this pull request for security issues"`
**B.** Set the environment variable `CLAUDE_HEADLESS=true` before running the command
**C.** Redirect stdin from `/dev/null`: `claude "Analyze this pull request for security issues" < /dev/null`
**D.** Add the `--batch` flag: `claude --batch "Analyze this pull request for security issues"`

**Correct Answer: A.** The `-p` (or `--print`) flag is the documented way to run Claude Code in non-interactive mode. It processes the prompt, outputs the result to stdout, and exits without waiting for user input — exactly what CI/CD pipelines require. The other options reference non-existent features (`CLAUDE_HEADLESS` environment variable, `--batch` flag) or use Unix workarounds that don't properly address Claude Code's command syntax.

### Question 11

Your team wants to reduce API costs for automated analysis. Currently, real-time Claude calls power two workflows: (1) a blocking pre-merge check that must complete before developers can merge, and (2) a technical debt report generated overnight for review the next morning. Your manager proposes switching both to the Message Batches API for its 50% cost savings. How should you evaluate this proposal?

**A.** Use batch processing for the technical debt reports only; keep real-time calls for pre-merge checks.
**B.** Switch both workflows to batch processing with status polling to check for completion.
**C.** Keep real-time calls for both workflows to avoid batch result ordering issues.
**D.** Switch both to batch processing with a timeout fallback to real-time if batches take too long.

**Correct Answer: A.** The Message Batches API offers 50% cost savings but has processing times up to 24 hours with no guaranteed latency SLA. This makes it unsuitable for blocking pre-merge checks where developers wait for results, but ideal for overnight batch jobs like technical debt reports. Option B is wrong because relying on "often faster" completion isn't acceptable for blocking workflows. Option C reflects a misconception — batch results can be correlated using `custom_id` fields. Option D adds unnecessary complexity when the simpler solution is matching each API to its appropriate use case.

### Question 12

A pull request modifies 14 files across the stock tracking module. Your single-pass review analyzing all files together produces inconsistent results: detailed feedback for some files but superficial comments for others, obvious bugs missed, and contradictory feedback — flagging a pattern as problematic in one file while approving identical code elsewhere in the same PR. How should you restructure the review?

**A.** Split into focused passes: analyze each file individually for local issues, then run a separate integration-focused pass examining cross-file data flow.
**B.** Require developers to split large PRs into smaller submissions of 3-4 files before the automated review runs.
**C.** Switch to a higher-tier model with a larger context window to give all 14 files adequate attention in one pass.
**D.** Run three independent review passes on the full PR and only flag issues that appear in at least two of the three runs.

**Correct Answer: A.** Splitting reviews into focused passes directly addresses the root cause: attention dilution when processing many files at once. File-by-file analysis ensures consistent depth, while a separate integration pass catches cross-file issues. Option B shifts burden to developers without improving the system. Option C misunderstands that larger context windows don't solve attention quality issues. Option D would actually suppress detection of real bugs by requiring consensus on issues that may only be caught intermittently.

---

## Scenario 6: Structured Data Extraction

> You are building a structured data extraction system using Claude. The system extracts information from unstructured documents, validates the output using JavaScript Object Notation (JSON) schemas, and maintains high accuracy. It must handle edge cases gracefully and integrate with downstream systems.

**Primary domains:** Prompt Engineering & Structured Output, Context Management & Reliability

**What this scenario tends to probe:** `tool_use`/JSON schema design (required vs. optional/nullable fields, enum + "other" patterns), validation-retry loop design and knowing when retry can't succeed, and human review routing via field-level confidence calibration and stratified sampling (Domain 4, Task Statements 4.3–4.5; Domain 5, Task Statement 5.5).

**Note on official sample questions:** the published exam guide does not include worked sample questions specifically tied to this scenario (its 12 published samples cover Scenarios 1, 2, 3, and 5 only). The knowledge and skills this scenario draws on — schema design with nullable fields, retry-with-error-feedback loops, batch processing strategy, and confidence-calibrated human review routing — are covered in depth in `tutorial.md` (Domain 4, Task Statements 4.3–4.5 and Domain 5, Task Statement 5.5), and exercised with original questions in `practice_questions.md` and `scenario_qa.md`.

---

## Quick Reference: Scenario → Domain Coverage

| Scenario | D1 Agentic Arch. | D2 Tool/MCP | D3 Claude Code | D4 Prompt/Structured | D5 Context/Reliability |
|---|:---:|:---:|:---:|:---:|:---:|
| 1. Customer Support Resolution Agent | ✔ primary | ✔ primary | | | ✔ primary |
| 2. Code Generation with Claude Code | | | ✔ primary | | ✔ primary |
| 3. Multi-Agent Research System | ✔ primary | ✔ primary | | | ✔ primary |
| 4. Developer Productivity with Claude | ✔ primary | ✔ primary | ✔ primary | | |
| 5. Claude Code for CI | | | ✔ primary | ✔ primary | |
| 6. Structured Data Extraction | | | | ✔ primary | ✔ primary |

Every domain is exercised by at least two of the six scenarios, so no domain can be skipped even if you get "unlucky" with which 4 of 6 scenarios your sitting draws.
