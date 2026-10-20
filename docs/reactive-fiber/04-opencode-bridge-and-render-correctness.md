# 04 — OpenCode model boundary and Python Reactive Fiber bridge

**Status:** target design, not shipped. 2026-10-20. Read together with [01](01-runtime-and-opencode.md). **Pinned upstream:** `anomalyco/opencode@23fb5e0516c99ac04a1aa46c193efda2e1b9bb24` (1.14.18), referenced by `harness2/config.py`.

## Primary design choice

OpenCode retains its agent/model/tool/session loop. A *protected* OpenCode plugin invokes a long-lived Python context engine based on FiberOpticsSDK, ReactiveLensesSDK and ReactiveComponentsSDK. Context is computed *inside* the model-request path, as a bounded, source-revision-pinned projection. Harness² chooses and tests candidate definitions but is not involved in every render. No RF-CLI, Forge SDK, alternate agent loop, global graph merger, or network service is needed.

The native ReactiveAgentsSDK mode remains a first-class independent executor. The two modes share versioned ContextProjection and ExperimentEvidence contracts, not internal lifecycle implementation.

## Upstream source seam and precise call ordering

At the pinned OpenCode revision:
1. `packages/opencode/src/session/prompt.ts` loops over a session's filtered messages; on each normal model step it invokes `plugin.trigger("experimental.chat.messages.transform", {}, {messages: msgs})` before `MessageV2.toModelMessagesEffect(msgs, model)`.
2. `packages/opencode/src/session/compaction.ts` invokes **the same** messages-transform hook before its own conversion. The hook alone does **not** distinguish normal inference from compaction.
3. `packages/opencode/src/session/llm.ts` invokes `experimental.chat.system.transform` before provider assembly. It has optional session ID and a model parameter, but is a *system context* authority: never insert tool/environment data as privileged instructions there.
4. `packages/plugin/src/index.ts` declares `tool.execute.before/after`, `event`, `experimental.session.compacting`, and `chat.message`, all with different information/coverage limits. Tool-after is not evidence that *every* attempted, rejected, provider-executed or cancelled tool produced an event.
5. `packages/opencode/src/plugin/index.ts` executes hooks sequentially and awaits each hook; any slow Python render directly affects request latency.

Do not claim full request identity, agent name or per-step id are available in `messages.transform` input: its input object is empty. Inspect the installed schema and real message parts; derive session identity only when uniquely grounded. An invocation without a unique identity must refuse the complete-context feature or use a narrowly labelled best-effort experimental mode. Do not guess.

## Two stages of OpenCode support

### OC-RF0: Hook-only compatibility probe

Goal: prove a sidecar can return current reactive context during real OpenCode inference, without patching upstream. Use a deterministic fake Python context engine first and then actual installed SDKs. Exercise normal inference, subagent session, tool round, compaction, multi-session concurrency and retry. Verify whether the message collection provides an unambiguous current session ID and whether the plugin can produce a valid, *ephemeral*, lower-trust model-visible context segment accepted by `toModelMessagesEffect`.

The injected segment must:
- Be freshly recomputed (or validated unchanged) per request basis, with one stable identity per model step and per source cut; no append-only inflation across steps.
- Never be written into persistent user-authored text, agent standing instructions or permanent session history; contain a clear delineation of observations versus instructions.
- Preserve untouched OpenCode message information, tool calls/results, and media representations; test resulting provider-compatible messages, not just the hook's array.
- Carry context digest, revision, include/exclude reason refs and evidence status into a sidecar receipt, not necessarily all into model tokens.
- Fail if required resources are unavailable or provenance is invalid; permitted optional resources can show explicitly marked stale/pending/unknown states.

**OC-RF0 is a probe**, not a claim of production correctness. In particular, the compaction hook invokes the same transformation. If verified request-kind discrimination is impossible, do not hack around it with wall-clock or last-event guesses.

### OC-RF1: Narrow context-request patch (conditional)

If OC-RF0 cannot distinguish normal request, compaction, and session/request identity in an authoritative manner, maintain a small pinned local patch to OpenCode's model-message conversion seam. Proposed `context.beforeModel` hook supplies `sessionID, modelID, agentID, modelCallID, requestKind, messageBasisDigest, configuredBudget`, receives typed **user-level/tool-level informational content**, and does not allow changing role authority or hidden evaluation state. The patch calls exactly once per model request and once separately for compaction, then constructs valid model messages; no global agent-loop replacement or side scheduler.

Required patch validation: before/after parity with the plugin disabled, all provider families used by actual Harness² benchmarks, compaction, retries, subagents, tool replay, hidden grading isolation, and prompt caching. Patch SHA/version becomes an experiment identity field. Rebase only through an explicit qualification gate.

## Bridge protocol: local, bounded, versioned

Protocol candidate `reactive.bridge/v1`. Start one Python context process per isolated rollout; sessions addressed by `(runID, sessionID, instanceEpoch)`. Use bounded newline-delimited JSON over a supervised local IPC channel (stdio or Unix socket); never require MCP, HTTP, an RF-CLI workspace or a cloud service for pre-model renders. Messages are length-bounded and carry monotonic request sequence, idempotency key and deadline.

| Request | Required basis | Response |
|---|---|---|
| `hello` | protocol versions, SDK lock, adapter hash | negotiated capabilities or explicit incompatibility |
| `open` | run, session, definition digest, authority scope | instance epoch + graph generation |
| `observe` | source owner/revision, event ID, typed payload or trusted payload ref | accepted/refused observation receipt |
| `render` | session, invocation kind, message basis, graph generation, model view and budget, admitted source cut | ContextProjection + render receipt |
| `compact` | session, context basis, retention/projection policy | compaction context + retained-source refs |
| `snapshot` | session and graph generation | pinned source/graph receipt |
| `close` | session identity | final counters + outstanding effect/revision status |

Example sketch:

```json
{
  "schema": "reactive.bridge/v1",
  "id": "req-15",
  "op": "render",
  "run_id": "run-01",
  "session_id": "session-A",
  "instance_epoch": "epoch-1",
  "model_call_id": "call-4",
  "request_kind": "normal",
  "definition_digest": "sha256:...",
  "graph_generation": 2,
  "message_basis_digest": "sha256:...",
  "source_cut_ref": "cut-7",
  "view": "coding-worker",
  "token_budget": 2400,
  "deadline_ms": 2000
}
```

A READY response contains model-visible blocks, ordering, token-budget rule, context digest, source observations/owner revisions, graph generation, freshness, and trace receipt references. Other statuses `PENDING | UNAVAILABLE | WITHHELD | INVALID | STALE` are distinct; query logical UNKNOWN is **not** a bridge PENDING status. Do not assume full joint causal consistency merely from one latest revision per source.

## Lifecycle and cache discipline

- Source/observation data is admitted by its owner adapters before Fiber invalidation. A digest-only notification makes the import dirty, not READY.
- Model inference is an explicitly admitted effect, never automatic on every lens/source change.
- The Python process retains graph sessions, dependency-index state and reusable render computations. Candidate revision/generation changes are explicit transitions, not live mutation of a running definition.
- Cache keys include context view, model/tokenizer accounting, component and definition revisions, accepted source cut, authority visibility projection and all observer-readable provenance/status/coverage fields.
- A stable text prefix is a measurable optimization, not a reason to serve stale context. Compare real provider prefill/cached-token figures when available.
- Cancelled/timed-out IPC does not infer remote cancellation or successful cleanup. Record uncertain outstanding work, generation fence, and receipt.
- A failed DevTools collector never prevents valid render or model execution; source-critical render errors can prevent the model request as specified.
- Observed visual/UI feedback has no privileged channel into prompts: import into separately admitted data blocks only.

## Plugin installation and protection

Harness² `jobbench.py` maps candidate `plugin/*.ts` into `.opencode/plugin/*.ts`; other benches use different install modes. Protected plugin is materialized from a separately pinned integration artifact after ordinary candidate staging, with collision rejection and fingerprint verification. Candidate-controlled C8 plugins cannot replace or impersonate the protected plugin; hook order/mutation interaction is tested with candidate plugins, not assumed. For every bench, record whether protected plugin was actually loaded and which hook fired.

## Non-negotiable acceptance tests

1. Render once per normal model call, no graph/model call on an observation-only event.
2. Normal vs compaction calls discriminated, or qualified OpenCode patch required.
3. Two concurrent sessions and a subagent cannot cross-contaminate context or source revisions.
4. Tool error/cancellation/provider-executed cases do not fabricate success receipts.
5. Model-visible context is ephemeral and not compounded in conversation history.
6. No context hierarchy escalation: source output remains lower-trust data.
7. An unchanged dependency closure returns semantically identical document at the same admitted source basis; cache correctness beats hit rate.
8. Known missing source/trace/clock coverage becomes UNKNOWN/INCOMPLETE, not empty/success.
9. Exact pinned OpenCode installed release and actual Python SDK installed artifacts pass; a mocked hook alone is insufficient.
10. Collect no-op plugin, Python sidecar, IPC, render and model latency separately under matched tasks and provider configuration.
