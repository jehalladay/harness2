# 03 — Experiment evidence, DevTools QA/LX, implementation work packages and gates

**Status:** accepted direction / planned implementation (2026-10-20). Do not claim any new executable package or experiment finished from this plan.

## Experiment design

Harness² must compare *executed behavior*, not only text artifacts. Preserve its existing (proposer/selector) versus (independent judge) separation and existing legacy 4k-rollout/3k-session accounting. Add new experiment modes only behind explicit names and separate budget identities. Never silently mutate old benchmark definitions or infer missing scores as zero.

The first benchmark family is isolated coding-context tasks with controlled source changes, bounded task work, repeated model/tool rounds and verifiable outputs. The second includes two independent agents, a shared observation source and controlled handoff. Federation comes later.

Each trial pins:
- harness fork/base/candidate SHA and component digests;
- OpenCode commit and adapter/plugin SHA (if used), Reactive Fiber SDK exact versions;
- model/provider parameters, environment capabilities and tokenizer accounting;
- task corpus/split, input fixture hashes, source owner revisions, random seeds;
- token/time/CPU/RSS/IPC budgets and concurrent worker count;
- evaluator protocol and independent held-out task partition.

Controls:
1. unmodified Harness²/OpenCode benchmark where applicable;
2. OpenCode with a no-op reactive plugin (adapter overhead);
3. native ReactiveAgentsSDK with fixed context graph;
4. reactive harness with fixed graph and instrumentation;
5. Harness²-synthesized reactive candidate.
Report costs and quality separately; only compare like-for-like tasks/models. OpenCode and native agent runtimes are different experimental populations until a conformance study establishes what comparisons mean.

## ExperimentEvidence contract (proposed)

```json
{
  "schema": "reactive.experiment-evidence/v1",
  "experiment_id": "exp-0001",
  "task_id": "task-001",
  "candidate_definition_digest": "sha256:...",
  "baseline_definition_digest": "sha256:...",
  "runtime": {
    "kind": "opencode-rf",
    "opencode_commit": "23fb5e0516c99ac04a1aa46c193efda2e1b9bb24",
    "adapter_digest": "sha256:...",
    "sdk_lock_digest": "sha256:..."
  },
  "basis": {
    "workload_digest": "sha256:...",
    "observation_cut_digest": "sha256:...",
    "model_config_digest": "sha256:...",
    "evaluation_protocol_digest": "sha256:..."
  },
  "evidence": {
    "execution_receipt": "artifact://run-001/receipt",
    "trace": "artifact://run-001/trace",
    "graph_diff": "artifact://run-001/graph-diff",
    "context_diff": "artifact://run-001/context-diff",
    "qa_report": "artifact://run-001/qa",
    "lx_report": "artifact://run-001/lx",
    "performance_report": "artifact://run-001/performance"
  },
  "coverage": {
    "trace_complete": false,
    "missing_events": "UNKNOWN",
    "exact_replay": false
  },
  "outcome": "DELIVERED"
}
```

Valid outcomes include NO_DELIVERABLE, REFUSED, FAILED, CANCELLED and INCOMPLETE; do not label `DELIVERED` as verified correct. Real measurements (wall, CPU, peak RSS, bytes, model tokens, context-block counts, render count, cache reuse, tool calls, wasted computation, external cost) live in a versioned `performance_report`, not untyped free text. Report missing measures explicitly; avoid fabricated zero denominators. Capture actual model/provider usage separately from token estimates.

Evidence exposure:
- `ProposerEvidence`: permitted traces/graph diffs/context explanations/QA/LX evidence, with hidden task verdicts excluded.
- `SelectorEvidence`: candidate deliverables and permitted comparisons according to the benchmark's published evaluation protocol.
- `IndependentEvaluation`: held-out correctness, quality or reference verdicts; must not enter proposer context or mutable harness files.
- `AdoptionReceipt`: selected candidate identity, changed components, actual gate outcomes, regression summary and allowed publication target.
- `ExperienceRecord`: reusable lessons linked to concrete evidence and constraints, not merely prose praise.

## DevTools headless integration

ReactiveFiberDevTools owns an independently deployable **experiment and QA/LX laboratory**, without requiring DevTools inside the model-request critical path. Follow its existing TypeScript-first source-neutral trace/query design and Python-compatible JSON/NDJSON. The core SDKs own source event truth; DevTools consumes/version-checks it, and cannot mutate live graph state.

Proposed headless APIs:
```text
importTrace(stream) -> ImportReceipt
validateTrace(traceId) -> CoverageReport
explainGraphDifference(base, candidate) -> GraphDiffReport
explainContextDifference(base, candidate) -> ContextDiffReport
compareExecutions(baseRun, candidateRun, protocol) -> ComparisonReport
runQaFixture(fixture, installedAdapters) -> QaReport
auditLxSurface(target, declaredCapabilities) -> LxReport
exportExperimentEvidence(runIds, scope) -> ExperimentEvidence
```
The JavaScript/TS and Python clients consume the same canonical protocol; do not implement two authoritative trace stores. A DevTools unavailable/failure state is a structured degradation of analysis, never changed model output or a silent PASS.

### QA laboratory (part of DevTools, not another test authority)

Headless test suites:
- Type/schema/canonicalization and graph/property tests.
- Cold versus incremental Fiber evaluation, dependency closure, component identity/state, cancellation and backpressure.
- Context inclusion/omission, token budgeting, source freshness, prompt-state consistency and prompt cache effects.
- Lens read provenance, negative reads, missing/unknown coverage.
- Agent turn tool/model effect ownership, verified stopping, replay boundaries and rerender behavior.
- Federation snapshot/cursor/epoch/revocation, hidden dependence, status, cycle and cross-authority failures.
- Counterfactual validity, incomplete traces, false-green fixture detection and observer overhead.
- Generated component conformance and mutation regression tests.

Provide deterministic fake sources, seeded random mutation sequences, reference interpreters and independent goldens where they are real and available. Build fixtures from SDK semantics, not speculative benchmark scores. Suites return `PASS | FAIL | INCOMPLETE | REFUSED`, linked to executed checks, failure examples and measured coverage.

### LX laboratory

The established reactive LX contract is: an agent can **DISCOVER, INVOKE, INSPECT and RECOVER** without a human. DevTools tests agent-facing SDK tools, context consumers and optional ReactiveMCP endpoints. Reuse ReactiveMCP capability and structured error semantics rather than redefining a second MCP transport. Measure:
- tool/port discovery correctness and schema/version self-description;
- risk and capability annotation accuracy;
- declared versus actual argument validation, bounded timeout and resource limits;
- structured refusal and suggested-action recoverability;
- discoverability of context selection/provenance/diagnostics;
- real agent workflows that attempt and recover from an induced error without manual intervention;
- no false success, hidden fallback, dropped result or invented authority.

LX and QA operate without Forge integrations/skills. Forge may be used to run local builds outside the process, and only its ordinary exit status/build products are consumed by a human or external build script—not embedded as SDKs/services.

## Quantitative comparison

Primary: verified task completion per unit cost and latency, with separate correctness/authority invariants. Secondary: component generalization across held-out tasks, context necessity/quality, reuse, dependency invalidation locality, model calls avoided, token use, time to repair, prompt-prefix reuse, resource pressure and swarm-level useful work.

No single scalar should conceal inadmissible candidates. First reject semantic/refinement, authority and effect-boundary failures; then compare an empirical multiobjective Pareto set. Confidence intervals/paired differences should reflect trial grouping and repeated tasks. Fixed replay cannot prove an unexecuted policy's actual wall-time win. Refuse exact claims when sources, feedback effects or randomization are insufficient.

## Work package DAG and practical file ownership

| ID | Owner | Delivery | Depends on | Demonstrated exit |
|---|---|---|---|---|
| H2-RF-00 | fork | pinned dependency/capability inventory + baseline golden fixture | none | unchanged legacy 4k/3k and CLI smoke |
| H2-RF-01 | fork + Agents | deterministic `rf-native` benchmark adapter, isolated installed AgentRuntime | 00 | context → model → tool → conditional rerender |
| H2-RF-02 | fork | typed RHD manifest reader, canonical refs, new opt-in surface | 00 | valid/refused artifact and old-tree parity |
| H2-RF-03 | fork + OpenCode pinned adapter | protected `oc-rf` plugin + local Python service bridge | 00,02 | per-inference context, compaction, tool-result reentry |
| H2-RF-04 | DevTools + SDK event producers | trace ingest/query + experiment evidence export | 01/03 | complete/incomplete causal analysis, off/on parity |
| H2-RF-05 | DevTools | QA/LX suites with executable oracles and negative fixtures | 04 | reproducible machine-readable PASS/FAIL/INCOMPLETE |
| H2-RF-06 | fork | typed mutation proposal, validation, isolated staging and replay | 01,02,05 | synthesize/test/reject/retain pure Python component |
| H2-RF-07 | fork | comparison/adoption/library metadata/evidence basis | 04,06 | independent held-out task win with full run receipts |
| H2-RF-08 | SDKs | cross-SDK canonical exported port contract, local snapshot importer | 01,02 | two independent native graphs exchange typed value |
| H2-RF-09 | Agents + fork | opt-in behavior/feature plan mutations, state admission | 06,07 | independent measured policy experiment |
| H2-RF-10 | SDK hosts | snapshot+cursor subscribe, gap/epoch/cycle tests | 08 | correct invalidation and offline replay |
| H2-RF-11 | RF-CLI optional | approved exported/imported adapter using canonical port contract | 08,10 | separate installed cross-host test (not native dependency) |
| H2-RF-12 | fork + DevTools | swarm/coordination experiments and meta search proposals | 07,09 | independent group-level matched-budget improvement |

Start H2-RF-00,01,02 in parallel where dependencies permit; build the first OpenCode bridge before considering a pinned OpenCode API patch. Do not block working native execution on DevTools full UI, QuerySDK implementation, TSX hosting, remote federation or RF-CLI.

## First four vertical-cut acceptance journeys

**J1 Native agent:** Two-step fake model requests a tool; tool result changes one Lens/Fiber source; affected context rerenders; unchanged component is reused. Verify exact rendered prompt snapshot, effect count and closing receipts. Negative: effect attempted inside pure render refuses.

**J2 Pinned OpenCode:** Fixed harness with zero dynamic context vs no-op plugin vs pinned reactive plugin. Each model step receives one ephemeral correctly scoped context block. Tool-result observation changes next step, compaction does not duplicate or elevate evidence. Negative: killed Python process/refused required observation cannot become an unlabelled success.

**J3 Recursive synthesis:** From a pinned base, candidate adds a Python diagnostic component, typed port connection and context selection policy. Reject schema-invalid and unauthorized proposals; run base/candidate on same frozen tasks, then independent held-out tasks; retain only with full reasons. Negative: leaked held-out verdict in proposer evidence is refused.

**J4 Independent graph port:** Native producer publishes typed output; another native agent imports snapshot and composes it into context; unchanged original replay succeeds with producer offline. Negative: different owner-instance epoch or missing query coverage never becomes READY/TRUE without witness.

## Release gates

- G0: baseline pin/current code audit; legacy tests identical with RF disabled.
- G1: typed manifest/candidate validator and exact mismatch refusals.
- G2: native installed-agent journey with receipts and resource measurements.
- G3: pinned OpenCode plugin install/transport/compaction parity and failure tests.
- G4: DevTools trace + QA/LX headless analysis accurate under dropped events.
- G5: actual recursive component synthesis measured on held-out work, including regression cases.
- G6: federation native/native pinned snapshot and cold evaluation equivalence.
- G7: optional subscriptions/TSX/RF-CLI/behavior/swarm/meta paths individually gated by their actual implementation readiness.

Build with each project's existing local toolchain and optionally external Forge. Do not add GitHub CI or PR automation. No performance win is asserted before paired measurement; all implementation reports include actual commands, source revisions and test exits.
