# 05 — Candidate lifecycle, mutation algebra, executable search and adoption

**Status:** implementation design, 2026-10-20. Builds on [02](02-definition-mutations-federation.md) and [03](03-experiments-qa-lx-roadmap.md).

## Compatibility first

The current Harness² `Tree` is `dict[str,str]` with C1..C8 editable files and `SurfaceSpec` path validators (`core/tree.py`). Existing `BenchmarkAdapter` in `core/spec.py` owns task input, execute, deliverables, trajectory and independent `score`. Current `core/ledger.py` demands R=4k rollouts and P=3k proposer sessions under published modes. Preserve these as-is. Introduce `reactive-v1` candidate cells and a separate explicit budgeting/version identity. Never silently rewrite old benchmark grading, source tree layout, selectors or counts.

## Identity distinctions

- **Definition**: immutable declarative program + content-addressed artifacts. May reference pinned pure Python implementations; does not serialize Python closures.
- **Candidate**: base definition plus a validated transformation patch, expected typed ports and stated performance hypothesis.
- **Instance**: mounted mutable runtime state bound to one graph generation and authority scope.
- **Observation cut**: admitted source owner revisions/scope/status and any proven causal closure.
- **Execution**: concrete task rollout pinned to candidate digest, model/provider, environment and source cut (or admitted live source policy).
- **Evidence**: externally observed facts and completeness/provenance, not automatically a reward.
- **Adoption**: explicit promotion of a candidate definition under known task family and validation scope, never alteration of historical execution.

## Candidate mutations as typed change operations

Every patch has: `op_id`, `base_revision`, `preconditions`, `target_ref`, `new_artifact_digest`, `read_set`, `declared_effects`, `expected_outputs`, `scope`, `hypothesis`, `validation_profile`.

Proposed operations:
- `component.create`, `component.replace`, `component.retire`, `component.compose`;
- `lens.bind`, `lens.unbind`, `observation.policy`;
- `graph.connect`, `graph.disconnect`, `graph.replace_fragment`;
- `context.view`, `context.select`, `context.order`, `context.budget`;
- `query.define`, `query.use` (only after QuerySDK ships and conformance exists);
- `agent.demand_policy`, `agent.tool_policy`, `agent.verify_policy`, `agent.delegation_policy`;
- `swarm.role_projection`, `swarm.coordination_policy`;
- `federation.export`, `federation.import`, `federation.unbind`;
- `improver.evidence_policy`, `improver.proposal_policy`, `improver.experiment_budget` (meta loop, separate qualification).

Do not represent a whole graph rewrite as one opaque script if the proposed change can be expressed as typed operations. Executable candidate files are allowed but declared/validated at an exact digest. No candidate may grant an authority capability to itself; an import/definition change is not an effect permission change.

## Algebra and laws to test

A patch `p` acts on an admitted base `H` as partial function `p : H ⇀ H'`. Preconditions, type checks and authority restrictions determine definedness.

- **Identity:** applying a no-op patch is observationally equivalent for every declared observer when state/source cuts are unchanged; check against cold reference evaluator.
- **Sequential composition:** `q ∘ p` is legal only when `q`'s preconditions hold on `p(H)`; preserve intermediate receipts.
- **Independent commutation:** disjoint write sets alone are insufficient: also check dynamic dependency reads, authority and shared resource effects. Otherwise test both orders or reject claims of commutativity.
- **Retirement:** removing a component must account for all consumers, leases, outstanding attempts and derived context blocks.
- **Substitution:** output equality at one snapshot is not sufficient for stateful/temporal equivalence; use declared observer and transition/trace refinement assumptions from FROE/CFRR where applicable.
- **Port evolution:** schema-compatible value substitution is not sufficient if status, freshness, provenance, coverage or sensitivity changed in observer-visible ways.

## Candidate state machine

```text
PROPOSED
  | parse/schema/manifest/precondition checks
  v
COMPILED --static or type failure--> REFUSED
  | pure reference fixtures + deterministic conformance
  v
ADMITTED --runtime effect/authority denial--> REFUSED
  | cold reference and captured source replay
  v
PROBED --failed fidelity/cost gate--> REJECTED
  | matched or randomized actual execution
  v
ASSESSED --no demonstrated benefit--> RETAIN_AS_NEGATIVE_EXPERIENCE
  | independent adoption criteria
  v
PROMOTED --later regression/expiry--> DEPRECATED
```

A pass means only that the named check ran and passed at the named revision. The system can retain informative failures, with negative controls and counterexamples, without promoting the associated candidate.

## Evaluation protocol

Candidate builder `proposer` sees task instructions, permitted historical components, prior non-grader evidence and admissible mutation grammar. It returns a typed patch and source artifacts.

Admissibility checker (deterministic) verifies integrity, source pin, graph topology, type/port compatibility, budgets, effect requirements, allowed resource declarations, installed package API availability and identity. For code generation, use ordinary isolated execution and known read/effect boundaries; do not infer purity from a string label.

Probe runner:
1. Compile fresh base and candidate from content-addressed artifacts.
2. Evaluate with identical frozen source snapshots where feasible, including dirty/source-change/retraction fixtures. Cold reference result is independent of incremental graph evaluation.
3. Compare render blocks, effects, provenance, error/status, context cost and p95 latency; record trace coverage.
4. Only qualified candidates move to paired actual work; randomize seed/task order where appropriate and isolate evaluation material.
5. Run held-out tests with independent grader access. Selector sees only expressly allowed candidate evidence; judge-only private outcome remains hidden from proposer.
6. Promote only under a declared utility/Pareto rule and correct bounds (time, task family, model, source authority and runtime revision).

Legacy Harness² `selection.py` first filters `ok && delivered`. Reactive search must preserve that filter and add correctness/admissibility gates before cost/utility. Do not treat missing score as zero; mark UNKNOWN and rerun or exclude according to policy.

## Search horizons

**Level A (first ship):** pure components, typed edges and context assembly; cheap deterministic fixtures, held-out task outcomes.

**Level B:** observers, QuerySDK views when implemented, context/retrieval budget control; simulate and execute meaningful source churn.

**Level C:** verified tool/model demand, retries, independent verification and delegation policies, within existing AgentSDK effect contracts.

**Level D:** multi-agent policy compositions and public federation ports; matched-budget group success, makespan and redundancy.

**Level E:** experiment-planning policy over the archive: resource-aware expected improvement and exploration. Do not permit self-grading or evaluator mutation.

Search selection should use a Pareto set over verified quality, tokens, p95 latency, CPU/RSS, total experiment compute and graph complexity rather than promising a universal scalar. Explicit hard validity and external authority constraints precede optimization.

## Provenance and evidence records

Minimum artifact set per run: `candidate.json`, `admission.json`, `source-cut.json`, `run.json`, `trace-manifest.json`, `comparison.json`, `judge-ref.json` (only an opaque restricted reference in proposer scope), and `adoption.json`. Each records version, source digest, producer/owner, authorizing scope, run identity and missing evidence. No secrets or hidden benchmark targets in proposer artifacts.

## Work slices in Harness² fork

- CAND-00: package schemas and canonical JSON fixture vectors; no runtime code.
- CAND-01: candidate read/write/validate module and `reactive/` opt-in tree paths; preserve original `Tree` behavior.
- CAND-02: install manifest and isolated pure fixture runner.
- CAND-03: `ReactiveSubstrateCapability` and `ReactiveBenchmarkAdapter` optional protocols.
- CAND-04: observation/graph/render records from installed ReactiveAgentsSDK.
- CAND-05: paired-run study and independent held-out scoring.
- CAND-06: library additions storing reusable graph fragments and bounded evidence, with versioned reuse/expiry.
- CAND-07: behavior/swarm/meta extensions gated separately.

Each milestone must include a positive and a deliberate counterexample fixture. Feature-disabled tests continue to exercise ordinary `oc`/`cx` Harness² with no reactive import.
