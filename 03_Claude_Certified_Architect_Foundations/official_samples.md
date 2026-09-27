# Official Sample Questions — Index

> Anthropic's own published sample questions/exercises — public pre-exam study material, not live exam content.

The CCAR-F exam guide (Section 9, "Sample Questions") publishes 12 worked sample questions with official rationale. Every one of them is framed against one of the guide's 6 named scenarios, so — to avoid duplicating content and splitting your attention across two files — **all 12 are reproduced in full, verbatim, with their official rationale, inside `official_scenarios.md`**, immediately under the scenario each one belongs to. That is the correct place to study them, since the exam presents items in exactly that scenario-framed way.

This file exists purely as an index/cross-reference, and to hold the guide's other non-question official study material (the preparation exercises) that isn't scenario-specific.

## Index: Official Sample Questions by Scenario

| # | Scenario | Topic | Location |
|---|---|---|---|
| Q1 | Customer Support Resolution Agent | Programmatic enforcement vs. prompt-based compliance for identity verification before refunds | `official_scenarios.md` → Scenario 1 |
| Q2 | Customer Support Resolution Agent | Tool description quality as the root cause of misrouting between similar tools | `official_scenarios.md` → Scenario 1 |
| Q3 | Customer Support Resolution Agent | Escalation calibration via explicit criteria vs. confidence scores / sentiment analysis | `official_scenarios.md` → Scenario 1 |
| Q4 | Code Generation with Claude Code | Correct scope (`.claude/commands/`) for team-shared slash commands | `official_scenarios.md` → Scenario 2 |
| Q5 | Code Generation with Claude Code | Plan mode vs. direct execution for an architectural restructuring task | `official_scenarios.md` → Scenario 2 |
| Q6 | Code Generation with Claude Code | Path-specific rules (`.claude/rules/` glob patterns) vs. monolithic/directory-bound CLAUDE.md | `official_scenarios.md` → Scenario 2 |
| Q7 | Multi-Agent Research System | Diagnosing a coordinator-level task decomposition failure vs. blaming downstream subagents | `official_scenarios.md` → Scenario 3 |
| Q8 | Multi-Agent Research System | Structured error propagation for intelligent coordinator recovery after a subagent timeout | `official_scenarios.md` → Scenario 3 |
| Q9 | Multi-Agent Research System | Scoped cross-role tool provisioning (principle of least privilege) to cut round-trip latency | `official_scenarios.md` → Scenario 3 |
| Q10 | Claude Code for Continuous Integration | The `-p`/`--print` flag for non-interactive Claude Code in CI pipelines | `official_scenarios.md` → Scenario 5 |
| Q11 | Claude Code for Continuous Integration | Matching synchronous vs. Message Batches API to each workflow's actual latency tolerance | `official_scenarios.md` → Scenario 5 |
| Q12 | Claude Code for Continuous Integration | Multi-pass review architecture to fix attention dilution across a 14-file PR | `official_scenarios.md` → Scenario 5 |

**Coverage note:** the guide's published samples cluster on Scenarios 1, 2, 3, and 5. Scenarios 4 (Developer Productivity with Claude) and 6 (Structured Data Extraction) have **no official sample questions** in the guide — see the "Note on official sample questions" under each of those two scenarios in `official_scenarios.md`. This study kit's `practice_questions.md` and `scenario_qa.md` include original questions written specifically to cover the knowledge/skill bullets those two scenarios draw on, so those domains aren't left unpracticed.

## Official Preparation Exercises (Guide Section 8)

The exam guide also publishes 4 hands-on preparation exercises (not multiple-choice questions, but full build-it-yourself exercises). They're reproduced here for completeness since they are official Anthropic study material, distinct from the sample questions above.

### Exercise 1: Build a Multi-Tool Agent with Escalation Logic

**Objective:** Practice designing an agentic loop with tool integration, structured error handling, and escalation patterns.

**Steps:**
- Define 3-4 MCP tools with detailed descriptions that clearly differentiate each tool's purpose, expected inputs, and boundary conditions. Include at least two tools with similar functionality that require careful description to avoid selection confusion.
- Implement an agentic loop that checks `stop_reason` to determine whether to continue tool execution or present the final response. Handle both `"tool_use"` and `"end_turn"` stop reasons correctly.
- Add structured error responses to your tools: include `errorCategory` (transient/validation/permission), `isRetryable` boolean, and human-readable descriptions. Test that the agent handles each error type appropriately (retrying transient errors, explaining business errors to the user).
- Implement a programmatic hook that intercepts tool calls to enforce a business rule (e.g., blocking operations above a threshold amount), redirecting to an escalation workflow when triggered.
- Test with multi-concern messages (e.g., requests involving multiple issues) and verify the agent decomposes the request, handles each concern, and synthesizes a unified response.

**Domains reinforced:** Domain 1 (Agentic Architecture & Orchestration), Domain 2 (Tool Design & MCP Integration), Domain 5 (Context Management & Reliability)

### Exercise 2: Configure Claude Code for a Team Development Workflow

**Objective:** Practice configuring CLAUDE.md hierarchies, custom slash commands, path-specific rules, and MCP server integration for a multi-developer project.

**Steps:**
- Create a project-level CLAUDE.md with universal coding standards and testing conventions. Verify that instructions placed at the project level are consistently applied across all team members.
- Create `.claude/rules/` files with YAML frontmatter glob patterns for different code areas (e.g., `paths: ["src/api/**/*"]` for API conventions, `paths: ["**/*.test.*"]` for testing conventions). Test that rules load only when editing matching files.
- Create a project-scoped skill in `.claude/skills/` with `context: fork` and `allowed-tools` restrictions. Verify the skill runs in isolation without polluting the main conversation context.
- Configure an MCP server in `.mcp.json` with environment variable expansion for credentials. Add a personal experimental MCP server in `~/.claude.json` and verify both are available simultaneously.
- Test plan mode versus direct execution on tasks of varying complexity: a single-file bug fix, a multi-file library migration, and a new feature with multiple valid implementation approaches. Observe when plan mode provides value.

**Domains reinforced:** Domain 3 (Claude Code Configuration & Workflows), Domain 2 (Tool Design & MCP Integration)

### Exercise 3: Build a Structured Data Extraction Pipeline

**Objective:** Practice designing JSON schemas, using `tool_use` for structured output, implementing validation-retry loops, and designing batch processing strategies.

**Steps:**
- Define an extraction tool with a JSON schema containing required and optional fields, an enum with an "other" + detail string pattern, and nullable fields for information that may not exist in source documents. Process documents where some fields are absent and verify the model returns null rather than fabricating values.
- Implement a validation-retry loop: when Pydantic or JSON schema validation fails, send a follow-up request including the document, the failed extraction, and the specific validation error. Track which errors are resolvable via retry (format mismatches) versus which are not (information absent from source).
- Add few-shot examples demonstrating extraction from documents with varied formats (e.g., inline citations vs. bibliographies, narrative descriptions vs. structured tables) and verify improved handling of structural variety.
- Design a batch processing strategy: submit a batch of 100 documents using the Message Batches API, handle failures by `custom_id`, resubmit failed documents with modifications (e.g., chunking oversized documents), and calculate total processing time relative to SLA constraints.
- Implement a human review routing strategy: have the model output field-level confidence scores, route low-confidence extractions to human review, and analyze accuracy by document type and field to verify consistent performance.

**Domains reinforced:** Domain 4 (Prompt Engineering & Structured Output), Domain 5 (Context Management & Reliability)

### Exercise 4: Design and Debug a Multi-Agent Research Pipeline

**Objective:** Practice orchestrating subagents, managing context passing, implementing error propagation, and handling synthesis with provenance tracking.

**Steps:**
- Build a coordinator agent that delegates to at least two subagents (e.g., web search and document analysis). Ensure the coordinator's `allowedTools` includes `"Task"` and that each subagent receives its research findings directly in its prompt rather than relying on automatic context inheritance.
- Implement parallel subagent execution by having the coordinator emit multiple `Task` tool calls in a single response. Measure the latency improvement compared to sequential execution.
- Design structured output for subagents that separates content from metadata: each finding should include a claim, evidence excerpt, source URL/document name, and publication date. Verify that the synthesis subagent preserves source attribution when combining findings.
- Implement error propagation: simulate a subagent timeout and verify the coordinator receives structured error context (failure type, attempted query, partial results). Test that the coordinator can proceed with partial results and annotate the final output with coverage gaps.
- Test with conflicting source data (e.g., two credible sources with different statistics) and verify the synthesis output preserves both values with source attribution rather than arbitrarily selecting one, and structures the report to distinguish well-established from contested findings.

**Domains reinforced:** Domain 1 (Agentic Architecture & Orchestration), Domain 2 (Tool Design & MCP Integration), Domain 5 (Context Management & Reliability)

## Official "How to Prepare" Checklist (Guide Section 7)

Reproduced verbatim from the guide as a final self-check before scheduling your exam:

- Build an agent with the Claude Agent SDK: implement a complete agentic loop with tool calling, error handling, and session management. Practice spawning subagents and passing context between them.
- Configure Claude Code for a real project: set up CLAUDE.md with a configuration hierarchy, create path-specific rules in `.claude/rules/`, build custom skills with frontmatter options (`context: fork`, `allowed-tools`), and integrate at least one MCP server.
- Design and test MCP tools: write tool descriptions that clearly differentiate similar tools. Implement structured error responses with error categories and retryable flags. Test tool selection reliability with ambiguous requests.
- Build a structured data extraction pipeline: use `tool_use` with JSON schemas, implement validation-retry loops, design schemas with optional/nullable fields, and practice batch processing with the Message Batches API.
- Practice prompt engineering techniques: write few-shot examples for ambiguous scenarios. Define explicit review criteria to reduce false positives. Design multi-pass review architectures for large code reviews.
- Study context management patterns: practice extracting structured facts from verbose tool outputs, implementing scratchpad files for long sessions, and designing subagent delegation to manage context limits.
- Review escalation and human-in-the-loop patterns: understand when to escalate (policy gaps, customer requests, inability to progress) versus resolve autonomously. Practice designing human review workflows with confidence-based routing.
