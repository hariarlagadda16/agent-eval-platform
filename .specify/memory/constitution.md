# agent-eval-platform Constitution

## Core Principles

### I. Reproducible Evaluation
Every evaluation MUST be re-runnable from recorded inputs: task suite, agent
version identifier, model/provider settings, and tool-environment configuration.
A run that cannot be reconstructed from stored artifacts MUST NOT be treated as
a valid benchmark result. Rationale: regressions are only meaningful if the
same experiment can be repeated.

### II. Comparable Runs
Cross-version comparison MUST use the same task suite, scoring rules, and
metric definitions. Changing a task, scorer, or metric schema MUST mint a new
suite or scorer version rather than silently mutating historical comparisons.
Rationale: mixed definitions produce false regressions and false confidence.

### III. Test-First Scoring (NON-NEGOTIABLE)
Scoring and aggregation logic MUST be specified by failing tests before
implementation. Tests MUST cover pass/fail decisions, tool-usage accounting,
latency, token usage, and cost rollups, including edge cases (zero tools,
failed runs, missing cost data). Implementation MUST NOT ship until those tests
exist and fail for the unimplemented behavior, then pass. Rationale: the
platform's product is the score; untested scorers cannot be trusted.

### IV. Adapter Isolation
Agent adapters MUST be isolated from scoring, storage, and comparison code.
Adapters MAY invoke agents and record traces; they MUST NOT compute official
metrics or mutate historical run records. Contract tests MUST cover adapter
input/output schemas whenever an adapter is added or its contract changes.
Rationale: mixing execution and judgment makes results non-portable across
agents.

### V. Measurable Observability
Every completed run MUST persist task success, tool-usage summary, latency,
token usage, and cost (or an explicit null with reason when a value cannot be
measured). Logs and stored traces MUST be structured enough to explain a
metric without re-running the agent. Rationale: unexplained numbers cannot
drive version decisions.

## Measurement Constraints

Required metrics for a complete run record:

- Task success (boolean or documented ordinal scale)
- Tool usage (counts and ordered call records sufficient to audit)
- Latency (end-to-end, with units)
- Token usage (input/output at minimum when the provider reports them)
- Cost (currency and units, or explicit unavailable)

Comparison and regression detection MUST operate on these stored metrics, not
on ad-hoc live recomputation that diverges from saved results. Secrets, API
keys, and raw provider credentials MUST NOT be written into run artifacts.

## Development Workflow

New features MUST start as a Spec Kit specification before implementation
unless the change is a constitution amendment. Scoring or metric-definition
changes MUST include tests and a statement of impact on historical
comparability. Pull requests MUST show how they comply with the five core
principles. Prefer the smallest design that preserves reproducibility and
comparability; added complexity MUST be justified against those two properties.

## Governance

This constitution supersedes informal practice and conflicting comments in
code or docs. Amendments MUST update this file, bump the version using
semantic versioning (MAJOR for removed or incompatible principles, MINOR for
new or expanded principles, PATCH for clarifications), set Last Amended to the
change date, and record rationale. Reviews and implementations MUST check
compliance with Core Principles before merge. Runtime guidance lives in
feature specs and plans; those documents MUST NOT weaken these rules.

**Version**: 1.0.0 | **Ratified**: 2026-09-14 | **Last Amended**: 2026-09-14
