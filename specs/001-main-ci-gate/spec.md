# Feature Specification: Main Branch CI Safety Gate

**Feature Branch**: `001-main-ci-gate`

**Created**: 2026-09-17

**Status**: Draft

**Input**: User description: "help create a cicd pipleine to ensure any new code being pushed to main is safe and doesnt cause any breaking changes"

## Clarifications

### Session 2026-09-17

- Q: Should landing on `main` be hard-blocked until the safety checks pass, or is it enough that checks run and maintainers turn on blocking later? → A: Pipeline + docs (Option B): checks always run; maintainer enables branch protection using provided instructions; enforcement optional at first
- Q: When the repo still has no real application tests, what should the safety pipeline do so `main` stays usable but ready to catch breaks later? → A: Minimal placeholder check now; becomes real test/lint gate as soon as those exist (Option B)
- Q: Should this safety gate upload short-lived reports (like test coverage summaries) on each run, or only show pass/fail in the check status? → A: Pass/fail check status only; no uploaded reports in this feature (Option A)

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Block Unsafe Merges to Main (Priority: P1)

As a maintainer, when someone proposes changes that would land on `main`, I need automated checks to run and fail clearly if the change breaks agreed quality rules (lint/style if present, type checks if present, and unit/contract tests), so broken or regressing code cannot be treated as ready for `main`.

**Why this priority**: Protecting `main` from breaking changes is the core reason for this feature; without a failing gate, “CI/CD for safety” delivers no value.

**Independent Test**: Open a change that intentionally fails a required check; confirm the pipeline reports failure and the change is not considered merge-ready. Open a change that passes all required checks; confirm the pipeline reports success.

**Acceptance Scenarios**:

1. **Given** a proposed change that fails one or more required automated checks, **When** the continuous integration pipeline runs, **Then** the overall result is failure and the failure reason is visible in the pipeline output.
2. **Given** a proposed change that passes all required automated checks, **When** the continuous integration pipeline runs, **Then** the overall result is success.
3. **Given** a maintainer has followed the feature’s branch-protection setup instructions and enabled required checks for `main`, **When** those required checks have not succeeded, **Then** merging into `main` is blocked.
4. **Given** branch protection has not been enabled yet, **When** the safety pipeline fails on a pull request, **Then** the failure is still visible as a check result, but the repository MAY still allow merge until protection is turned on.

---

### User Story 2 - Fast Feedback on Every Relevant Change (Priority: P2)

As a contributor, when I push or update a pull request that targets `main` (or push directly to `main` if allowed), I need the same safety checks to run automatically without manual setup, so I learn quickly whether my change is safe.

**Why this priority**: A gate that only runs occasionally still allows broken code onto `main`; automatic runs on every relevant change close that gap.

**Independent Test**: Create or update a change targeting `main` and confirm the pipeline starts without a manual trigger and finishes with a pass/fail status.

**Acceptance Scenarios**:

1. **Given** a new or updated pull request targeting `main`, **When** the change is published, **Then** the safety pipeline starts automatically.
2. **Given** a push to `main`, **When** the push completes, **Then** the safety pipeline starts automatically and records pass/fail for that commit.

---

### User Story 3 - Safe Checks Without Live Agent Spend (Priority: P3)

As a project owner, I need the `main` safety gate to validate scoring and adapter contracts using recorded fixtures—not live paid agent/provider calls—so CI stays cheap, deterministic, and free of secret leakage risk on public forks.

**Why this priority**: Live evals are valuable later but conflict with “safe and non-breaking” PR/main gates when they add cost, flakiness, and secret exposure; fixtures preserve constitution principles of test-first scoring and adapter isolation without live calls.

**Independent Test**: Run the safety pipeline with no provider credentials configured; confirm required checks still execute using fixtures and do not attempt live agent invocations.

**Acceptance Scenarios**:

1. **Given** no provider API credentials are available to the safety pipeline, **When** the pipeline runs, **Then** required unit/contract checks still complete using fixtures or recorded scrubbed traces.
2. **Given** a public fork pull request, **When** the safety pipeline runs, **Then** it does not expose or require project provider secrets.

---

### Edge Cases

- What happens when the repository has no application tests yet? The pipeline MUST run a minimal placeholder check that succeeds today, and MUST automatically include real automated tests (and lint/static checks when configured) as soon as those exist so failing tests fail the gate.
- What happens when only lint or only tests are configured? The pipeline MUST run every configured required check and fail if any required check fails.
- How does the system handle a flaky or cancelled run? Contributors MUST be able to re-run the pipeline; a cancelled run MUST NOT count as success for merge readiness.
- What happens if someone force-pushes or amends commits on a PR? Checks MUST re-run against the latest commit.
- How are secrets handled? The safety gate MUST NOT write API keys, credentials, or raw secrets into logs or published short-lived reports.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST run an automated continuous integration safety pipeline for changes intended for `main`.
- **FR-002**: System MUST execute all configured required quality checks for a change. Until a real test suite (and optional lint/static tooling) exists, System MUST run a minimal placeholder check that always executes and currently passes. Once a test suite exists, System MUST run those automated tests as required checks; once lint/static checks are configured, System MUST run them as required checks as well.
- **FR-011**: System MUST treat the placeholder check as temporary bootstrap only; it MUST NOT permanently replace real tests once scoring, adapter, or other application tests are present.
- **FR-003**: System MUST mark the pipeline as failed if any required check fails, and MUST surface enough output for a contributor to identify which check failed.
- **FR-004**: System MUST trigger the safety pipeline automatically on pull requests targeting `main` and on pushes to `main`.
- **FR-005**: System MUST publish check results suitable for use as required status checks on `main`, and MUST include maintainer instructions for enabling branch protection (or equivalent merge policy). Enabling that protection is a recommended one-time settings step outside the repo files and is NOT a hard deliverable for feature completion; until it is enabled, the pipeline still runs and reports pass/fail without blocking merges by itself.
- **FR-006**: Required checks in this safety gate MUST NOT depend on live agent or paid provider calls; validation MUST use fixtures or scrubbed recorded traces.
- **FR-007**: System MUST NOT persist provider credentials or other secrets into pipeline artifacts or repository history as part of this feature.
- **FR-008**: When scoring or adapter contract tests exist, the safety gate MUST run them so breaking changes to scoring rules or adapter contracts fail the pipeline (aligned with test-first scoring and adapter isolation).
- **FR-009**: System MUST keep this feature scoped to the cheap/deterministic safety gate; scheduled or on-demand live evaluation/regression suites are out of scope and MUST be specified as a separate later feature.
- **FR-010**: This feature MUST rely on pass/fail check status (and the pipeline log) for contributor feedback. It MUST NOT upload short-lived coverage or summary report artifacts as part of the safety gate. Such reports remain out of scope until a later change explicitly adds them; they still MUST NOT be treated as durable official evaluation records if added later.

### Key Entities

- **Safety Pipeline Run**: A single execution of the main-branch safety checks for one commit or pull request, with overall pass/fail and per-check outcomes.
- **Required Check**: An individual quality verification (tests, lint, types, contract tests) that must pass for the pipeline to succeed.
- **Fixture / Recorded Trace**: Scrubbed sample inputs used by automated tests so scoring and adapter behavior can be verified without live agents.
- **Merge Gate Policy**: The rule that prevents incomplete or failing checks from landing on `main`.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of pull requests targeting `main` automatically receive a safety pipeline run without a manual start step.
- **SC-002**: 100% of intentionally failing required-check examples cause a failed pipeline result in verification.
- **SC-003**: Contributors can determine which required check failed within 2 minutes of opening the pipeline result view.
- **SC-004**: After a maintainer enables the documented merge policy, 0 merges to `main` complete while the latest required safety pipeline for that change is failing or missing. Before that setting is enabled, 100% of relevant changes still receive a visible pass/fail pipeline result.
- **SC-005**: Safety pipeline runs complete without live provider credentials and without attempting live agent calls in 100% of gate runs.
- **SC-006**: Once scoring/adapter contract tests exist, a deliberate breaking change to those contracts fails the safety pipeline before merge.
- **SC-007**: With no application test suite present, 100% of safety pipeline runs still execute a minimal placeholder check and report an overall result; after a real failing test is added, 100% of subsequent runs that hit that failure report pipeline failure.

## Assumptions

- “Safe / no breaking changes” for this feature means automated quality and contract checks (including scoring/adapter tests when present), not full live agent benchmark campaigns.
- Live evaluation, durable object-storage of official run metrics, and baseline regression gates against paid model calls are a separate follow-up feature.
- The project will host the pipeline on its existing source-hosting continuous integration facility; exact vendor product names are left to planning.
- Application language/tooling may still be chosen; until then the gate ships with a minimal placeholder check and is structured so real install/test/lint commands plug in when a package manifest and tests exist.
- Branch protection (or equivalent) is a one-time repository settings change by a maintainer; this feature ships the pipeline plus setup instructions, and treats enforcement as optional until that setting is turned on.
- Small scrubbed fixtures committed to the repo for unit/contract tests are acceptable; large eval dumps and secrets are not.
- Direct pushes to `main` may still occur in a solo/resume workflow; the pipeline still runs on those pushes and records pass/fail even if policy is not yet enforced.
