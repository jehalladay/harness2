# 01 — Native runtime, OpenCode adapter, and exact Harness² seams

**Status:** design. Source paths and hook signatures below were inspected at the pinned revisions in the program README.

## Two execution modes sharing one context semantics

**native-rf:** The model/tool loop is owned by ReactiveAgentsSDK. It uses `TurnLoopPipeline`, `TurnRenderSource`, `render_turn_context`, `ContextDocumentToModelPromptAdapter`, and already-declared Fiber effect bindings. Harness² supplies an executor/benchmark adapter and immutable candidate definitions. Do not add an agent loop to Harness². Need a real installed package test proving turn → render → model → tool → new transaction → conditional rerender → output.

**opencode-rf:** The model/tool loop is owned by pinned OpenCode. A small, protected TypeScript plugin intercepts model-visible messages and captures lifecycle events. A resident Python service manages reactive graph and rendering state; an opt-in adapter binds the plugin to the service using local process IPC. Harness² owns the candidate assembly, evaluation and rollout identity. The plugin is infrastructure, not a candidate-mutable c8 plugin.

**optional ordinary OpenCode:** Preserve upstream `oc` and `cx` benchmarks byte-compatible. Add a separately named opt-in mode/cell so changing evaluation substrate does not silently move baselines. For experiments comparing ordinary vs reactive, match model, task, resource allowance and execution environment and label different substrate/runtime costs.

## Verified OpenCode hooks and their limits

- `packages/plugin/src/index.ts` defines `experimental.chat.messages.transform(input: {}, output: {messages})`. The call in `packages/opencode/src/session/prompt.ts` runs *on each model step*, before `MessageV2.toModelMessagesEffect`. `session/compaction.ts` uses the same hook on compaction messages. It is the preferred initial dynamic context-injection seam.
- `experimental.chat.system.transform` receives `{sessionID?, model}` and `{system:string[]}` inside `session/llm.ts`. Restrict this to stable model/harness policy, **not** lower-trust environmental evidence.
- `tool.execute.before` and `tool.execute.after` expose tool identity, session/call ID and arguments/output, but their existence does not prove all attempted/failed/provider-executed tools appear. Reconcile actual message/tool-part state and use explicit UNKNOWN outcomes for missing evidence.
- `event` receives bus events for session lifecycle/health; provide a versioned event mapping and report coverage rather than claiming a complete source trace from every event.
- `experimental.session.compacting` changes compaction instructions. Dynamic evidence must remain separately re-readable after compaction. Its invocation cannot be treated as a regular inference request.
- In the pinned plugin signature, messages-transform does not receive a typed request ID, step, model ID or dedicated session ID. Recover available session identity from the validated message records and add an adapter-local sequence. An unambiguous model-call identity requires a small upstream-fork compatibility patch or plugin-API enhancement; do not fabricate it.
- OpenCode's plugin system triggers hooks in order and awaits their promises. The Python adapter must use bounded timeouts and cancellation so it cannot indefinitely stall model execution.

## Concrete fork file changes: implementation in slices

| Existing file | Extension |
|---|---|
| `harness2/core/tree.py` | additive opt-in candidate artifact surface validation; leave c1..c8 validation unchanged |
| `harness2/core/spec.py` | new opt-in execution installation mode and optional reactive candidate/evidence capabilities; preserve legacy `BenchmarkAdapter` contract |
| `harness2/substrate/base.py` | optional typed capability protocol (`materialise_candidate`, `validate_candidate`, `read_execution_receipt`); do not require old substrates to implement it |
| `harness2/substrate/opencode.py` | protected plugin installer + per-run Python service config for `oc-rf`; preserve three existing OpenCode install modes |
| `harness2/core/recursion.py` | proposal admission and candidate preflight before rollout, compare unchanged base and candidate; don't alter published 4k/3k budget identity in old modes |
| `harness2/core/context.py` | non-grader provenance/evidence refs in proposer context; raw evidence retained; reactive context assembly opt-in |
| `harness2/core/selection.py` | policy-aware evidence comparison only after deterministic admissibility; separate candidate selection from held-out grading |
| `harness2/core/library.py` | content-addressed component/assembly refs, validity bounds, evidence pointers and deprecation; existing library continues |
| `harness2/core/schedule.py` | experiment revision pins, independent matched runs, reuse and resume verification |
| `harness2/cli/_common.py` + `run_sweep.py` | explicit `rf-native` and `oc-rf` selector/experiment profiles, exact identity/pins |
| `harness2/benchmarks/reactive.py` (new) | deterministic source/task fixtures, native reactive benchmark adapter and optional opt-in real-task adapter |
| `harness2/reactive/{artifacts,validation,adapters,protocol,experiments}/` (new) | pure manifests/validation, SDK-lazy adapters, IPC/protocol clients; no redundant graph scheduler |
| `tests/test_reactive_*.py` (new) | integration and negative tests; upstream `tests/test_workflow.py` remains unchanged |

`harness2/benchmarks/jobbench.py` currently maps candidate `plugin/*.ts` to `.opencode/plugin/*.ts`. Keep that existing editable plugin behavior, but install the protected reactive adapter from a separate trusted file after candidate staging. Reject collisions, fingerprint the protected adapter separately, and check that no candidate can replace it. WorkBuddy and LAB each use different installation modes, so prove opt-in compatibility per cell rather than assuming the same prompt-file layout.

## Python render service (proposed wire, not shipped API)

Use one bounded, resident Python service per isolated rollout (with multiple sessions explicitly keyed). Start with Unix socket or stdin/stdout NDJSON with length limits and monotonic request sequence; do not start a general web server. Keep the service alive across model/tool rounds to preserve graph state, cache and observation revisions.

Request variants:
```text
hello(protocol, sdkVersions, capabilities) -> agreedVersion
open(runId, sessionId, candidateDigest, ownerEpoch) -> runtimeHandle
render(modelStepKey, modelId, sourceCut, tokenBudget, agentView) -> ContextProjection
observe(eventId, sessionId, kind, payloadRef, sourceRevision) -> Receipt
compact(sessionId, basis, policy) -> ContextProjection
snapshot(sessionId) -> SourceRevisionReceipt
close(runId, sessionId) -> CloseReceipt
```
Response carries status (READY, PENDING, UNAVAILABLE, WITHHELD, INVALID, STALE), candidate/source/graph revisions, context blocks, digest, inclusion reasons, original source authority, provenance refs, dependency witnesses, cost/counters, and diagnostics. Guard against duplicate event IDs and reuse existing SDK canonical encoding when proven compatible; do not invent a new Fiber scheduler or persistence engine in the adapter.

Call sequence in `opencode-rf`:
1. Materialize candidate and protected plugin in a uniquely isolated execution workspace; pin model, sources, plugin, SDK revisions.
2. Open a Python Fiber session and its typed graph. Pre-capture explicitly admitted source observations.
3. Each normal OpenCode model step calls messages-transform; plugin derives valid current session/step basis, requests a pinned context render, and inserts one **ephemeral**, tagged, lower-priority context segment into the model-bound messages. Do not commit that injected segment as a new permanent user instruction.
4. Source/tool events enter `observe`; validate event identity/revision, invalidate affected Fiber dependency closure; never invoke model merely because an observation changed.
5. Next step renders only necessary changes; preserve unchanged context/prefix when possible and measure the actual provider prompt-token/cache impact.
6. Compaction gets a separately identified request. Raw source observations remain available outside the compressed model history, and the next normal step reconstructs current context.
7. Emit a stable run receipt and close worker; recover from clean process crash without claiming a stateful replay if needed evidence is missing.

**Hook fallback:** If a required context basis cannot be obtained, abort that model request with a typed diagnostic. An optional block may be omitted or explicitly marked stale/unknown under its declared policy; never silently present an unavailable observation as fresh. No direct external effects from pure component renders.

## Stable future hook enhancement (only if V1 cannot carry the contract)

If actual tests show message-mutation hooks cannot safely distinguish normal inference vs compaction, session/agent identity, or request idempotence, introduce a minimal OpenCode adapter patch at the conversion seam in `packages/opencode/src/session/prompt.ts`: a typed `context.beforeModel` event with `sessionID, agent, model, stepKey, invocationKind, sourceRevisionBasis` and a return value of model-visible **context content** only. Call it once per model request, not once per tool event, and separately before compaction. Preserve upstream plugin execution and order. Keep patch small, versioned, and differential-tested against the pinned version; avoid carrying a full downstream OpenCode fork.

## Performance and concurrency

- One graph session per agent session, with per-session generations and bounded cache; do not start Python once per token or model step.
- Counters: source reads, invalidated closure, rendered components, cold/warm render duration, IPC bytes/latency, context tokens, prefix cache reuse, peak RSS, Python/TS bridge overhead and model demand count.
- Tests for concurrent sessions, reentrant hooks, timeouts, cancellation, duplicated events, compaction, subprocess exit/restart, skipped tool hooks, schema drift and provider model variants.
- Enforce lifecycle budget and backpressure. Trace/DevTools failures must not become hidden runtime effects.
- Explicitly measure when OpenCode's normal prompt-compaction or provider prompt caching changes the economic value of incremental Fiber evaluation.
