# References

## Primary Source (Authoritative)

- **CCAR-F Architect Foundations Exam Guide** (Version 1.0, Effective July 2026, Exam code CCAR-F) — Anthropic's official published exam guide for the Claude Certified Architect – Foundations certification.
  Local copy: `C:\Users\arl\LLM\Apps\Claude-Certifications\source_pdfs\CCAR-F_Architect_Foundations_Exam_Guide.pdf`
  This is the single authoritative source for the exam blueprint, domain weights, task statements, the 6 named scenarios, sample questions with rationale, preparation exercises, and the in-scope/out-of-scope appendix used throughout this study kit. Every file in this folder is derived from, and should be checked against, this document.

## Anthropic Documentation (docs.claude.com)

These are real, currently-published Anthropic documentation pages directly relevant to the exam's five domains. Use them to go deeper than the exam guide's summaries — the guide tells you *what* is tested, these pages tell you the full mechanics.

- **Claude Agent SDK overview** — https://docs.claude.com/en/docs/agents-and-tools/claude-agent-sdk/overview
  Agent definitions, the agentic loop, session management — foundational for Domain 1.
- **Claude Code sub-agents** — https://docs.claude.com/en/docs/claude-code/sub-agents
  Coordinator/subagent patterns, the `Task` tool, `AgentDefinition` configuration — Domain 1, Task Statements 1.2–1.3.
- **Claude Code hooks reference** — https://docs.claude.com/en/docs/claude-code/hooks
  `PostToolUse` and other hook events for tool-call interception and data normalization — Domain 1, Task Statement 1.5.
- **Model Context Protocol (MCP) in Claude Code** — https://docs.claude.com/en/docs/claude-code/mcp
  `.mcp.json` scoping, environment variable expansion, MCP resources — Domain 2, Task Statement 2.4.
- **Tool use overview (Claude API)** — https://docs.claude.com/en/docs/build-with-claude/tool-use/overview
  `tool_use`, `tool_choice` (`auto`/`any`/forced), JSON schema design for tools — Domain 2 and Domain 4.
- **Claude Code memory (CLAUDE.md)** — https://docs.claude.com/en/docs/claude-code/memory
  CLAUDE.md hierarchy (user/project/directory), `@import` syntax — Domain 3, Task Statement 3.1.
- **Claude Code slash commands** — https://docs.claude.com/en/docs/claude-code/slash-commands
  `.claude/commands/` project vs. user scoping — Domain 3, Task Statement 3.2.
- **Claude Code CLI reference** — https://docs.claude.com/en/docs/claude-code/cli-reference
  `-p`/`--print`, `--output-format json`, `--resume`, session flags used in CI/CD integration — Domain 3, Task Statement 3.6.
- **Message Batches API** — https://docs.claude.com/en/docs/build-with-claude/batch-processing
  50% cost savings, 24-hour processing window, `custom_id` correlation, no mid-request tool calling — Domain 4, Task Statement 4.5.
- **Messages API reference** — https://docs.claude.com/en/api/messages
  `stop_reason` values (`tool_use`, `end_turn`), request/response shape underlying the agentic loop — Domain 1, Task Statement 1.1.
- **Context windows / prompt engineering overview** — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
  Few-shot prompting, explicit criteria design — Domain 4, Task Statements 4.1–4.2.
- **Model Context Protocol specification** — https://modelcontextprotocol.io/
  The protocol-level specification behind MCP tools, resources, and the `isError` pattern referenced throughout Domain 2 — maintained by Anthropic and the open MCP community, not an Anthropic-only property, but the normative reference for the protocol tested on this exam.

## Anthropic Engineering Blog

Long-form engineering writing on the exact architecture patterns this exam tests — multi-agent orchestration, tool design, and agentic system design.

- **"Building effective agents"** — https://www.anthropic.com/engineering/building-effective-agents
  The canonical reference for prompt chaining vs. routing vs. orchestrator-worker patterns — directly underlies Domain 1, Task Statement 1.6 (task decomposition strategies) and the plan-mode-vs-direct-execution judgment tested in Domain 3.
- **"How we built our multi-agent research system"** — https://www.anthropic.com/engineering/built-multi-agent-research-system
  Anthropic's own account of building a coordinator/subagent research pipeline — this is the real-world system the exam's Scenario 3 ("Multi-Agent Research System") is modeled on, and covers parallel subagent spawning, context passing, and synthesis with provenance in more depth than the exam guide.
- **"Claude Code: Best practices for agentic coding"** — https://www.anthropic.com/engineering/claude-code-best-practices
  CLAUDE.md conventions, plan mode, and CI/CD integration patterns — Domain 3 in its entirety.
- **"Writing tools for agents"** — https://www.anthropic.com/engineering/writing-tools-for-agents
  Tool description quality, avoiding overlapping/ambiguous tools, scoping tool access per agent — Domain 2, Task Statements 2.1 and 2.3.

## Notes on Using These References

- The exam guide (source PDF above) is the only source that should be treated as normative for exam content — domain weights, scenario wording, and sample-question rationale must match it exactly. The documentation and blog links above are supplementary depth reading; if anything in this study kit's explanatory prose seems to conflict with the exam guide, the exam guide wins.
- URLs above were selected because they are pages/posts the author is confident actually exist on docs.claude.com and anthropic.com/engineering as of this writing. Anthropic documentation URLs do change over time — if a link 404s, search docs.claude.com or anthropic.com/engineering for the same topic title.
- No fine-tuning, embedding/vector-database, computer-use, or vision documentation is listed here, consistent with the exam guide's own out-of-scope appendix (see `tutorial.md`).
