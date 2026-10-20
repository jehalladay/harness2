# Harness² + Reactive Fiber — developer entrypoint

**Status:** implementation guidance; existing upstream Harness² and pinned OpenCode remain the working baseline. No new runtime adapter is declared shipped by these documents.

## Begin here

1. [09 — source-file implementation sequence, pinned OpenCode probe and installed integration test matrix](09-developer-start-and-integration-tests.md).
2. [08 — all package-owned implementation programs and development dependencies](08-cross-repository-implementation-map.md).
3. [04 — Python rendering bridge, OpenCode hooks and request/compaction correctness](04-opencode-bridge-and-render-correctness.md).
4. [05 — immutable candidate lifecycle and typed mutation tests](05-candidate-lifecycle-and-recursive-search.md).
5. [06 — optional typed federation interfaces](06-port-federation-and-sdk-ownership.md).
6. [07 — DevTools visual evidence and Scrawl provider boundary](07-visual-feedback-and-evidence-fusion.md).
7. [Draft contracts and candidate fixture](contracts/README.md).

## First implementation slice

On a clean checkout, inventory installed SDK APIs and run the existing Harness² tests before modifications. Add an opt-in native ReactiveAgentsSDK benchmark/test adapter under the Harness² fork, then a minimal immutable component candidate/staging validator. Do not change the legacy C1–C8 tree schema's behavior or 4k/3k published accounting for existing modes.

The executable acceptance journey is a true native two-step model/tool turn: source observations are admitted to Fiber, ContextBlocks are assembled and budgeted, model demand is explicit, a tool result enters a later Fiber transaction, only affected context blocks rerender and verified task completion is independently checked. Require a failing test for an unauthorized effect and one for a source change that spuriously calls the model.

For OpenCode, perform the pinned hook-only compatibility probe before writing a production plugin; the messages-transform hook is shared by compaction and has no typed session/model-step input. If request identity cannot be recovered unambiguously, use a narrowly scoped OpenCode compatibility patch, not an inferred context switch. Preserve ephemeral lower-priority context data, provider conversion, tool history, subagent separation and prompt-cache measurements.

## Rules

- Only Reactive Fiber ecosystem integration dependencies; no SmithyWorkshopCompiler, Forge SDK/skills/runtime, or other user ecosystems. Forge is permitted externally for local builds.
- RF-CLI is an independent optional federated peer; no dependency on its CLI for internal rendering.
- Scrawl is optional external input via DevTools, never a required runtime package.
- DevTools is optional, nonauthoritative experiment inspection; no QA claims from incomplete traces.
- Explicit public port source authority, source epochs, graph generations and failure statuses.
- Separate proposer-visible evidence from independent grader/held-out verdict.
- Installed real API tests and measured baseline/regression evidence before claiming shipped features.
- Direct commits to owned default branch when implementing; no GitHub CI or PR workflow.

## Required implementation receipt

Record commit SHA, SDK/OpenCode version pins, actual executed test names/exits, fixtures/negative controls, provider/model settings or deterministic fake basis, source cut and owner identities, original/candidate artifact digests, CPU/RSS/wall and model/token cost, missing/unavailable evidence, and links to installed conformance tests. A design document or schema is not evidence that the feature works.
