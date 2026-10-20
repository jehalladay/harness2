# Recursive Reactive Harness Evolution — accepted integration program

**Status:** implementation design (2026-10-20). **Owner:** the `jehalladay/harness2` fork. No runtime feature is claimed shipped by this document.

## Scope and fixed architectural decisions

Harness² remains the recursive experimental search, proposal, comparison, adoption and experience-retention mechanism. Native Reactive Fiber SDKs own the actual context, graph, agent and query semantics. A thin OpenCode plugin is a compatibility adapter, not another reactive scheduler or agent loop.

**Allowed ecosystem dependencies:** FiberOpticsSDK, ReactiveLensesSDK, ReactiveComponentsSDK, ReactiveAgentsSDK, ReactiveFiberTSXSDK (optional), ReactiveQuerySDK (when implemented), ReactiveMCP (optional tools), and ReactiveFiberDevTools (optional experiment/LX/QA tooling). The supporting ReactiveInferenceSDK remains a dependency of the existing agent SDK when needed.

**Explicitly out of scope:** dependencies on any other user ecosystem; SmithyWorkshopCompiler and Smithy lowering; Forge SDKs, skills, QA runtime or orchestration integration; use of RF-CLI as a native harness dependency; an additional generic graph scheduler; a new model/tool loop inside Harness²; GitHub CI or PR-based workflow. Local Forge may externally build/validate outputs without being imported, called, or modeled by this program. Retain existing benchmark-only third-party test dependencies as independent optional cells; new reactive features must not require them.

RF-CLI continues unchanged as an independent tool for agents. Its future federated graph connection uses owner-controlled, typed, versioned public ports. No host shares mutable graph internals or delegates its authority merely by connecting.

## Verified source baseline

- Fork `jehalladay/harness2` equals `google-research/harness2` at Git commit `1cfe254e8d25b15f37048c3730134686abe60335` on 2026-10-20.
- The fork pins OpenCode `anomalyco/opencode@23fb5e0516c99ac04a1aa46c193efda2e1b9bb24` (1.14.18) in `harness2/config.py`. Preserve that pin until a separate compatibility decision.
- Harness² `core/tree.py` owns the c1..c8 editable string-file tree and validation; `core/recursion.py` drives proposal stages and execution; `core/context.py` builds proposer evidence; `core/selection.py` selects delivered artifacts; `core/library.py` retains experience; `core/spec.py` separates benchmark and substrate; `substrate/opencode.py` installs editable OpenCode files.
- OpenCode's pinned `packages/plugin/src/index.ts` declares `experimental.chat.messages.transform`, `experimental.chat.system.transform`, `tool.execute.before/after`, `event`, and `experimental.session.compacting`. In the pinned `session/prompt.ts`, messages-transform runs each normal model step immediately before converting to model messages; `session/compaction.ts` also invokes it. The hook input for messages-transform is empty: session identity and model-step identity are **not** supplied as typed hook parameters.
- ReactiveAgentsSDK already has a native `turn_render.py` and `turn_loop.py` where context comes from ReactiveComponents and tool outputs re-enter through Fiber transactions. Its broader all-graph-authoritative agent program is still partially a design/migration; do not claim the procedural loop has disappeared.
- ReactiveQuerySDK and ReactiveFiberDevTools were documentation/design-only when inspected. Do not import imaginary APIs.

## Program documents

1. [Runtime and OpenCode integration](01-runtime-and-opencode.md) — precise source seams, execution modes, protocol, lifecycle, compaction, migration and plugin behavior.
2. [Harness definition, mutation algebra and federation](02-definition-mutations-federation.md) — canonical candidate artifacts, admission, schema, graph transitions and public ports.
3. [Experiment evidence, DevTools and delivery](03-experiments-qa-lx-roadmap.md) — headless LX/QA lab, judge boundaries, benchmark matrix, work breakdown, acceptance gates.
4. [OpenCode bridge correctness](04-opencode-bridge-and-render-correctness.md) — pinned hook behavior, provisional plugin mode, identity/compaction gating, IPC, prompt injection boundaries.
5. [Candidate lifecycle and search](05-candidate-lifecycle-and-recursive-search.md) — typed mutations, invariants, candidate state machine and selection/holdout separation.
6. [Host-independent public ports](06-port-federation-and-sdk-ownership.md) — owner-qualified identities, snapshot/cursor semantics, SDK package boundaries and optional RF-CLI peer.
7. [Visual feedback and evidence fusion](07-visual-feedback-and-evidence-fusion.md) — generic Scrawl-first annotation evidence, time/correlation confidence and QA/LX.
8. [Draft portable protocol schemas](contracts/README.md) — candidate, definition, context projection and proposer-visible evidence schemas/fixture; not runtime implementation.
9. [Companion DevTools laboratory](https://github.com/jehalladay/ReactiveFiberDevTools/blob/main/docs/design/harness2-reactive-experiment-lab-2026-10-20.md) and [visual provider design](https://github.com/jehalladay/ReactiveFiberDevTools/blob/main/docs/design/visual-annotation-evidence-provider-2026-10-20.md).

## Existing program authority, not duplicated

- [ReactiveAgentsSDK composable agents](https://github.com/jehalladay/ReactiveAgentsSDK/tree/master/docs/programs/composable-react-fiber-agents-2026-10-16).
- [FiberOpticsResearch adaptive computation](https://github.com/jehalladay/FiberOpticsResearch/tree/main/docs/research/foa/programs/adaptive-computation-and-control-2026-10-17).
- [Federated-workspaces port calculus](https://github.com/jehalladay/FiberOpticsResearch/tree/main/docs/research/foa/programs/federated-reactive-workspaces-2026-10-18).
- [RF-CLI federation design](https://github.com/jehalladay/ReactiveFiberCLI/tree/main/docs/programs/federated-workspaces-2026-10-18): **design only**, not a shipped host adapter.
- [Certified Federated Reactive Refinement](https://github.com/jehalladay/research/tree/main/programs/certified-federated-reactive-refinement): conditions for substitution and compositional claims, not a production correctness certificate.
- [DevTools design](https://github.com/jehalladay/ReactiveFiberDevTools/tree/main/docs/design): nonauthoritative trace, evidence and analysis.
- ReactiveMCP's accepted LX requirements and capability-scoped tool contracts.

## Architectural sketch

```text
                         Harness² improvement method
                propose / validate / rollout / compare / retain
                                     |
                        candidate revision + task
                                     v
                       Substrate / Benchmark Adapter
                       /                         \
         Native ReactiveAgentsSDK            OpenCode + adapter plugin
            (owns agent loop)                 (owns agent loop)
                       \                         /
                        Python Reactive Fiber engine
                       (FiberOptics, Lenses, Components)
                                   |
                      pinned observations + render
                                   |
                          model-visible context
                                   |
                          actual execution receipts
                                   v
               DevTools evidence + QA/LX analysis (optional)
                                   |
                         Harness² experiment input

    independent RF-CLI / SDK graphs <-- public, typed ports --> harness graph
```

No runtime calls to DevTools are required for correct context rendering. DevTools attaches as an optional observer / experiment consumer, and a failed inspector does not alter model effects or candidate outcomes.

## Proof standard

Every program stage has a working installed example and a negative control. Architecture descriptions, type declarations and fictional demonstrations are not completion. Preserve upstream behavior with the feature disabled and retain benchmark separation; test actual pinned OpenCode and actual installed Python packages. Record source SHAs, run IDs, time/memory/token costs, authority and observation revisions, incomplete evidence, and reasons for refusal. Never use a grader's hidden verdict as proposer context.
