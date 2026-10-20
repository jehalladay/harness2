# 06 — Portable federation ports and Reactive Fiber package ownership

**Status:** target design, not implemented by Harness² or RF-CLI at the time of writing. 2026-10-20.

## Purpose and excluded dependencies

The native Harness² runtime must *not* depend on RF-CLI, and no cross-graph link should require MCP tool calls inside the context hot path. The public-port program applies to pairs of independent SDK-native agent graphs, and to SDK-native ↔ RF-CLI pairs only when an RF-CLI adapter later ships. The design is rooted in the existing [FROE public-port calculus](https://github.com/jehalladay/FiberOpticsResearch/tree/main/docs/research/foa/programs/federated-reactive-workspaces-2026-10-18) and [RF-CLI federation program](https://github.com/jehalladay/ReactiveFiberCLI/tree/main/docs/programs/federated-workspaces-2026-10-18). Do not mint conflicting semantics.

Smithy-written port contracts remain a *future optional authoring language*, not a dependency for this implementation stage. No SmithyWorkshopCompiler analysis, compiler runtime, integration or validation gate is included.

## Ownership model

An exported port belongs to a source authority. Its consumer owns an import alias, local admission, accepted snapshots and downstream graph policy, **not** the source's internal nodes, effects or hooks.

```text
 SDK graph A                       SDK graph B / RF-CLI host B
 -----------                        -----------------------
 imported source input       <--   published typed export
 local acceptance receipt          owner-controlled revision
 downstream Fiber consumers        snapshot/cursor + operations
```

Canonical owner identity = `{owner_kind, owner_id, owner_instance_id}`. Canonical resource identity additionally includes `{resource_kind, export_id, definition_digest, value_schema_digest}`. An observation adds `{source_epoch, source_revision, value_digest, status, provenance_digest?, coverage?, cursor?}`. Alias/local filesystem path must never substitute for owner/instance identity. Opaque source revisions from distinct owners are not totally ordered.

## Public port kinds

- `value_output`: typed published data, not an inference effect.
- `view_output`: committed rendered context/view artifact at a specified definition, budget and source basis.
- `lens_value`: owner-resolved lens observation at an explicit revision with source/freshness evidence.
- `query_view`: maintained QuerySDK result when the SDK exists, retaining K3 truth, readiness, scope and coverage.
- `graph_projection`: bounded visible dependency/structural projection; not arbitrary private graph access.
- `definition`: portable immutable artifact. Importing a definition compiles a *new* local state machine; it does not import source-owned hook state.
- `state_snapshot`: explicitly published read-only state record; no write authority.
- `evaluate` and `effect_operation`: separate, source-admitted requests, never triggered as a side-effect of invalidation.

## Port operations and their separation

Operations: `describe` (metadata, may be sensitivity-scoped), `read` (owner-committed immutable value), `subscribe` (notification stream), `evaluate` (explicit producer-side pure/effect evaluation as declared), `invoke` (effect, separately authorized). `read` alone never grants `evaluate` or `invoke`. A subscription notification can mark dirty; it cannot invoke tools/models. Source owner owns effect receipts; imported values do not transfer that authority.

Example abstract export (proposed):

```json
{
  "schema": "reactive.federation-export/v1",
  "owner": {"kind":"agent-harness","id":"harness-A","instance":"instance-17"},
  "port": {"id":"build-health","kind":"value_output",
            "definition_digest":"sha256:...","value_schema_digest":"sha256:..."},
  "operations": ["describe","read","subscribe"],
  "consistency": ["PINNED","LATEST_ACCEPTED"],
  "sensitivity": "internal",
  "dependency_summary": {"kind":"bounded","digest":"sha256:..."},
  "capability_policy_ref": "policy:build-health-reader"
}
```

A host-local binding chooses locator, expected owner/instance, scope, capabilities and accepted freshness policy. It must not contain credentials in the exported manifest.

## Import state machine and consistency

Proposed state progression:

```text
UNBOUND -> RESOLVING -> BOUND -> (READY|PENDING|UNAVAILABLE|WITHHELD|INVALID|STALE)
READY --notification--> DIRTY --owner snapshot read/admit--> READY
DIRTY --source inaccessible--> STALE if allowed, otherwise UNAVAILABLE
cursor gap/epoch mismatch --> RESYNC_REQUIRED --> RESOLVING
revoke --> WITHHELD
```

- `PINNED`: replay against exact immutable authorized source snapshot; no owner contact in replay.
- `LATEST_ACCEPTED`: most recently *admitted* source revision in one owner stream; may be stale.
- `CAUSAL` (later): complete causally closed cut across owner dependencies with checkable provenance; not a vector of arbitrarily latest revisions.

Owner's atomic snapshot+cursor relation and consumer's atomic value+cursor acceptance are essential for live subscription correctness. Duplicate, reordered or lost messages cannot silently regress accepted state. Import commit occurs before downstream invalidation. Source status/coverage/provenance are readable semantics where exposed; equal payload hashes do not imply equal observations.

## Explicit cycles and delayed feedback

Compose graph boundaries logically, never by merging their live mutable graphs. Reject instantaneous cross-host cycles unless an admitted delayed/fixed-point protocol with proven conditions exists. Conservative `dependency_unknown` blocks strong guarantees rather than assuming a safe acyclic closure. Never infer general federation liveness or fairness from local DAG correctness.

## Proposed SDK ownership

- **FiberOpticsSDK**: common typed public port descriptors, graph source/admission primitives, codec/conformance; avoid adding a network transport into graph core.
- **ReactiveLensesSDK**: owner source binding, observable status/freshness/coverage, import value adapter and admitted snapshots.
- **ReactiveComponentsSDK**: consume imported values in ordinary component dependencies and context inclusion explanations; no remote I/O in pure render.
- **ReactiveAgentsSDK**: expose graph/public ports, per-agent binding, authority-aware lifecycle, demand and state transitions.
- **ReactiveQuerySDK**: authoritative query truth/coverage/retractions once shipped.
- **ReactiveFiberTSXSDK**: optional `useRemoteResource`/port hooks on same semantics; not a separate cache or protocol.
- **ReactiveMCP**: optional capability-scoped inspection/invocation adapter, not required for data-plane imports.
- **ReactiveFiberDevTools**: causal inspector of export/read/admission/invalidations and replay; cannot synthesize source truth.
- **RF-CLI**: optional independent source/consumer adapter only when its federation program is implemented.
- **Harness² fork**: version and mutate *definitions* of export/import declarations as candidate artifacts, and run comparative experiments. No federation protocol fork.

## Verification gates

F0 static types/codec/identities and forbidden cross-owner revision comparison.
F1 SDK B exports a typed value; SDK A binds/pins/consumes it with a real component context render; source B unavailable during offline replay.
F2 duplicate, cursor gap, restart, epoch mismatch, schema drift, sensitivity mismatch and revocation have distinct typed diagnostics.
F3 subscription and invalidation alter only affected A dependencies after a committed accepted snapshot; observer and original owner receipts remain separate.
F4 explicitly declared evaluation operation cannot be reached from ordinary read/subscription.
F5 graph composition cycle, unknown-dependency and stateful substitution counterexamples.
F6 RF-CLI ↔ SDK integration when RF-CLI adapter ships; RF-CLI remains unaffected when unconfigured.
F7 (deferred) Smithy-authored port model lowers to byte-equivalent canonical descriptor under a future independent authoring compiler. No current runtime/implementation dependence on that option.
