# Implementation Plan: Main Branch CI Safety Gate

**Branch**: `001-main-ci-gate` | **Date**: 2026-09-17 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-main-ci-gate/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Deliver a GitHub Actions continuous integration safety gate that runs automatically on pull requests to `main` and on pushes to `main`, reports clear pass/fail status (no uploaded artifacts), uses a minimal placeholder check until a real suite exists, and documents optional branch-protection setup. Live agent evals and durable run storage remain out of scope.

## Technical Context

**Language/Version**: Python 3.12 (minimal project scaffold for future scoring/adapters; CI placeholder is a pytest smoke test)

**Primary Dependencies**: GitHub Actions (`actions/checkout`, `actions/setup-python`); pytest (dev dependency)

**Storage**: N/A (no durable eval artifacts; check status + workflow logs only)

**Testing**: pytest; CI job invokes `pytest` as the required check

**Target Platform**: GitHub-hosted Linux runners (`ubuntu-latest`)

**Project Type**: Repository tooling / CI gate (bootstrap for later library/CLI eval platform)

**Performance Goals**: Placeholder/smoke CI completes in under 3 minutes on GitHub-hosted runners

**Constraints**: No provider API secrets in the workflow; no `upload-artifact` for coverage/reports; no live agent calls; fork PRs must not require secrets

**Scale/Scope**: Single workflow file, one CI job, short maintainer doc for branch protection; no deploy or eval pipelines

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Gate | Status |
|-----------|------|--------|
| I. Reproducible Evaluation | This feature MUST NOT invent “official” eval results without recorded suite/agent inputs | PASS — CI gate produces check status only, not eval run records |
| II. Comparable Runs | MUST NOT mutate suite/scorer definitions via CI | PASS — no suite/scorer changes |
| III. Test-First Scoring | Pipeline MUST be ready to run scoring tests once they exist | PASS — pytest entrypoint; smoke placeholder today |
| IV. Adapter Isolation | When contract tests exist, CI MUST run them | PASS — same pytest discovery will pick up future `tests/contract/` |
| V. Measurable Observability | MUST NOT treat CI logs as durable official metrics; MUST NOT write secrets into artifacts | PASS — status/logs only; no artifact uploads; no provider secrets |
| Measurement Constraints | Secrets MUST NOT appear in run artifacts | PASS — no secrets configured for this workflow |
| Simplicity | Prefer smallest design that preserves reproducibility/comparability | PASS — one workflow, one job, docs only for protection |

**Post-Phase 1 re-check**: Still PASS — contracts describe workflow triggers/outcomes and branch-protection instructions only; no durable eval storage introduced.

## Project Structure

### Documentation (this feature)

```text
specs/001-main-ci-gate/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
.github/
└── workflows/
    └── ci.yml                 # Safety gate workflow (PR + push to main)

docs/
└── ci-branch-protection.md    # Maintainer instructions for optional required checks

pyproject.toml                 # Minimal Python package metadata + pytest config
tests/
└── test_ci_smoke.py           # Placeholder/smoke test (passes; replaced/expanded later)

README.md                      # Link to CI docs / how checks appear
```

**Structure Decision**: Single-repo CI tooling at the root. No `src/` application packages yet—only enough Python/pytest scaffolding for a real, failable gate that grows into scoring and adapter contract tests under `tests/` later.

## Complexity Tracking

> No constitution violations requiring justification.
