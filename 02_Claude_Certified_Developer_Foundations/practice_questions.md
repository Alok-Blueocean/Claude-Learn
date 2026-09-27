# Practice Questions — CCDV-F

35 original practice items, allocated proportionally to each domain's exam weight (see `tutorial.md` for the blueprint). These are **original** questions written for this study kit — not reworded versions of the official samples in `official_samples.md`. Most are single-answer multiple-choice; a few are multiple-response, marked "(Select TWO)" or similar, matching the real exam's item format. Answers and rationale are in the Answer Key at the end — try each question before checking.

Domain distribution: Agents and Workflows (5) · Applications and Integration (10) · Claude Code (1) · Eval, Testing, and Debugging (2) · Model Selection and Optimization (6) · Prompt and Context Engineering (4) · Security and Safety (3) · Tools and MCPs (4).

---

### Q1 (Domain 1 — Agent Architecture)
A team needs to build a system that extracts three fixed fields from an invoice, validates them against a schema, and files the result — the same three steps, every time, regardless of the invoice. Which approach is the better fit, and why?

A. An agent, because agents are always more capable than workflows.
B. A workflow, because the sequence of steps is fully knowable in advance and doesn't vary per input.
C. A manager/subagent hierarchy, because any multi-step task benefits from delegation.
D. A single, very long system prompt with no tools, to keep it simple.

### Q2 (Domain 1 — Agent Architecture)
In a manager/subagent hierarchy, what is the primary reason the manager agent should receive only a subagent's summary rather than its full transcript?

A. Subagent transcripts are always confidential.
B. It keeps the manager's context window from being consumed by the subagent's intermediate, noisy tool output.
C. Summaries are cheaper to generate than full transcripts.
D. Anthropic's API does not allow full transcripts to be passed between agents.

### Q3 (Domain 1 — Agent Construction with Claude)
A developer wants Anthropic to run both the agentic loop *and* host the sandboxed environment where the agent's tools (bash, file operations) execute, with persisted, versioned agent configuration. Which surface is the correct fit?

A. A manual `while stop_reason == "tool_use"` loop
B. The Tool Runner (`client.beta.messages.tool_runner`)
C. Managed Agents (`client.beta.agents` / `client.beta.sessions`)
D. The Claude Agent SDK running on the developer's own server

### Q4 (Domain 1 — Agent Construction with Claude)
Why are hooks preferred over a system-prompt instruction for enforcing that a destructive shell command is never executed?

A. Hooks run faster than the model can generate text.
B. A hook is a deterministic, code-executed checkpoint that runs outside the model's control, so it cannot be argued around by a prompt; a system-prompt instruction is only a strong suggestion the model could still fail to follow.
C. System prompts cannot mention shell commands at all.
D. Hooks and system prompts enforce identical guarantees, so it is purely a style preference.

### Q5 (Domain 1 — Agent Patterns and Frameworks) — code reading
```python
@beta_tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"72F and sunny in {city}"

runner = client.beta.messages.tool_runner(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[get_weather],
    messages=[{"role": "user", "content": "What's the weather in Austin and in Denver?"}],
)
final_message = runner.until_done()
```
What does this code do?

A. It manually implements a tool-use loop, checking `stop_reason` and appending `tool_result` blocks itself.
B. It uses the SDK's Tool Runner helper to drive the full agentic tool-call loop automatically, calling `get_weather` as needed until Claude produces a final answer.
C. It calls `get_weather` exactly once regardless of how many cities are mentioned.
D. It requires a separate Managed Agents session to execute `get_weather`.

---

### Q6 (Domain 2 — Claude API Mechanics)
A customer support app must answer user questions in real time inside a chat widget. Which combination is the most appropriate default?

A. The Message Batches API, because it is always the cheapest option.
B. Realtime `messages.create`/`.stream`, because a human is waiting synchronously on the response.
C. Batch requests polled once per hour and cached for reuse.
D. A single request with `max_tokens` set to 0 to minimize latency.

### Q7 (Domain 2 — Claude API Mechanics)
A request uses `max_tokens=64000` to generate a long report. Why should this request use `client.messages.stream(...)` rather than `client.messages.create(...)`?

A. Streaming is required by the API whenever `max_tokens` exceeds 1024.
B. Non-streaming requests with very large `max_tokens` risk exceeding client/HTTP timeout limits waiting for the full response to be generated; streaming avoids the timeout by delivering output incrementally.
C. Streaming responses are billed at a lower per-token rate than non-streaming ones.
D. Streaming is only a UI concern and has no effect on timeouts.

### Q8 (Domain 2 — Claude API Mechanics)
Which content block correctly represents a base64-encoded PNG image as part of a user message?

A. `{"type": "text", "text": "<base64 data>"}`
B. `{"type": "file", "path": "chart.png"}`
C. `{"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": "<base64 data>"}}`
D. `{"type": "image", "url": "chart.png"}`

### Q9 (Domain 2 — Claude API Mechanics)
A request sets `cache_control: {"type": "ephemeral"}` on a large block of system instructions, but on every repeated, otherwise-identical request, `usage.cache_read_input_tokens` is 0. Which of the following is the most likely cause?

A. Ephemeral caches never produce cache reads; only persistent caches do.
B. Something upstream of the cache breakpoint (e.g., the system block or tool list) is varying between requests — such as an embedded timestamp or unsorted JSON — silently invalidating the shared prefix.
C. `cache_control` only applies to `messages`, never to `system`.
D. Cache reads only appear starting on the fourth identical request.

### Q10 (Domain 2 — Claude API Mechanics) — (Select TWO)
Which two statements correctly describe invoking Claude through a third-party platform such as Amazon Bedrock or Google Vertex AI?

A. You should use a dedicated client class (e.g., `AnthropicBedrock`, `AnthropicVertex`) rather than pointing the first-party `Anthropic()` client at a different `base_url`.
B. Every beta feature and header available on the first-party API is guaranteed to be available identically on every third-party platform.
C. Vertex AI authentication uses GCP application-default credentials rather than an Anthropic API key.
D. Bedrock, Vertex, and Foundry all require the exact same model ID string as the first-party API with no prefix or version-separator differences.

### Q11 (Domain 2 — Software Engineering Foundations) — code reading
```python
async def handle_request(user_message: str) -> str:
    client = anthropic.Anthropic()  # note: synchronous client
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": user_message}],
    )
    return response.content[0].text
```
This function is declared `async def` and awaited elsewhere in an async web server. What is the problem with this code?

A. There is no problem; `async def` alone makes any call inside it non-blocking.
B. It uses the synchronous `Anthropic` client inside an async function, so the blocking network call still stalls the event loop; it should use `anthropic.AsyncAnthropic` and `await client.messages.create(...)`.
C. `messages.create` cannot be called from inside a function that is itself async, and this code will raise a `TypeError`.
D. The `model` parameter is invalid for asynchronous requests.

### Q12 (Domain 2 — Claude Application Design)
A team has a detailed CLAUDE.md file in their repository describing coding conventions. A different engineer calls the same model directly through the raw Messages API (not through Claude Code) and is surprised the conventions aren't being followed. What is the most accurate explanation?

A. CLAUDE.md content is embedded into every model checkpoint, so this should never happen — it's a bug.
B. CLAUDE.md is a Claude Code-specific instruction layer; it is not automatically visible to a bare API call, which only sees whatever is explicitly passed in `system` and `messages`.
C. CLAUDE.md only applies to Claude Desktop, never to Claude Code or the API.
D. The raw API always outperforms Claude Code, so following conventions is unnecessary there.

### Q13 (Domain 2 — Claude Application Design)
A developer needs Claude's output to reliably populate a typed `Invoice` object in downstream code (an invoice number and a numeric total). Which approach best fits, versus asking for "valid JSON" in the prompt text alone?

A. Prose instruction is sufficient and is the industry-standard approach.
B. Use `output_config: {"format": {...}}` (structured outputs) or a strict tool schema so the response's shape is enforced, rather than relying on the model to informally comply with a text instruction.
C. Ask for the answer in XML instead, since XML is always safer than JSON.
D. Lower `max_tokens` to force a shorter, therefore more reliable, response.

### Q14 (Domain 2 — Systems Life Cycle / Configuration Management)
Why should a production Claude application pin an exact, dated model ID rather than always using "the latest" model?

A. Dated model IDs are required by the Anthropic Terms of Service for all customers.
B. Pinning prevents a new model release from silently changing application behavior; upgrades should be deliberate and validated against the app's eval suite before rollout.
C. Only dated model IDs support tool use.
D. "Latest" model IDs cost more per token than dated ones.

### Q15 (Domain 2 — Claude API Mechanics) — code reading
```python
batch = client.messages.batches.create(requests=[
    Request(custom_id=f"doc-{i}", params=MessageCreateParamsNonStreaming(
        model="claude-haiku-4-5", max_tokens=512,
        messages=[{"role": "user", "content": f"Classify document {i}."}],
    )) for i in range(1000)
])
# ... later, after the batch has ended:
results = list(client.messages.batches.results(batch.id))
for i, result in enumerate(results):
    save_classification(doc_id=i, output=result.result.message)
```
What is the bug in the code above?

A. `Request` objects cannot be created in a list comprehension.
B. Batch results are not guaranteed to arrive in the same order the requests were submitted, so zipping result index `i` to `doc_id=i` is unsafe; results must be keyed by `result.custom_id` instead.
C. `MessageCreateParamsNonStreaming` cannot be used inside a batch request.
D. `claude-haiku-4-5` is not a valid model for the Batches API.

---

### Q16 (Domain 3 — Claude Code Operation) — (Select TWO)
Which two statements about Claude Code configuration and operation are correct?

A. `settings.json` can define permission allow/deny rules and hooks such as `PreToolUse`.
B. A user-level CLAUDE.md always completely replaces and disables any project-level CLAUDE.md content, rather than layering with it.
C. Headless mode (`claude -p "..."`) requires an interactive terminal session and cannot be used in CI.
D. Auto-mode lets Claude Code proceed through a sequence of actions without pausing for per-step confirmation.

---

### Q17 (Domain 4 — Debugging and Error Handling)
A tool call's `input` field fails `json.loads()` partway through an agentic loop. Which layer does this failure belong to, and what is the correct next step?

A. It is a model-output/reasoning failure; the fix is to switch to a larger model.
B. It is an integration-layer parsing issue; investigate whether eager/fine-grained streaming is being used without validating a possibly-incomplete `partial_json` accumulation, or otherwise harden the parsing before treating this as a model quality problem.
C. It is unrecoverable; the session must be discarded entirely.
D. `json.loads()` failures never occur with tool use and indicate a corrupted SDK installation.

### Q18 (Domain 4 — Debugging and Error Handling) — code reading
```python
try:
    response = client.messages.create(model="claude-sonnet-5", max_tokens=1024, messages=[...])
except Exception as e:
    log.error("Claude call failed: %s", e)
    return fallback_response()
```
What is the main weakness of this error-handling code from a debugging-and-recovery standpoint?

A. It never logs anything, making the failure invisible.
B. It collapses every failure — a non-retryable 400/404, a retryable 429/5xx, and a network-level `APIConnectionError` — into one branch, losing the distinction needed to choose the correct recovery strategy (fix-and-don't-retry vs. back off and retry).
C. `except Exception` is invalid Python syntax.
D. `fallback_response()` should never be called under any circumstances.

---

### Q19 (Domain 5 — LLM Fundamentals)
A developer sends the exact same request to Claude twice, with identical parameters, and receives two different completions. What best explains this?

A. This indicates a bug in the SDK — identical requests should always return byte-identical output.
B. Claude generates text via sampling over a probability distribution at each step; even with fixed parameters, this makes exact reproducibility across calls something the application should not assume.
C. It means the `system` prompt was silently dropped on one of the two calls.
D. It only happens when `thinking` is enabled; without thinking, output is always deterministic.

### Q20 (Domain 5 — LLM Fundamentals)
What is the key difference between extended thinking with a fixed `budget_tokens` and adaptive thinking (`{"type": "adaptive"}`)?

A. Adaptive thinking disables reasoning entirely to save cost.
B. With a fixed budget, the developer specifies a hard thinking-token ceiling up front; with adaptive thinking, the model itself decides how much (if any) reasoning depth to use for the given request.
C. They are two names for the exact same mechanism with no behavioral difference.
D. Adaptive thinking is only available for image inputs.

### Q21 (Domain 5 — Model Selection and Tradeoffs)
A high-volume classification endpoint must stay under tight latency and cost budgets, and the classification task itself is simple (single-label, short input). Which starting point best matches the guide's model-selection reasoning?

A. Always start with the largest, most capable model tier "to be safe," regardless of task simplicity.
B. Favor a smaller, faster model tier for this route, and only escalate to a larger tier if measurement shows the smaller tier/lower effort setting doesn't hold quality for this specific task.
C. Model tier choice has no effect on latency or cost, only on output length.
D. Use extended thinking at maximum effort on every request to guarantee the highest possible accuracy regardless of task simplicity.

### Q22 (Domain 5 — Cost and Token Management)
Which `usage` field should be checked to confirm that prompt caching is actually producing cache hits, rather than assuming it based on request structure alone?

A. `usage.output_tokens`
B. `usage.cache_read_input_tokens`
C. `usage.input_tokens` alone, with no other field
D. There is no way to verify cache hits; caching is a black box.

### Q23 (Domain 5 — Model Selection and Tradeoffs)
A team upgrades their production model from an older release to a newer one without re-running their eval suite, and afterward a previously reliable `thinking` configuration starts returning errors. What does this scenario illustrate?

A. Model upgrades are always fully backward-compatible, so this must be an unrelated infrastructure issue.
B. Breaking behavior changes can occur across model releases (e.g., a parameter that was accepted on the old model being rejected on the new one); this is exactly why model versions should be pinned and upgrades validated against an eval suite before rollout.
C. `thinking` configurations never vary by model and this error is impossible.
D. The fix is to stop using `thinking` permanently on all future models.

### Q24 (Domain 5 — Technical Fundamentals) — code reading
```python
with client.messages.stream(
    model="claude-sonnet-5", max_tokens=512,
    messages=[{"role": "user", "content": "Summarize this."}],
) as stream:
    for event in stream:
        pass
```
What transport mechanism underlies this streaming call at the HTTP level?

A. A persistent WebSocket connection that stays open for the life of the client.
B. Server-Sent Events (SSE) over a standard HTTPS connection — each `event` corresponds to one SSE event from the server.
C. A raw TCP socket with a custom binary protocol.
D. UDP datagrams for low-latency delivery.

---

### Q25 (Domain 6 — Prompt Engineering)
A stable, role-defining persona instruction is currently placed inside the `messages` array on every single turn of a long-running conversation, instead of in `system`. What is the main downside of this placement?

A. Instructions placed in `messages` are ignored by the model entirely.
B. It is repeated (and paid for as input tokens) on every turn instead of benefiting from being cached once as a stable prefix, which is the placement `system` is suited for.
C. `messages` content is always more expensive per token than `system` content.
D. There is no downside; placement between `system` and `messages` is purely cosmetic.

### Q26 (Domain 6 — Context Engineering)
When using server-side compaction in a multi-turn conversation, what is the critical requirement for how the harness must handle the model's response before the next turn?

A. Extract only the plain text of the response and append that string to `messages`.
B. Append the full `response.content` (including any compaction blocks), not just the extracted text, back into `messages` — the API needs those blocks to replace the compacted history on the next request.
C. Discard the response entirely and re-send the original first message every turn.
D. Compaction requires no special handling; any representation of the response works identically.

### Q27 (Domain 6 — Output Handling)
A response is returned via structured outputs and validates successfully against the declared JSON schema. Which statement is most accurate about what this guarantees?

A. Schema validation guarantees the response is both correctly shaped and factually/semantically correct.
B. Schema validation guarantees the response's shape conforms to the schema; it does not guarantee the values are factually or semantically correct, so validation and skepticism toward confident output are still both needed for high-stakes use.
C. Structured outputs make defensive parsing unnecessary in all cases.
D. A schema-valid response can never contain an incorrect value.

### Q28 (Domain 6 — Prompt Engineering / Security overlap)
An application inserts the full text of a fetched web page directly into the same prompt as the system instructions, with no delimiting or labeling. What prompt-engineering practice is being violated?

A. Few-shot prompting.
B. Input sanitization / content-boundary separation — untrusted external content should be clearly delimited from trusted instructions, not concatenated indistinguishably into the same instruction channel.
C. Output constraint specification.
D. Prompt versioning.

---

### Q29 (Domain 7 — AI Application Security)
An internal tool can transfer funds between accounts. Which control most directly limits the damage if a prompt injection attack manages to convince the model to attempt this action?

A. Increasing the model's `max_tokens`.
B. Enforcing least-privilege access on the tool itself (e.g., requiring a separate authorization check, scoping the credential it runs under, or requiring human approval) so that even a successful injection cannot complete the transfer unchecked.
C. Adding a friendly reminder in the system prompt asking the model not to transfer funds without permission.
D. Disabling streaming for this particular request.

### Q30 (Domain 7 — Claude Hooks)
Why is a `PreToolUse` hook that blocks a proposed `Bash` tool call a stronger control than a system-prompt rule saying "never run destructive commands"?

A. The hook runs as deterministic code outside the model's control and can reject the call before execution, regardless of what the model generates; a system-prompt rule is an instruction the model could still fail to follow.
B. Hooks and system-prompt rules provide mathematically identical guarantees.
C. System-prompt rules execute faster than hooks.
D. Hooks can only run after a command has already executed, never before.

### Q31 (Domain 7 — Identity, Secrets, and Key Management)
Where should a production Claude application's API key be stored?

A. Hard-coded directly in the system prompt so it's always available to the model.
B. Committed to the application's version-controlled source code for convenience.
C. In an environment variable or a dedicated secrets manager, loaded at runtime and never embedded in prompts or committed source.
D. In a code comment near the client initialization line.

---

### Q32 (Domain 8 — Tool Implementation)
A tool named `search_orders` is described only as `"searches orders"`. Claude frequently fails to call it when appropriate, or calls it with the wrong parameters. What is the most likely fix?

A. Switch to a larger, more expensive model, since tool selection is purely a function of model size.
B. Rewrite the tool's `description` to state clearly what it does, when to use it, what each input parameter means, and what the output looks like — the description is the only information Claude has about the tool.
C. Remove the tool entirely, since vague tools can never be fixed.
D. Increase `max_tokens` on the request.

### Q33 (Domain 8 — Tool Implementation)
What is the correct way to signal to Claude that a tool call failed, so the agentic loop can recover gracefully?

A. Silently omit any `tool_result` for the failed call and continue the loop.
B. Return a `tool_result` block for that `tool_use_id` with `is_error: true` and a description of the failure, so Claude can see the failure and decide how to proceed.
C. Raise an unhandled exception that terminates the process.
D. Return a `tool_result` with `is_error: false` and empty content.

### Q34 (Domain 8 — Agentic Customization)
A team needs Claude to call a bespoke internal REST API that only their own application uses, with no plan to share it with other applications or teams. Which approach best fits?

A. Build and deploy a full MCP server for this one internal, single-app integration.
B. A custom, client-side tool defined and executed within this application's own code is the better fit, since the capability isn't meant to be reused across multiple applications or maintained independently.
C. Rely on a built-in server-side tool, since built-in tools reach any internal API automatically.
D. Paste the entire API's possible responses into the system prompt ahead of time.

### Q35 (Domain 8 — MCP Server Development)
An MCP server needs to run as a shared, remote service reachable by multiple different client applications over the network, rather than as a local subprocess of a single client. Which transport is the appropriate choice?

A. stdio, because it is the only transport MCP supports.
B. A network-based transport (HTTP/SSE or streamable HTTP), since stdio is scoped to a local subprocess of one client and can't be shared across independently-running remote clients.
C. MCP servers cannot be shared across multiple client applications under any transport.
D. Direct in-process function calls, bypassing MCP's protocol entirely.

---

## Answer Key and Rationale

**Q1: B.** Fixed, invariant, knowable-in-advance sequences are the textbook case for a workflow rather than an agent. (A) is false — agents are not universally "more capable," they trade predictability for flexibility. (C) invokes delegation the task doesn't need. (D) discards the schema-validation step described.

**Q2: B.** Context isolation is the stated purpose — this keeps the manager's window from bloating as work fans out to subagents.

**Q3: C.** "Anthropic runs the loop *and* hosts the sandbox" plus "persisted, versioned configuration" is the definition of Managed Agents specifically, distinguishing it from harness-only options (A, B, D) which all leave hosting to the developer.

**Q4: B.** This is the core rationale for using hooks as guardrails: deterministic, unbypassable enforcement vs. a prompt instruction the model could still fail to honor.

**Q5: B.** `client.beta.messages.tool_runner(...)` plus `.until_done()` is exactly the SDK-managed loop pattern — it will call `get_weather` as many times as needed (including for both cities) without the developer hand-writing the `stop_reason` check.

**Q6: B.** A human waiting synchronously in a chat widget is the definitive realtime case; Batches (A, C) are for latency-tolerant workloads, and (D) misunderstands `max_tokens=0` (used for cache pre-warming, not latency reduction, and would produce no output at all).

**Q7: B.** Large `max_tokens` on a non-streaming call risks hitting HTTP/client timeout limits before generation completes; streaming delivers output incrementally and avoids this. (A) and (C) are fabricated constraints/pricing claims.

**Q8: C.** This is the correct `image` content-block shape: `type: "image"`, `source: {type: "base64", media_type, data}`. The others use non-existent shapes (`"file"` with a bare local `path`, or a `url` field on a base64 image block).

**Q9: B.** A silent invalidator upstream of (or at) the cache breakpoint — a timestamp, non-deterministic ordering, a varying tool list — is the classic cause of a persistently-zero `cache_read_input_tokens`. (A) and (C) are false claims about how ephemeral caching works.

**Q10: A and C.** Use the dedicated client class rather than a `base_url` override (A), and Vertex AI uses GCP ADC auth, no Anthropic API key (C). (B) is false — feature/beta parity is not guaranteed across platforms. (D) is false — Bedrock uses an `anthropic.` prefix and some platforms use dated-snapshot IDs with an `@` separator, unlike the bare first-party ID.

**Q11: B.** Declaring a function `async def` does not make a blocking call inside it non-blocking; the synchronous `Anthropic()` client's network call still stalls the event loop. The fix is the async client with `await`.

**Q12: B.** CLAUDE.md is loaded by Claude Code specifically; it is not part of the model itself and isn't injected into a bare Messages API call unless the developer explicitly passes equivalent content in `system`/`messages`.

**Q13: B.** Structured outputs (or a strict tool schema) enforce the response shape; prose instructions asking for "valid JSON" are not enforced and can be ignored or malformed under edge cases.

**Q14: B.** Pinning is a deliberate-upgrade control, protecting production behavior from a model release changing behavior underneath the application without a validated re-test.

**Q15: B.** Batch results arrive in arbitrary order; the code assumes result order matches submission order, which is unsafe. Results must be correlated via `result.custom_id`.

**Q16: A and D.** `settings.json` does define permissions and hooks (A), and auto-mode does let Claude Code proceed without per-step confirmation (D). Option B is false — CLAUDE.md scopes layer (more specific scopes add to or locally override broader ones), a user-level file does not wholesale disable a project-level one. Option C is false — headless mode is specifically designed for non-interactive use such as CI.

**Q17: B.** A JSON parse failure on tool input is an integration-layer/parsing concern, not evidence of a "dumb model" needing a bigger tier — especially relevant when eager/fine-grained input streaming is in play, since accumulated partial JSON can be incomplete and must be guarded rather than assumed valid.

**Q18: B.** A single catch-all `except Exception` collapses retryable and non-retryable failures into one branch, which prevents choosing the correct recovery strategy for each.

**Q19: B.** Sampling-based next-token generation means exact reproducibility across identical calls is not guaranteed, independent of whether thinking is enabled.

**Q20: B.** Fixed `budget_tokens` is a developer-supplied hard ceiling; adaptive thinking lets the model itself decide reasoning depth per request.

**Q21: B.** Start smaller/cheaper for simple, high-volume, latency-sensitive work, and escalate only if measurement shows it's needed — the guide's own reasoning for model-tier and effort selection.

**Q22: B.** `usage.cache_read_input_tokens` is the field that confirms an actual cache hit occurred, rather than assuming caching worked from request structure alone.

**Q23: B.** This is a textbook breaking-change-across-releases scenario, and the described root cause (skipping eval re-validation before an unpinned upgrade) is exactly what pinning + eval-gated upgrades exist to prevent.

**Q24: B.** The Messages API streams over Server-Sent Events on a standard HTTPS connection, not a persistent WebSocket or a custom raw-socket protocol.

**Q25: B.** Placing a stable instruction in `messages` on every turn forfeits the caching benefit a stable `system` prefix would get, and pays for it as fresh input tokens repeatedly.

**Q26: B.** Server-side compaction requires the harness to preserve the compaction blocks by appending the full `response.content` back into `messages`; extracting only the text string silently loses the compaction state.

**Q27: B.** Schema validation is a shape guarantee, not a correctness guarantee — a syntactically valid but factually wrong value still passes validation, so skepticism and independent checks remain necessary.

**Q28: B.** Untrusted external content (a fetched web page) needs to be clearly separated/delimited from trusted instructions; concatenating it indistinguishably into the same channel as the system prompt is the exact anti-pattern that enables prompt injection.

**Q29: B.** Least-privilege enforcement on the sensitive action itself limits blast radius regardless of whether the model was successfully manipulated; the other options don't provide any enforceable control over the action.

**Q30: A.** A hook is code that runs deterministically and can reject a call before execution; a system-prompt rule is only an instruction the model might not follow. (D) is false — `PreToolUse` runs before execution, which is the entire point.

**Q31: C.** Secrets belong in environment variables or a secrets manager, loaded at runtime — never embedded in prompts, comments, or committed source.

**Q32: B.** The tool description is Claude's only information about the tool; a vague description is almost always the actual root cause of poor tool selection, not model size.

**Q33: B.** Returning a `tool_result` with `is_error: true` lets Claude see and react to the failure; omitting the result (A) or crashing (C) breaks the loop, and marking a real failure as non-error (D) hides the problem from the model.

**Q34: B.** A single-app, non-reusable, internally-maintained integration is the textbook custom-tool case; an MCP server is justified specifically by cross-application reuse and independent maintenance, which this scenario explicitly rules out. (C) repeats the guide's own explicit misconception about built-in tools.

**Q35: B.** stdio is scoped to a local subprocess of a single client; a shared, remote, multi-client server needs a network-based transport such as HTTP/SSE or streamable HTTP.
