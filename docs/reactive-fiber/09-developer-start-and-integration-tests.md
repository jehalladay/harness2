# 09 — Harness² fork developer guide and executable vertical-cut backlog

**Status:** exact engineering plan, not a runnable quickstart. Extends [01 OpenCode](01-runtime-and-opencode.md), [04 bridge](04-opencode-bridge-and-render-correctness.md), [05 candidate search](05-candidate-lifecycle-and-recursive-search.md), [08 package map](08-cross-repository-implementation-map.md).

## Working baseline

Fork `jehalladay/harness2` had identical upstream tip `1cfe254e8d25b15f37048c3730134686abe60335` on 2026-10-20 before this documentation program. `harness2/config.py` pins OpenCode `23fb5e0516c99ac04a1aa46c193efda2e1b9bb24`. Pin exact model/SDK/provider versions per test; do not upgrade OpenCode incidentally.

Existing code paths:
- `harness2/core/tree.py`: c1–c8 editable file tree, allowlist, validation and staging. Preserve legacy behavior.
- `harness2/core/recursion.py`: proposal/rollout stages and published budgets.
- `harness2/core/spec.py`: `BenchSpec`, `BenchmarkAdapter` with task/execute/score separation.
- `harness2/core/context.py`: task evidence tree; grader leak checks.
- `harness2/core/selection.py`: candidate anonymization and selection.
- `harness2/core/ledger.py`: legacy R=4k rollouts, P=3k proposer sessions.
- `harness2/core/library.py`: adopted experience store.
- `harness2/substrate/base.py`: Substrate protocol and Install.
- `harness2/substrate/opencode.py`: OpenCode materialization, launcher, event stream.
- `harness2/benchmarks/jobbench.py`, `workbuddy.py`, `lab.py`: **different** OpenCode installation modes. Don't assume universal `.opencode/plugin` layout without checking bench.
- `harness2/cli/_common.py` and `run_sweep.py`: current benchmark/substrate selection; current `oc`/`cx` remain unchanged.

## Proposed implementation layout (none of these modules is shipped yet)

```text
harness2/reactive/
  contracts/          # manifest/patch/context/evidence validators; schema versions
  candidates/         # typed mutations, per-op preconditions and staging
  runtime/            # optional native AgentsSDK external execution adapter
  bridge/             # Python resident render service, local IPC, source observation receipts
  opencode/           # protected TypeScript plugin source and patch qualification fixtures
  experiments/        # run grouping, comparison basis and evidence scopes
harness2/benchmarks/reactive.py
tests/reactive/
  fixtures/
  test_contracts.py
  test_native.py
  test_opencode_plugin.py
  test_candidate_mutation.py
  test_evidence_isolation.py
```

Prefer lazy imports so old `harness2` invocations without Reactive Fiber installed still work. The schema fixture directory `docs/reactive-fiber/contracts/` is **draft** and must be reconciled with actual SDK canonical typing before a new distribution public API ships.

## Development slices

### H2-00: baseline

Run `tests/test_workflow.py` and a harmless dry-run of existing CLI under the pinned environment. Record versions, exit codes and disabled-mode behavior. No GitHub CI. If a legacy benchmark needs external credentials, report NOT_RUN and test isolated internal paths; do not falsify baseline success.

### H2-01: immutable candidate files

Add additive candidate artifact validation on a separately selected reactive surface. Ensure C1–C8 users, `Tree = dict[str,str]` and old allowed paths remain unchanged. Check `reactive/harness.json` with an exact major version, artifact SHA, simple bounded graph and component declarations. Do not load arbitrary candidate Python merely by parsing JSON.

Negative tests: path traversal, unknown file, missing declared code digest, duplicate port, implicit tool capability request, unsupported SDK version and borrowed held-out evaluator file.

### H2-02: native reactive benchmark

Implement opt-in `ReactiveBenchmarkAdapter` using a deterministic fake model+tool environment and real installed ReactiveAgentsSDK. `execute` returns a source-backed run directory, `trajectory` maps real events into the historical grammar, and `score` stays independent. The SDK owns the agent loop. Candidate A may have changed context assembly; do not claim it changed orchestration without measured behavior.

J1 expected: task → first render → exactly one admitted inference demand → tool intent/approval/attempt → tool observation transaction → affected context recomputation → second admitted inference → externally verified task conclusion. Test missing tool receipt and source change without inference-demand controls.

### H2-03: OpenCode pinned compatibility probe

First run real OpenCode with a **no-op plugin**, then a temporary synthetic context plugin with mock sidecar; inspect the precise incoming `messages.transform` shape in normal execution and compaction. Pinned OpenCode also invokes that hook in `session/compaction.ts`; the plugin input lacks explicit request-kind identity.

Only proceed to a real Python Fiber bridge if normal/compaction/session separation, model-visible role and ephemeral injection are proven. Otherwise create a tiny, independently versioned OpenCode patch at the pre-model conversion seam with explicit `sessionID`, model call ID, request kind, model ID, source-cut basis and a lower-priority data channel. Patch must not replace agent loop. Use actual pinned CLI execution, not only a TypeScript unit test.

### H2-04: resident Python engine

Single process per isolated rollout, multiplex explicitly keyed session instances; local bounded stdio/socket protocol with `hello/open/observe/render/compact/snapshot/close`. Call actual FiberOptics+Lenses+Components APIs, retaining graph and component state across tool/model turns. Source notifications mark dirty; source snapshot admission commits before downstream invalidation. No model request on raw source change.

Receipt: run/session/owner epoch, graph generation, definition digest, source-cut ref, context/provenance/inclusion summary, receipt ID, elapsed/cost counters, READY/PENDING/etc status and typed errors. No hot-path DevTools import or RF-CLI invocation.

### H2-05: proposer, experiment and adoption

Proposer edits an allowlisted typed patch/artifacts, not the protected adapter plugin. A deterministic validator checks source/authority/schema/effect boundaries; test captured observations and cold-render oracle; then paired live task runs. Selection sees only authorized candidate artifacts and non-grader evidence. Scoring remains separate, with held-out task split never stored in proposer context.

Keep published existing 4k/3k identity; any new experiment count/budget profile has a **distinct versioned name** and measurable executed calls. Adoption records exactly which independent evidence gates passed, not a subjective LLM summary.

### H2-06: optional DevTools and visual evidence

Emit bounded trace source receipts. Attach source-neutral DevTools `ExperimentEvidence` refs to the proposer evidence tree only when scoped permitted; visual Scrawl evidence is routed through DevTools, never into privileged system instructions or hidden grader state. DevTools unavailable => evidence analysis unavailable, **not** model execution failure.

### H2-07: after first recursive component result

Only then enable behavior policy mutation, native SDK graph federation, optional RF-CLI peer, TSX/QuerySDK, visual UI mapping, swarm policy experiments and meta-selection. Gate each on its own installed SDK implementation.

## Local validation matrix

| Check | Pass condition | Deliberate failure |
|---|---|---|
| Disabled compatibility | old Harness² behavior unchanged | reactive imports missing |
| Candidate parse | canonical known-major schema | wrong major, undeclared artifact |
| Native turn | real Fiber source change and conditional rerender | source change causes unauthorised model call |
| OpenCode model boundary | single correct scoped ephemeral projection per inference | compaction confusion, duplicate inserted context |
| Tool effects | auth→attempt→receipt→postcondition | timeout incorrectly marked success |
| Graph state | candidate mutation version fenced | stale result adopted after generation change |
| Context | provenance and budget inclusion correct | same text with different visibility reused |
| Experiment | matched trials and hidden judge isolation | evaluator leakage via trace/deliverables |
| Observer | DevTools off/on output-effect parity | dropped events misreported complete |
| Federation | owner-qualified pinned value replay | wrong owner epoch, implicit remote tool invocation |

Every test suite needs a valid positive fixture, a contradictory/failure fixture, actual execution count, source/config SHA and wall/CPU/RSS when meaningful. Documentation must not present hypothetical commands as if run.

## Developer handoff checklist

- [ ] Exact installed SDK API inventory; no invented symbols.
- [ ] Additive and opt-in plugin/adapter; upstream benchmarks unchanged.
- [ ] Pinned OpenCode identity/compaction probe first.
- [ ] First native 2-turn source → context → model → tool → source experiment.
- [ ] Typed candidate/source digests with no autonomous authority gain.
- [ ] Trace completeness, counterexamples, held-out scope and cost controls.
- [ ] Clean installed client and reproducible local build record; external Forge only.
- [ ] Direct commits to owned main when asked to implement; no CI/PR.
