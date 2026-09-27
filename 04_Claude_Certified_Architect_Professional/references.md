# CCAR-P: References

## Primary source

- **CCAR-P Architect Professional Exam Guide** (Version 1.0, effective July 2026, exam code CCAR-P) — Anthropic's official, published exam guide for this credential.
  Local path: `C:\Users\arl\LLM\Apps\Claude-Certifications\source_pdfs\CCAR-P_Architect_Professional_Exam_Guide.pdf`
  This is the authoritative source for the exam blueprint, domain weights, task statements, sample questions, scoring model, and all policy content in this study kit's `tutorial.md`, `official_scenarios.md`, and `official_samples.md`.

All URLs below were checked live during preparation of this study kit and were confirmed to resolve to real, current Anthropic (or Anthropic-affiliated) content. Note that Anthropic's developer documentation has migrated from `docs.anthropic.com` to `platform.claude.com` — the old `docs.anthropic.com/...` paths currently 301-redirect to the equivalent `platform.claude.com/docs/...` path, and both are listed as encountered.

## Official Anthropic documentation — production, scale, and architecture topics

**Models and model selection (Domain 2)**
- Models overview (lineup comparison, context windows, pricing, retirement dates): https://platform.claude.com/docs/en/models/overview
- Anthropic's Transparency Hub (per-model capability/safety evaluation reports — relevant to Domain 5's risk/limitations material): https://www.anthropic.com/transparency

**Prompting, context, and cost/latency optimization at scale (Domains 2 and 4)**
- Prompt caching (mechanics, TTLs, pricing — directly underlies the caching material in this kit's `tutorial.md` Section 4.4 and the guide's own Sample Question 2): https://platform.claude.com/docs/en/build-with-claude/prompt-caching
  (legacy path, still resolves via redirect: https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)

**Tool use, agents, and integration (Domain 3)**
- Tool use with Claude — overview (client tools, server tools, the agentic tool-call loop, pricing): https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
- MCP connector — connecting to remote MCP servers from the Messages API, including allowlist/denylist tool configuration (directly relevant to this kit's least-privilege/capability-bloat material in Section 5.1–5.2): https://platform.claude.com/docs/en/agents-and-tools/mcp-connector
- "Introducing the Model Context Protocol" — Anthropic's announcement of MCP as an open standard: https://www.anthropic.com/news/model-context-protocol
- Model Context Protocol specification (the open standard itself, not Anthropic-hosted but the canonical MCP reference): https://modelcontextprotocol.io/

**Evaluation and testing at scale (Domain 4)**
- "Define success criteria and build evaluations" — success-criteria design (specific/measurable/achievable/relevant) and evaluation methods (exact match, cosine similarity, ROUGE-L, LLM-as-judge variants) directly underlying this kit's Section 6 evaluation-methodology material: https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
  (legacy path, still resolves via redirect: https://docs.anthropic.com/en/docs/test-and-evaluate/develop-tests)

**Architecture patterns for agentic systems (Domain 1)**
- "Building Effective AI Agents" — Anthropic's engineering guidance on workflow vs. agent patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer, autonomous agents), the direct basis for this kit's Section 3.1 and 3.3 pattern-selection material: https://www.anthropic.com/engineering/building-effective-agents

## Note on currency

Anthropic's documentation, model lineup, and API surface change frequently. The specific model names, prices, and feature details visible on the pages above at the time this kit was built (September 2026) will drift over time — always check the live page for current specifics rather than treating any number reproduced elsewhere in this kit as permanently accurate. The exam guide itself (Section 1) carries the same caveat: "This guide is subject to change without notice."
