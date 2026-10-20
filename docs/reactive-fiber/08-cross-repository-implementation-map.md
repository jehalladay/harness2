# 08 — Cross-repository implementation ownership and critical-path schedule

**Status:** package implementation coordination plan. **Date:** 2026-10-20. Existing [01–07](README.md) plus [draft contracts](contracts/README.md) remain authoritative on detailed semantics. This cross-repository index coordinates additive work; it does not declare implementation success.

## Accepted constraints

Only Reactive Fiber ecosystem packages are runtime/integration dependencies. Harness² fork is the recursive experiment owner; OpenCode is an external executor which Harness² already pins. No SmithyWorkshopCompiler integration, Forge SDK/QA skill/runtime dependency, or other user ecosystem. Forge may run local build commands externally. RF-CLI remains optional independent graph host; Scrawl is optional external DevTools visual-evidence *input*. No GitHub CI; write to owned repository default branch.

## Package programs and developer handoffs

| Repo/branch | Contract ownership | Developer program |
|---|---|---|
| [Harness² fork](https://github.com/jehalladay/harness2) main | reactive substrate, OpenCode adapter/plugin, candidate mutation, experiment/selection | [09 developer guide](09-developer-start-and-integration-tests.md) |
| [FiberOpticsSDK](https://github.com/jehalladay/FiberOpticsSDK) master | typed ports, public value publication, admitted graph source/invalidation, trace producer | [Program](https://github.com/jehalladay/FiberOpticsSDK/tree/master/docs/programs/harness2-reactive-evolution-2026-10-20) |
| [ReactiveLensesSDK](https://github.com/jehalladay/ReactiveLensesSDK) master | owner/source observation, freshness/provenance, snapshot import | [Program](https://github.com/jehalladay/ReactiveLensesSDK/tree/master/docs/programs/harness2-reactive-evolution-2026-10-20) |
| [ReactiveComponentsSDK](https://github.com/jehalladay/ReactiveComponentsSDK) master | component artifacts, ContextDocument/Block, occurrence budgeting, render receipts | [Program](https://github.com/jehalladay/ReactiveComponentsSDK/tree/master/docs/programs/harness2-reactive-evolution-2026-10-20) |
| [ReactiveAgentsSDK](https://github.com/jehalladay/ReactiveAgentsSDK) master | native executor, model/tool demand/effects, graph transitions, agent exports | [Program](https://github.com/jehalladay/ReactiveAgentsSDK/tree/master/docs/programs/harness2-reactive-evolution-2026-10-20) |
| [ReactiveInferenceSDK](https://github.com/jehalladay/ReactiveInferenceSDK) master | provider/model capability snapshots, token/cache/inference demand accounting | [Program](https://github.com/jehalladay/ReactiveInferenceSDK/tree/master/docs/programs/harness2-reactive-evolution-2026-10-20) |
| [ReactiveQuerySDK](https://github.com/jehalladay/ReactiveQuerySDK) main | K3 query truth, negative dependencies, coverage proof, query view wire | [Program](https://github.com/jehalladay/ReactiveQuerySDK/tree/main/docs/program/harness2-reactive-evolution-2026-10-20) |
| [ReactiveFiberTSXSDK](https://github.com/jehalladay/ReactiveFiberTSXSDK) main | optional TSX candidate consumer, admitted resource hooks, UI→Fiber mapping | [Program](https://github.com/jehalladay/ReactiveFiberTSXSDK/tree/main/docs/programs/harness2-reactive-evolution-2026-10-20) |
| [ReactiveFiberDevTools](https://github.com/jehalladay/ReactiveFiberDevTools) main | trace ingest/query, QA/LX, comparison reports, Scrawl-first visual evidence | [Program](https://github.com/jehalladay/ReactiveFiberDevTools/tree/main/docs/design/harness2-recursive-reactive-laboratory-2026-10-20) |
| [ReactiveMCP](https://github.com/jehalladay/ReactiveMCP) main | optional agent-accessible inspection and bounded tools | [Program](https://github.com/jehalladay/ReactiveMCP/tree/main/docs/programs/harness2-reactive-evolution-2026-10-20) |
| [ReactiveFiberCLI](https://github.com/jehalladay/ReactiveFiberCLI) main | optional independent federation host adapter | [Program](https://github.com/jehalladay/ReactiveFiberCLI/tree/main/docs/programs/harness2-federated-peer-2026-10-20) |
| [FiberOpticsResearch](https://github.com/jehalladay/FiberOpticsResearch) main | mathematical interface and theorem→conformance obligations | [Program](https://github.com/jehalladay/FiberOpticsResearch/tree/main/docs/research/foa/programs/recursive-reactive-harness-evolution-2026-10-20) |
| [research](https://github.com/jehalladay/research) main | independently checkable recursive improvement and evidence sufficiency research | [Program](https://github.com/jehalladay/research/tree/main/programs/recursive-reactive-harness-evolution) |

## Dependency DAG — do not make every package a blocker

```text
Existing installed SDK/API inventory
      +----> FiberOptics typed port schema -------------> Lenses import
      |                                                      |
      +----> Components render receipts ----+                +----> SDK/SDK federation
      |                                      |                            |
      +----> Agents native execution --------+----> Harness² native        +---> optional RF-CLI peer
      |                                                   |
      +----> Harness² candidate manifest ----------------+----> recursive context synthesis
      |                                                   |
      +----> OpenCode pinned hook probe + Python bridge -+----> optional OpenCode experiments
      |
      +----> producer trace schemas -> DevTools headless ingest + QA/LX -> experiment evidence

After native verified outcome:
  QuerySDK when its own gates pass -> query-aware context
  TSX hosting when installed qualified -> optional TSX candidate/visual witness
  Scrawl offline parser -> optional visual evidence/QA, without gating native execution
  MCP -> optional evidence/port inspection
  Agent behavior/swarm/meta -> later independently budgeted experiments
```

No requirement to complete QuerySDK, federation, TSX, RF-CLI, MCP, Scrawl, graphical DevTools or theoretical mechanization before first native Python component optimization.

## Cross-package artifacts that require a single owner

| Artifact | Owner | Consumers |
|---|---|---|
| Owner-qualified port/value publication/status/graph receipts | FiberOpticsSDK | Lenses, Agents, Components, Query/TSX hosts, federation peers, DevTools |
| Source freshness/accepted owner snapshot | ReactiveLensesSDK | FiberOptics source, Components, Agents |
| Component definition + ContextDocument selection receipt | ReactiveComponentsSDK | Agents, Harness² execution, DevTools |
| Agent definition/instance/demand and effect receipts | ReactiveAgentsSDK | Harness² native adapter, DevTools |
| Model capability/usage observations | ReactiveInferenceSDK | Agents, context budgets, DevTools |
| K3 QueryView / source completeness | ReactiveQuerySDK | Components, Agents, TSX, DevTools |
| Executable Harness Definition/Candidate Patch/ExperimentEvidence envelope | Harness² fork | Harness² selection/adoption; DevTools reads permitted refs |
| Source-neutral trace/visual evidence/QA/LX comparison | DevTools | Harness², optional MCP, human/TUI |
| Optional agent-visible MCP tool protocol | ReactiveMCP | agent clients only |
| RF-CLI workspace publication/authorization | RF-CLI | RF-CLI users; optional federated imported peer |

A descriptor claiming to be from another owner must be verified against that owner's source protocol and actual installed code; don't clone protocol logic to simplify one experiment. Never grant tool permission because a candidate declared it.

## Integrated work and QA gates

**I0 Inventory:** freeze per-repo source HEAD, installed exports, manifests and pinned test commands. Identify which planned APIs are nonexistent; baseline old Harness² modes untouched.

**I1 Native single-agent:** installed Python 2-round render/model/tool/observation/re-render pipeline and cold context oracle. Rejected unauthorized tool and state leak controls.

**I2 Candidate mutation:** parse and validate immutable candidate artifacts, stage isolated definitions, compile with actual SDK installed, never execute unapproved source from JSON alone. Verified base vs candidate render fixture.

**I3 OpenCode compatibility:** pin 1.14.18 OpenCode, no-op plugin control, real plugin+Python sidecar, normal/compaction/subagent identity. If hook lacks authoritative request identity, use explicitly scoped minimal patch. No context accumulation/privilege escalation.

**I4 DevTools:** source-trace ingest, multi-parent/gap honesty, QA negative fixture, one LX recovery journey, independent experiment evidence output with no grader leakage.

**I5 Recursive improvement:** proposer synthesis of a Python context component, base/candidate paired task execution, held-out verified quality, resource measurements, documented retained/rejected artifact.

**I6 Native federation:** owner-qualified value exported SDK B -> imported as local SDK A source, producer offline pinned replay, epoch/cursor/unknown-dependency negative controls.

**I7 Optional tracks:** QuerySDK, TSX/Scrawl visual feedback, MCP, RF-CLI peer, behavior/swarm, learned experiment allocation. Each must name actual prerequisites and tests; no all-up deployment dependency.

## Definition of done

An item is finished only after source implementation, clean installed consumer, independent positive/negative tests, exact owner/source/evaluator revisions, measured resources and a reproducible source-linked report. A design document, a JSON Schema or a synthetic counterfactual is *not* a shipped runtime feature or measured productivity win.
