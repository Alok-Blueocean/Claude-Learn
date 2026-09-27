# Claude Certified Developer — Foundations (CCDV-F) — Study Guide

Source of truth: `CCDV-F_Developer_Foundations_Exam_Guide.pdf` (Version 1.0, Effective July 2026). This guide follows the blueprint in that document domain-for-domain. Code examples use the official `anthropic` Python SDK and reflect current API surface (Messages API, tool use, streaming, vision, prompt caching, extended/adaptive thinking, structured outputs). Where a concept is Claude Code– or MCP-specific rather than API-specific, that's called out.

## Exam at a Glance

| Item | Value |
|---|---|
| Credential | Claude Certified Developer – Foundations |
| Exam code | CCDV-F |
| Number of items | 53 |
| Item format | Multiple-choice and multiple-response; each item states how many responses to select |
| Time limit | 120 minutes |
| Delivery | Proctored — online proctored and/or test center |
| Passing score | Scaled score of 720 on a 100–1,000 scale |
| Exam fee | $125 USD |
| Validity period | 12 months |
| Result reporting | Pass/fail with scaled score, plus percent-correct by domain |

## Exam Blueprint

| # | Domain | Weight |
|---|---|---|
| 1 | Agents and Workflows | 14.7% |
| 2 | Applications and Integration | 33.1% |
| 3 | Claude Code | 3.1% |
| 4 | Eval, Testing, and Debugging | 2.6% |
| 5 | Model Selection and Optimization | 16.8% |
| 6 | Prompt and Context Engineering | 11.0% |
| 7 | Security and Safety | 8.1% |
| 8 | Tools and MCPs | 10.6% |
| | **Total** | **100%** |

Sub-skill weights (from the guide's "Detailed objectives by domain"):

| Domain | Sub-skill | Weight |
|---|---|---|
| 1 | Agent Architecture | 4.5% |
| 1 | Agent Construction with Claude | 5.3% |
| 1 | Agent Patterns and Frameworks | 4.9% |
| 2 | Understanding Requirements | 3.4% |
| 2 | Systems Life Cycle | 2.8% |
| 2 | Claude API Mechanics | 6.8% |
| 2 | Software Engineering Foundations | 7.4% |
| 2 | Claude Application Design | 8.6% |
| 2 | Configuration Management | 4.1% |
| 3 | Claude Code Operation | 3.1% |
| 4 | Debugging and Error Handling | 2.6% |
| 5 | LLM Fundamentals | 5.2% |
| 5 | Technical Fundamentals | 6.1% |
| 5 | Model Selection and Tradeoffs | 2.7% |
| 5 | Cost and Token Management | 2.8% |
| 6 | Context Engineering | 3.8% |
| 6 | Prompt Engineering | 4.6% |
| 6 | Output Handling | 2.6% |
| 7 | AI Application Security | 3.2% |
| 7 | Guardrails and Safe Deployment | 2.3% |
| 7 | Claude Hooks | 1.0% |
| 7 | Identity, Secrets, and Key Management | 1.6% |
| 8 | Tool Implementation | 4.4% |
| 8 | MCP Server Development | 2.1% |
| 8 | Agentic Customization | 4.1% |

Weight order for study priority, heaviest first: **Applications and Integration (33.1%)** → **Model Selection and Optimization (16.8%)** → **Agents and Workflows (14.7%)** → **Prompt and Context Engineering (11.0%)** → **Tools and MCPs (10.6%)** → **Security and Safety (8.1%)** → **Claude Code (3.1%)** → **Eval, Testing, and Debugging (2.6%)**. Domains 2 and 5 alone are ~50% of the exam — prioritize accordingly.

---

# Domain 1: Agents and Workflows (14.7%)

## 1.1 Agent Architecture (4.5%)

**Workflow vs. agent — the core decision.** A **workflow** is a predefined sequence of LLM calls and code, wired by the developer (e.g., "classify → route → summarize"). Control flow is fixed at build time. An **agent** is a loop where Claude decides, at each step, what tool to call next based on the result of the previous step — control flow is dynamic and determined by the model at run time.

Choose a workflow when:
- The task decomposition is knowable in advance and doesn't vary per input
- Predictability, latency, and cost control matter more than open-ended flexibility
- You can enumerate the steps and test each one in isolation

Choose an agent when:
- The number and order of steps can't be known ahead of time (e.g., "debug this failing test" — you don't know how many tool calls that takes)
- The task benefits from Claude inspecting intermediate results and adapting
- The cost of a wrong path is recoverable (a step can be undone, or errors surface safely)

**Manager/supervisor hierarchies.** In a multi-agent system, a manager (or "orchestrator") agent decomposes a goal into subtasks and dispatches each to a subagent (which may run with its own tools, model, and context window). The manager sees only subagent *summaries*, not full subagent transcripts — this is what keeps the manager's context window from bloating as work fans out. Subagents improve task execution by:
- **Context isolation** — a subagent researching a narrow question fills its own context with the noisy tool output, and only the digest crosses back to the manager
- **Parallelism** — independent subtasks can run concurrently
- **Specialization** — a subagent can be given a narrower tool set, cheaper model, and a tighter system prompt suited to a single kind of work

## 1.2 Agent Construction with Claude (5.3%)

Anthropic exposes four distinct ways to build "an agent," and picking the right one is itself testable. The key axis is **who supplies the harness** (the agentic loop + context management) and **who supplies the deployment** (the compute the agent runs on):

| Approach | Harness | Deployment | Best for |
|---|---|---|---|
| Manual loop (`while stop_reason == "tool_use"`) | You write it | You host | Full control, no beta dependency |
| Tool Runner (`client.beta.messages.tool_runner`) | SDK provides it | You host | Custom-tool agent without hand-writing the loop |
| Managed Agents (beta, `client.beta.agents` / `sessions`) | Anthropic provides it | Anthropic hosts a per-session sandbox | Server-managed stateful agents, persisted/versioned configs, long-running sessions |
| Claude Agent SDK (`claude-agent-sdk`, separate package) | Anthropic provides it (Claude Code's harness) | You host | Batteries-included coding/filesystem agent on your own infra |

**Manual agentic loop** (you own everything):

```python
import anthropic

client = anthropic.Anthropic()

def get_weather(city: str) -> str:
    return f"72F and sunny in {city}"

tools = [{
    "name": "get_weather",
    "description": "Get the current weather for a city.",
    "input_schema": {
        "type": "object",
        "properties": {"city": {"type": "string"}},
        "required": ["city"],
    },
}]

messages = [{"role": "user", "content": "What's the weather in Austin?"}]

while True:
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        tools=tools,
        messages=messages,
    )
    messages.append({"role": "assistant", "content": response.content})

    if response.stop_reason != "tool_use":
        break

    tool_results = []
    for block in response.content:
        if block.type == "tool_use":
            if block.name == "get_weather":
                result = get_weather(**block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result,
                })
    messages.append({"role": "user", "content": tool_results})

final_text = "".join(b.text for b in response.content if b.type == "text")
print(final_text)
```

**Tool Runner** (SDK drives the loop for you):

```python
from anthropic import Anthropic
from anthropic.lib.tools import beta_tool

client = Anthropic()

@beta_tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"72F and sunny in {city}"

runner = client.beta.messages.tool_runner(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[get_weather],
    messages=[{"role": "user", "content": "What's the weather in Austin?"}],
)
final_message = runner.until_done()
```

**Claude Agent SDK** (Claude Code's harness, packaged as a library — a different product from the Tool Runner):

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    options = ClaudeAgentOptions(
        system_prompt="You are a senior code reviewer.",
        allowed_tools=["Read", "Grep", "Bash"],
        permission_mode="acceptEdits",
    )
    async for message in query(prompt="Review the diff in this repo for bugs.", options=options):
        print(message)

asyncio.run(main())
```

**Managed agent deployment models.** "Self-hosted" means you own the agent loop and the compute where tools execute (manual loop, Tool Runner, or Claude Agent SDK all fall here). "Anthropic-hosted" refers specifically to **Managed Agents**: you create a persisted, versioned Agent config (`POST /v1/agents`), then start Sessions against it. Each session provisions a container as the agent's workspace — bash, file ops, and code execution run there — while the agent loop itself runs on Anthropic's orchestration layer. This is the choice when you want Anthropic to run the loop *and* host the per-session sandbox, or you need persisted/versioned agent configs and long-running sessions with file mounts.

**Hooks for deterministic actions.** A hook is a shell command the harness runs at a fixed lifecycle point (before a tool call, after a tool call, on session start, etc.) — regardless of what the model "decides." Hooks give you a deterministic, unbypassable control point, which is why they're the mechanism of choice for guardrails (see Domain 7) rather than relying on a system-prompt instruction the model could ignore or be talked out of.

## 1.3 Agent Patterns and Frameworks (4.9%)

**Common agent design patterns:**

- **Tool-use loop** — the fundamental pattern: call the model, execute any requested tools, feed results back, repeat until `stop_reason` is no longer `tool_use`.
- **Sub-agents** — spin up a fresh model context (often a cheaper model like `claude-haiku-4-5`) to handle one bounded sub-task and return only a summary, preventing the parent context from absorbing raw tool output.
- **Memory** — persisting facts or state across turns/sessions beyond what fits in one context window (a memory tool, a vector store, or a simple key-value store the agent writes to and reads from).
- **Context-window management** — actively pruning, summarizing, or isolating context so a long-running agent doesn't silently degrade as its transcript grows (see Domain 6 for the specific techniques: compaction, context editing, tool-output pruning).

**Agentic abstraction frameworks.** The exam guide names three by example: **Strands**, **LangGraph**, and **PydanticAI**. All three add a framework-level abstraction (graphs of nodes, typed agents, structured state) on top of the raw Messages API tool-use loop. The tradeoff to know: a framework buys you faster scaffolding, built-in patterns (retries, state machines, multi-agent orchestration primitives) and ecosystem integrations, at the cost of an extra abstraction layer between you and the raw API — which can obscure exactly what's being sent to Claude and make some prompt/cache optimizations harder to control precisely. A hand-rolled loop or the Tool Runner gives full visibility and control at the cost of writing more of the harness yourself. Neither is "correct" — the decision criteria mirror workflow-vs-agent: use a framework when the abstraction matches your problem's shape and team velocity matters; go closer to the metal when you need fine control over context, caching, or latency.

---

# Domain 2: Applications and Integration (33.1%)

This is the largest domain — nearly a third of the exam — and covers everything from requirements gathering through the mechanics of the Messages API to configuration management.

## 2.1 Understanding Requirements (3.4%)

Before writing code, translate business requirements into functional requirements (what the system must do — e.g., "summarize a support ticket in under 200 words") and infrastructure requirements (what it must run on/within — e.g., "must run inside our VPC," "must not send PII to a third party," "must respond in under 3 seconds at p95"). A Claude-specific wrinkle: requirements should explicitly state latency tolerance (streaming vs. batch), data-residency/vendor constraints (first-party API vs. Bedrock/Vertex/Foundry), and whether outputs feed a downstream system that needs strict schema conformance (→ structured outputs).

## 2.2 Systems Life Cycle (2.8%)

Standard SDLC concepts apply to Claude applications like any other software system: requirements → design → build → test → deploy → operate → maintain. Two Claude-specific SDLC touchpoints:

- **Model version pinning** is a release-management decision, not just a technical one — pin a dated model ID in production so a new model release can't silently change your application's behavior mid-flight; upgrade deliberately, on your own schedule, after re-running your eval suite.
- **Prompt versioning** — treat system prompts and tool schemas as versioned artifacts (in source control, with change review), not as strings that get edited in place in production, because a prompt is executable logic.

## 2.3 Claude API Mechanics (6.8%)

Everything goes through `POST /v1/messages`. Tools, streaming, and structured outputs are all features of this one endpoint.

**Basic request:**

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    system="You are a concise technical assistant.",
    messages=[{"role": "user", "content": "Explain the CAP theorem in two sentences."}],
)
print(response.content[0].text)
```

**Streaming** — needed for chat UIs, or any request with a large `max_tokens` that could otherwise hit an HTTP timeout:

```python
with client.messages.stream(
    model="claude-sonnet-5",
    max_tokens=2048,
    messages=[{"role": "user", "content": "Write a short poem about databases."}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
    final_message = stream.get_final_message()  # full Message once the stream ends
```

Use `.get_final_message()` when you want the complete response object (e.g., to inspect `usage` or `stop_reason`) but don't need to handle every individual stream event yourself.

**Vision (image input):**

```python
import base64

with open("chart.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {
                "type": "base64", "media_type": "image/png", "data": image_data,
            }},
            {"type": "text", "text": "What trend does this chart show?"},
        ],
    }],
)
```

**Extended / adaptive thinking:**

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "Design a rate limiter for a multi-tenant API."}],
)
for block in response.content:
    if block.type == "thinking":
        pass  # reasoning summary (or empty, depending on display setting)
    elif block.type == "text":
        print(block.text)
```

**Prompt caching** — mark a stable prefix (system prompt, tool definitions, a large reference document) with `cache_control` so repeated requests skip re-processing it:

```python
response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    system=[
        {"type": "text", "text": LONG_STATIC_INSTRUCTIONS, "cache_control": {"type": "ephemeral"}},
    ],
    messages=[{"role": "user", "content": "Summarize the attached policy in 3 bullets."}],
)
print(response.usage.cache_read_input_tokens)  # >0 confirms the cache hit
```

Rendering order matters: `tools` → `system` → `messages`. Any byte-level change anywhere in the cached prefix invalidates everything after it, so put volatile content (timestamps, per-request IDs) *after* the cache breakpoint.

**Batch API vs. realtime.** The Message Batches API processes large, latency-tolerant workloads asynchronously (results typically within 24 hours) at roughly half the per-token cost of realtime calls. Use it whenever a job doesn't need an answer immediately (nightly reports, bulk classification, dataset labeling); use realtime (`messages.create` / `.stream`) whenever a human or another system is waiting on the response.

```python
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

batch = client.messages.batches.create(requests=[
    Request(
        custom_id=f"doc-{i}",
        params=MessageCreateParamsNonStreaming(
            model="claude-haiku-4-5",
            max_tokens=512,
            messages=[{"role": "user", "content": f"Classify document {i}."}],
        ),
    )
    for i in range(10_000)
])

# poll until done, then stream results — order is NOT guaranteed, key by custom_id
status = client.messages.batches.retrieve(batch.id).processing_status
for result in client.messages.batches.results(batch.id):
    if result.result.type == "succeeded":
        handle(result.custom_id, result.result.message)
```

**Invoking Claude through third-party vendors.** The same Messages API surface is available through Amazon Bedrock, Google Vertex AI, and Microsoft Foundry, each with a dedicated client class rather than a `base_url` override on the first-party client:

```python
from anthropic import AnthropicBedrock, AnthropicVertex, AnthropicFoundry

bedrock_client = AnthropicBedrock(aws_region="us-west-2")
vertex_client = AnthropicVertex(project_id="my-gcp-project", region="us-east5")
foundry_client = AnthropicFoundry(api_key="...", resource="my-foundry-resource")
```

Know the practical differences: Bedrock/Vertex/Foundry model IDs and auth differ from the first-party API (Bedrock uses an `anthropic.` prefix; Vertex uses GCP ADC auth with no Anthropic API key; region is required on both), and not every feature (e.g., some beta headers, Priority Tier, fast mode) is available on every platform — check platform availability before assuming parity.

**Messages API data access patterns.** A "data access pattern" here means how you get data *into* a request: inline in the `messages` array (small, ephemeral data), via the Files API (upload once, reference by `file_id` across many requests without re-uploading), or via a tool call to a live system (for data that must be current at request time). Choose inline content for one-off small payloads, the Files API for a document reused across many requests, and a tool for anything that changes between requests.

## 2.4 Software Engineering Foundations (7.4%)

This sub-skill is *general* software engineering as applied to Claude apps — the exam expects you to reason about it in a Claude-integration context, not in the abstract.

- **REST APIs & JSON** — the Messages API *is* a REST API; requests and responses are JSON. Tool `input_schema` is JSON Schema; tool call arguments arrive as JSON you must parse defensively (`json.loads`, never string-matching).
- **Asynchronous programming** — use the async client (`anthropic.AsyncAnthropic`) inside an async web server so a slow Claude call doesn't block the event loop; use streaming plus async iteration for chat UIs.

```python
import anthropic, asyncio

async def main():
    client = anthropic.AsyncAnthropic()
    async with client.messages.stream(
        model="claude-sonnet-5", max_tokens=1024,
        messages=[{"role": "user", "content": "Hello"}],
    ) as stream:
        async for text in stream.text_stream:
            print(text, end="")

asyncio.run(main())
```

- **Version control** — prompts, tool schemas, and CLAUDE.md/settings.json belong in the same repo and review process as application code; a prompt change is a code change.
- **SDLC integration** — evals and integration tests that exercise the real API (or a recorded fixture of it) belong in CI, gated before merge, the same as unit tests.
- **Code review** — review prompt/tool-schema diffs for the same classes of risk as code diffs: does a broadened tool description now let Claude reach a more dangerous action than intended? Does a schema change break a downstream consumer's structured-output contract?
- **Small- and large-scale refactoring** — refactoring a Claude integration commonly means: extracting tool implementations behind an interface so the model-facing schema doesn't change while the backend does; migrating a manual loop to the Tool Runner (or vice versa) without changing tool behavior; and updating pinned model IDs as part of a deliberate migration, with an eval re-run as the safety net (see Domain 4).

## 2.5 Claude Application Design (8.6%)

**How Claude interprets instructions across interfaces.** The same model can be reached through Claude Code, Claude Desktop, claude.ai, the raw API, and language SDKs — but *instruction precedence* differs by surface. In the API, the `system` parameter and `messages` array are the whole instruction surface (nothing else is injected unless you put it there). In Claude Code, instructions are layered: CLAUDE.md files (project, user, and enterprise-level), Skills, slash commands, and the system prompt Claude Code itself constructs all combine — and a candidate needs to know that a project's CLAUDE.md is *not* visible to a bare API call against the same model; it's a Claude Code-specific injection.

**Content boundaries.** Decide, explicitly, what's trusted instruction vs. untrusted data. Content a user or an external system supplies (a fetched web page, an uploaded file, a database row) should never be concatenated indistinguishably into the same instruction channel as your system prompt — see Domain 7 for the security consequences of getting this wrong.

**Schema design.** When Claude's output must be consumed by code, design the schema *before* the prompt: use `output_config: {format: {...}}` (structured outputs) or a tool-shaped schema (`strict: true` tool with `additionalProperties: false`) rather than asking for "JSON" in prose and hoping.

```python
response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Extract the invoice number and total from this text: ..."}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "invoice_number": {"type": "string"},
                    "total": {"type": "number"},
                },
                "required": ["invoice_number", "total"],
                "additionalProperties": False,
            },
        }
    },
)
```

**Session hygiene.** For multi-turn apps, decide a clear session boundary: when does a conversation reset, how is history truncated or compacted as it grows, and is per-user history isolated so one user's context can never leak into another's request. Don't let a single long-lived `messages` array grow unbounded in production — apply compaction or context editing (Domain 6) before it silently degrades quality or blows past the context window.

**Plugin management.** In Claude Code specifically, plugins bundle Skills, commands, agents, and MCP servers for reuse across projects; managing them means tracking plugin versions and their declared dependencies the same way you'd track a package manager's lockfile, so a plugin update doesn't silently change behavior in a shared repo.

## 2.6 Configuration Management (4.1%)

- **CLAUDE.md files** — project-level instructions Claude Code loads automatically, following a hierarchy (enterprise → project → user), each level able to add to or override the ones above it.
- **settings.json** — Claude Code's configuration file for permissions, hooks, environment variables, and tool allow/deny lists; can exist at user, project, and local (gitignored) scope.
- **Model version pinning** — always pin an exact model ID in production configuration (never "latest") so behavior doesn't change without a deliberate, tested upgrade.
- **Prompt versioning** — store system prompts as versioned files (not inline string literals scattered through code) so a prompt regression can be bisected and rolled back like any other change.
- **Plugin dependencies** — a Claude Code plugin can depend on specific MCP servers or Skill versions; pin those the same way you'd pin a library version.

---

# Domain 3: Claude Code (3.1%)

Claude Code's operating model is built from a small set of composable primitives:

| Component | What it is |
|---|---|
| **Rules** | Persistent instructions loaded into every session (CLAUDE.md content, and enterprise-managed rules) |
| **Skills** | Packaged, discoverable capabilities (a `SKILL.md` plus supporting files) Claude loads on demand when relevant |
| **Commands** | Slash commands — built-in (`/clear`, `/compact`) or custom, defined as markdown files under `.claude/commands/` |
| **Agents** | Subagent definitions (`.claude/agents/*.md`) with their own system prompt, tools, and model — dispatched for a bounded sub-task |
| **Agent Memory** | Persisted notes/state a session or subagent writes and later sessions can read back |

**Session management** — a Claude Code session holds a running transcript; `/clear` resets it, `/compact` summarizes it in place to reclaim context without losing the thread entirely.

**Headless mode** — `claude -p "<prompt>"` runs Claude Code non-interactively (e.g., in a CI job or script) and exits after producing output, instead of opening the interactive REPL.

**Streaming mode** — Claude Code can emit structured JSON events as it works (useful for a script or UI consuming Claude Code's output programmatically) rather than only rendering to a terminal.

**Auto-mode** — a setting that lets Claude Code proceed through a sequence of actions without pausing for per-step confirmation, appropriate for well-scoped, low-risk tasks and generally paired with tighter permission scoping (see Domain 7) since it removes the human-in-the-loop checkpoint.

**The CLAUDE.md hierarchy.** Anthropic-recommended layering: an enterprise-managed CLAUDE.md (org-wide policy), a project CLAUDE.md (checked into the repo, shared by the team), and a user-level CLAUDE.md (personal preferences, not checked in). More specific scopes add to — and can locally override — broader ones.

**Repository initialization** — `claude /init` (or the `init` workflow) scans a codebase and generates a starting CLAUDE.md describing structure, conventions, and commands, rather than a developer hand-writing one from scratch.

**settings.json configuration** — controls permission rules (which tools/commands run automatically vs. require approval), hooks (see Domain 7), environment variables, and the default model — at user, project, and local scope, merged in that order.

Example minimal `settings.json`:

```json
{
  "permissions": {
    "allow": ["Bash(git *)", "Read", "Grep", "Glob"],
    "deny": ["Bash(rm -rf *)"]
  },
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "scripts/check-command.sh" }] }
    ]
  }
}
```

---

# Domain 4: Eval, Testing, and Debugging (2.6%)

## Debugging and Error Handling

**Error type identification.** The first branch is always: is this an *integration-layer* failure (network error, malformed request, auth failure, rate limit) or a *model-output* failure (Claude produced a well-formed but wrong/unhelpful answer)? The two require different remediation, so isolating which one you're looking at is the diagnostic skill being tested.

Typed exceptions make this identification mechanical rather than string-matching:

```python
import anthropic

client = anthropic.Anthropic()

try:
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello"}],
    )
except anthropic.NotFoundError as e:
    # 404 — bad model ID, bad endpoint: an integration-layer bug, not retryable
    log.error("bad request target: %s", e)
except anthropic.RateLimitError as e:
    # 429 — back off and retry (the SDK's built-in retries already handle some of this)
    log.warning("rate limited, backing off: %s", e)
except anthropic.APIStatusError as e:
    # any other non-2xx: inspect e.status_code / e.response for detail
    log.error("API error %s: %s", e.status_code, e)
except anthropic.APIConnectionError as e:
    # network-level failure — never reached Anthropic's servers
    log.error("connection failed: %s", e)
```

Write the chain **most-specific-first**: a single broad `except anthropic.APIStatusError` (or worse, `except Exception`) collapses retryable failures (429, 5xx) and non-retryable ones (400, 404) into the same branch, which is exactly the failure-mode the exam probes for.

**Recovery strategy selection.** Retryable errors (429, 5xx, connection errors) call for backoff-and-retry — the SDK already retries these automatically (`max_retries`, default 2) before you ever see the exception, so an application-level retry loop should generally back off *further*, not duplicate the SDK's own retry immediately. Non-retryable errors (400 malformed request, 404 unknown model) call for fixing the request, not retrying it unchanged. A `stop_reason: "max_tokens"` (truncated output, not an exception at all) calls for a larger `max_tokens` or continuation, not a retry of the same request.

**Trace analysis to identify failure modes.** In an agentic loop, a failure often isn't the *last* turn — it's a wrong tool call, a misread tool result, or a dropped constraint several turns earlier that compounds. Debugging means reading the full transcript of `tool_use` → `tool_result` pairs, not just the final answer, to find the turn where behavior diverged from the intended path.

**Problem origin isolation between the integration layer and model output.** Concretely: if a tool call's arguments fail JSON parsing, that's integration-layer (or a schema issue) — fix schema/parsing, don't blame "the model being unreliable." If the arguments parse cleanly but reflect a wrong understanding of the task, that's a prompt/model-behavior issue — fix the tool description, system prompt, or add a few-shot example, not the parsing code.

---

# Domain 5: Model Selection and Optimization (16.8%)

## 5.1 LLM Fundamentals (5.2%)

- **Tokens** — the unit of input/output cost and context-window accounting; a token is roughly (not exactly) a word-fragment. Count tokens programmatically rather than estimating: `client.messages.count_tokens(model=..., messages=[...])`.
- **Context windows** — the maximum combined input+output tokens a request can carry; exceeding it is a hard error, not a soft degradation.
- **Sampling & non-determinism** — Claude generates the next token from a probability distribution; even at fixed parameters, repeated identical requests can produce different completions. Applications that require reproducibility should not assume identical output across calls; if reproducibility matters, minimize sampling variance and validate output rather than relying on determinism.
- **Next-token generation** — the fundamental generation loop: the model predicts a distribution over the next token conditioned on everything before it, samples one, appends it, and repeats until a stop condition.
- **Model options**: 
  - **Fast mode** — a research-preview mode on select models that trades premium pricing for materially higher output tokens/sec; useful when latency matters more than cost for a specific route.
  - **Extended thinking** — the model produces intermediate reasoning (`thinking` content blocks) before its final answer, improving performance on multi-step reasoning tasks at the cost of extra tokens and latency.
  - **Adaptive thinking** (`thinking: {"type": "adaptive"}`) — the model decides for itself when and how much to think, rather than the developer supplying a fixed token budget.
  - **Effort levels** (`output_config: {"effort": "low"|"medium"|"high"|...}`) — a separate lever from thinking that controls how much overall depth/spend a response gets; lower effort means faster, cheaper, more terse responses, appropriate for simple or high-volume routes.
- **Fundamental prompting techniques**:
  - **Zero-shot** — asking directly, with no examples, relying on the instruction alone.
  - **Single-shot (one-shot)** — one worked example included to anchor format/style.
  - **Multi-shot (few-shot)** — several worked examples, most useful when the desired output has a specific structure or subtle style that's easier to demonstrate than to describe.

## 5.2 Technical Fundamentals (6.1%)

The Anthropic SDKs are thin, typed wrappers over plain REST — every SDK call is ultimately an HTTP request to `api.anthropic.com` (or a partner platform's equivalent endpoint), and streaming responses use HTTP Server-Sent Events (SSE), not a persistent websocket connection, for the Messages API itself. (Websockets appear in some real-time/voice-oriented product surfaces, but the core Messages API streams over SSE.) Knowing this matters for debugging: SDK exceptions map cleanly onto HTTP status codes (see Domain 4), so a raw `curl` reproduction of a failing SDK call is always a valid diagnostic step.

```python
# The SDK's stream() ultimately consumes an SSE response; you rarely touch this directly,
# but knowing it explains why streaming works over plain HTTPS with no special protocol upgrade.
with client.messages.stream(model="claude-sonnet-5", max_tokens=512, messages=[...]) as stream:
    for event in stream:
        pass  # each `event` corresponds to one SSE "event" from the server
```

## 5.3 Model Selection and Tradeoffs (2.7%)

| Tier | Use case | Relative cost | Relative latency |
|---|---|---|---|
| Opus | Highest-stakes reasoning, complex agentic/coding tasks, tasks where quality dominates cost | Highest | Slowest |
| Sonnet | The default balance — strong reasoning at moderate cost/latency, the right starting point for most production routes | Medium | Medium |
| Haiku | High-volume, latency-sensitive, or simple tasks (classification, extraction, subagent workers) | Lowest | Fastest |

Decision heuristic: start with Sonnet, measure; move to Haiku for routes where quality holds at lower effort/model tier (worth measuring before assuming Opus is needed); reserve Opus for tasks that demonstrably need its ceiling (hard multi-step reasoning, high-stakes correctness) — verified by comparing a cheaper model at *lower effort* against a pricier model, since effort tuning on a strong model often beats jumping tiers.

Not every model supports every feature — e.g., adaptive thinking support and effort-level granularity differ across the model line, and a fixed `budget_tokens` thinking configuration that works on one model can be rejected outright on a newer one. **Breaking behavior changes across model releases** are a real operational risk: an upgrade can change stop-reason semantics, remove a parameter (e.g., a fixed thinking token budget), or change JSON escaping behavior in tool inputs. Never assume forward compatibility — pin model versions (Domain 2.6) and re-run your eval suite before rolling a model upgrade into production.

## 5.4 Cost and Token Management (2.8%)

**Token usage tracking** — every response carries a `usage` object; log it per request so cost is measurable, not estimated:

```python
response = client.messages.create(model="claude-sonnet-5", max_tokens=1024, messages=[...])
print(response.usage.input_tokens, response.usage.output_tokens)
print(response.usage.cache_read_input_tokens, response.usage.cache_creation_input_tokens)
```

**Cost modeling** — cost is a function of input tokens, output tokens, and (with caching) a mix of cache-write and cache-read tokens at different rates than a plain input token; model your app's cost per completed task, not per request — a cheaper model that needs three retries to succeed isn't actually cheaper.

**Caching techniques**:
- **Prompt caching** — mark a stable prefix with `cache_control: {"type": "ephemeral"}`; a cache hit is billed at a steep discount versus a fresh input token, but only the *unchanged prefix up to the first invalidation point* benefits.
- **Cache check-pointing** — placing multiple `cache_control` breakpoints (up to 4 per request) at different levels of stability (e.g., one after tool definitions, one after a large static document, one after slower-changing conversation history) so that partial invalidation doesn't force a full-prefix cache miss.

Verify caching is actually working — don't assume it: `usage.cache_read_input_tokens` should be non-zero on repeat requests; if it's persistently zero, something in the "stable" prefix is silently varying (an embedded timestamp, unsorted JSON keys, a per-request tool list).

---

# Domain 6: Prompt and Context Engineering (11.0%)

## 6.1 Context Engineering (3.8%)

**Context window management** — actively budget what occupies the context window rather than letting it fill passively; a bloated context degrades both latency/cost and answer quality (irrelevant tokens compete for the model's attention).

**Preventing context drift and bloat**:
- **Tool output pruning** — a tool result (e.g., a full file read, a large API response) doesn't need to stay in context verbatim forever; once its information has been used or summarized, clear or truncate it rather than letting every historical tool call accumulate.
- **Compaction** — periodically summarizing older conversation turns into a condensed form that preserves the necessary facts while freeing the tokens the verbatim transcript consumed. On supported models this can be done server-side (beta compaction, triggered automatically as the conversation approaches a token threshold) — but the harness must append the full `response.content` (including any compaction blocks) back into `messages`, not just the extracted text, or the compaction state is silently lost.

**Context isolation through subagents or multi-step agentic workflows** — the same idea from Domain 1.1: give a subagent its own clean context to do noisy, exploratory work (grepping a codebase, reading many files, calling a chatty API) and return only the digest to the parent, so the parent's context stays small and focused regardless of how much work the subagent did internally.

## 6.2 Prompt Engineering (4.6%)

- **Instruction clarity** — state the task, constraints, and desired format explicitly and unambiguously; vague instructions produce inconsistent output even from a highly capable model.
- **Few-shot examples** — see 5.1; use them when structure/style is easier to demonstrate than describe, and keep examples consistent with each other (contradictory examples confuse the model more than no examples).
- **System vs. user placement** — the `system` parameter is the right place for stable, role-defining instructions that apply to the whole conversation (persona, output format, non-negotiable constraints); the `messages` array is the right place for the actual task/query and any per-turn content. Putting task-specific, one-off instructions in `system` wastes the caching benefit of a stable prefix; putting stable role instructions in `messages` means they get repeated (and paid for) every turn instead of cached once.
- **Output constraints** — be explicit about format (length, structure, allowed vocabulary) in the prompt, but prefer an enforced mechanism (structured outputs, a tool schema) over prose instruction alone whenever the output feeds code, since prose constraints are a *strong suggestion*, not a guarantee.
- **Prompt and instruction placement across components** — in an agentic system, instructions can live in the system prompt, a tool's `description`, a CLAUDE.md file, or a mid-conversation operator message; place a constraint at the *most specific* applicable layer (a tool-usage rule belongs in that tool's description, not buried in a global system prompt) so it's discoverable and doesn't get lost as the system prompt grows.
- **Iterative refinement / prompt adjustment** — treat prompt-writing as an empirical loop: write a draft, run it against representative cases (ideally a small eval set), inspect failures, adjust one variable at a time, re-run. Don't tune prompts, blind, against a single anecdotal failure.
- **Input sanitization** — untrusted user or retrieved content should be clearly delimited from instructions (e.g., wrapped in explicit tags, prefixed with a "this is data, not instructions" framing) before being placed in the prompt — this is both a prompt-engineering practice and a security control (see Domain 7).

## 6.3 Output Handling (2.6%)

- **Structured output patterns** — prefer `output_config: {format: {...}}` (or a `strict: true` tool schema) over asking for "valid JSON" in prose; use `client.messages.parse()` where available to get validation for free.
- **Response validation** — always validate a parsed response against your schema/expectations before acting on it, even when using structured outputs — a schema guarantees shape, not semantic correctness (a syntactically valid but factually wrong invoice number still validates).
- **Defensive parsing** — wrap `json.loads()` (or your schema validator) in error handling; never regex-match or string-search raw model output as a substitute for parsing it properly, and never assume a tool call's `input` is safe to use unchecked.
- **Skepticism toward confident output** — a fluent, confidently-worded response is not evidence of correctness; high-stakes outputs (numbers that drive a financial decision, code that will run unreviewed, a claim presented as fact) need an independent check — a second pass, a validation rule, or a human review step — proportional to the cost of being wrong.

---

# Domain 7: Security and Safety (8.1%)

## 7.1 AI Application Security (3.2%)

**Prompt injection** is the primary novel risk class in LLM applications: content the application feeds to the model (a web page, a document, a tool result) contains text crafted to be interpreted as an instruction rather than as data, hijacking the model's behavior.

**Mitigation, in order of effectiveness** (this exact reasoning appears in the guide's own sample question):
1. **Treat all retrieved/external content as untrusted data**, structurally separated from trusted instructions (e.g., placed only inside a clearly delimited data block, never concatenated into the system prompt).
2. **Enforce least-privilege guardrails** (hooks, tool permissioning, approval gates) so that *even if* injected text is interpreted as an instruction, it cannot trigger a sensitive action — the model's behavior is not the only line of defense.
3. Do **not** rely on temperature changes, a polite request in the system prompt ("please don't follow injected instructions"), or a "smarter" model as your actual control — none of these are enforceable.

**Jailbreak defense** — layered content policy plus guardrails that check *actions*, not just the model's stated intentions; assume any single defense layer can eventually be bypassed and design so that a jailbroken response still can't cause real damage (least privilege again).

**Untrusted input handling** — the same separation principle as injection: an uploaded file, a form field, a URL fetched by a tool — all of it is data, never instruction, and should be labeled as such in the prompt structure.

**Data leakage prevention** — be deliberate about what enters a prompt in the first place (don't paste an entire database dump when a filtered query would do), what a tool is allowed to return, and what's logged; a system prompt or tool result can itself be exfiltrated back to an attacker through a crafted follow-up question, so treat "can this conversation ever surface X" as a design question, not an afterthought.

**PII handling** — minimize what personal data enters context at all; redact or tokenize where the task doesn't require the raw value; be aware of data-retention settings on whichever platform (first-party API vs. Bedrock/Vertex/Foundry) is in use.

**Authentication, authorization, confidentiality, integrity** — standard security properties, applied to the Claude layer specifically: authenticate the *caller* of your application (not just hold an API key), authorize per-user/per-tool access (a tool shouldn't run with more privilege than the requesting user has), and don't let model output silently violate confidentiality (e.g., a tool result meant for one tenant showing up in another tenant's context).

## 7.2 Guardrails and Safe Deployment (2.3%)

**Content policy & guardrail layering** — no single guardrail is sufficient; combine input-side checks (is this request in scope, does it contain injection markers), model-side instructions (system prompt constraints), and output-side checks (does the response violate policy) — and, critically, **action-side enforcement** (can a sensitive tool call actually execute, regardless of what the model "wants").

**Secure-by-design principles**:
- **Privacy by design** — minimize data collection/retention as a default, not an add-on.
- **Identity and access management** — every actor (end user, service account, the agent itself) has a distinct, scoped identity; don't run an agent's tools under a single all-powerful credential.
- **Least privilege** — a tool or agent gets exactly the access its task requires and no more; this is the single most load-bearing principle across this whole domain, and it's why "isolate untrusted content" alone is an incomplete answer to prompt injection — least privilege is what limits the blast radius when isolation fails.

## 7.3 Claude Hooks (1.0%)

Hooks are deterministic, code-executed checkpoints in the agent lifecycle (e.g., Claude Code's `PreToolUse`, `PostToolUse`, `SessionStart`) — the mechanism for enforcing a guardrail that *cannot* be argued out of by a clever prompt, because it runs outside the model's control entirely.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "scripts/block-destructive-commands.sh" }]
      }
    ]
  }
}
```

A `PreToolUse` hook can inspect the proposed tool call and exit non-zero to block it before it ever runs — the correct place to stop a destructive `rm -rf` or an unauthorized network call, rather than trying to prevent it purely through system-prompt wording.

## 7.4 Identity, Secrets, and Key Management (1.6%)

- Never place API keys or credentials directly in a system prompt or in code committed to version control; load from environment variables or a secrets manager.
- **Identity validation and authentication** — verify the calling user/service before honoring a request, independent of anything Claude itself does.
- **Access approval and level verification** — a tool that touches a sensitive system should check the *current* caller's authorization at call time, not rely on a check performed earlier in the session.
- **Authorized access monitoring** — log which credentials/tools were used for which action, so an incident can be reconstructed; this is what turns "least privilege" from a design intent into an auditable guarantee.

---

# Domain 8: Tools and MCPs (10.6%)

## 8.1 Tool Implementation (4.4%)

**Tool use / function calling — the basic shape:**

```python
tools = [{
    "name": "search_orders",
    "description": (
        "Search the order database by customer email. Returns up to 10 matching orders "
        "with order ID, status, and total. Use this before answering any question about "
        "a customer's order history."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "email": {"type": "string", "description": "Customer's email address"},
        },
        "required": ["email"],
    },
}]
```

**Tool description writing** is a first-class skill on this exam: the description is the *only* information Claude has about when and how to use a tool. A good description states what the tool does, when to call it (and when not to), what its inputs mean, and what its output looks like — vague descriptions ("searches orders") produce inconsistent tool selection; the fix is almost always a better description, not a bigger model.

**Error handling** — a failed tool call should still return a `tool_result`, marked accordingly, rather than being silently dropped:

```python
tool_results.append({
    "type": "tool_result",
    "tool_use_id": block.id,
    "content": "Error: order database timed out after 5s",
    "is_error": True,
})
```

Returning `is_error: True` lets Claude see the failure and decide how to proceed (retry, ask the user, try a different approach) instead of the loop silently stalling or Claude hallucinating a result.

**Tool usage patterns:**
- **Agentic harness dispatch** — the harness (manual loop, Tool Runner, or Managed Agents sandbox) is responsible for actually executing whatever tool Claude requests; Claude only ever *proposes* a call (name + validated-shape input) and never executes anything itself.
- **Client-side vs. server-side tools** — a client-side (user-defined) tool runs in your own code/infrastructure, invoked via the standard tool-use loop. A server-side tool (web search, code execution, computer use) runs on Anthropic's infrastructure and its result arrives as a content block in the same response, with no execution loop on your side at all.
- **Approval patterns** — for sensitive tools, insert a human-in-the-loop approval step between `tool_use` and execution (common in Managed Agents' permission policies: `always_allow` / `always_ask` / `auto`), rather than auto-executing every proposed call.

**Tool set construction best practices** — keep the number of tools in one request focused and non-overlapping (an oversized, redundant tool set makes selection *worse*, not better); write schemas that make invalid inputs structurally impossible where feasible (enums over free text, `strict: true` where supported); and give each tool one clear responsibility rather than one mega-tool with a dozen modes selected by a parameter.

## 8.2 MCP Server Development (2.1%)

The **Model Context Protocol (MCP)** standardizes how an application exposes tools, resources, and prompts to an LLM client, so a capability built once can be reused across many Claude applications (and non-Claude MCP clients) rather than being re-implemented per app.

**Server authoring** (Python, using the official MCP SDK):

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("inventory-server")

@mcp.tool()
def check_stock(sku: str) -> str:
    """Check current stock level for a SKU."""
    return f"SKU {sku}: 42 units in stock"

@mcp.resource("inventory://catalog")
def get_catalog() -> str:
    """Expose the product catalog as a readable resource."""
    return "..."

if __name__ == "__main__":
    mcp.run()  # defaults to stdio transport
```

**MCP resources, tools, and prompts** — the three primitives an MCP server can expose: **tools** (callable actions, analogous to Claude API tool use), **resources** (readable data the client can fetch, like files or query results), and **prompts** (reusable, parameterized prompt templates the server offers to any connecting client).

**Communication patterns:**
- **stdio** — the server runs as a local subprocess and communicates over standard input/output; simplest, common for local development and Claude Code/Desktop integrations.
- **Sockets (HTTP/SSE or streamable HTTP)** — the server runs as a standalone network service; required when the server is remote, shared across users, or deployed independently of any one client.
- **Client vs. server roles** — the *client* (e.g., Claude Code, or your own application via the MCP connector) initiates the connection and calls tools/reads resources; the *server* owns the actual integration logic and can be versioned/deployed independently of any client that consumes it.

**Connecting an MCP server from the Messages API** (the MCP connector — both halves are required):

```python
response = client.beta.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    betas=["mcp-client-2025-11-20"],
    mcp_servers=[{"type": "url", "url": "https://mcp.example.com/inventory", "name": "inventory"}],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "inventory"}],
    messages=[{"role": "user", "content": "Is SKU 88213 in stock?"}],
)
```

## 8.3 Agentic Customization (4.1%)

Weighing **built-in tools vs. custom tools vs. Skills vs. MCPs** is a recurring exam theme (it's also one of the guide's own sample questions):

| Approach | What it is | Best for |
|---|---|---|
| **Built-in tool** | Anthropic-hosted, no code to run (web search, code execution, computer use) | Capabilities Anthropic already operates server-side — don't build what's already offered |
| **Custom tool** | You define the schema and execute it yourself, in-process | One-off, app-specific integrations not worth externalizing |
| **Skill** | Packaged instructions + optional supporting files/scripts, loaded on demand | Reusable *know-how* (how to build a slide deck, how to follow a team's runbook) more than a live system integration |
| **MCP server** | A standalone service exposing tools/resources/prompts over a protocol | A capability that must be reusable across multiple applications and maintained independently of any one of them |

A built-in tool does **not** automatically reach an arbitrary internal system — it only does what Anthropic has implemented server-side (e.g., public web search), so "we need Claude to call our internal REST API" is never satisfied by a built-in tool alone. The deciding question for custom tool vs. MCP server is almost always *reuse and independent lifecycle*: a single app's one-off integration is a custom tool; a capability meant to serve several Claude applications, maintained on its own release schedule, is an MCP server.
