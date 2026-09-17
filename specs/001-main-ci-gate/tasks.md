---
description: "Task list for Main Branch CI Safety Gate"
---

# Tasks: Main Branch CI Safety Gate

**Input**: Design documents from `/specs/001-main-ci-gate/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Spec does not request a separate TDD suite beyond the smoke placeholder that *is* the CI required check. `tests/test_ci_smoke.py` is implementation of FR-002, not an optional extra test phase.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- Repository root: `.github/workflows/`, `docs/`, `tests/`, `pyproject.toml`, `README.md`

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create directories and Python package metadata so CI has a project to install

- [x] T001 Create directories `.github/workflows/`, `docs/`, and `tests/` at repository root per `specs/001-main-ci-gate/plan.md`
- [x] T002 Create minimal `pyproject.toml` at repository root for Python 3.12 with package name `agent-eval-platform`, `[project.optional-dependencies] dev` including `pytest`, and `[tool.pytest.ini_options]` testpaths set to `tests`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Placeholder/smoke check that pytest can run and fail later when assertions break (FR-002, FR-011, SC-007)

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T003 Add passing smoke placeholder test in `tests/test_ci_smoke.py` (kind `placeholder_smoke` per `specs/001-main-ci-gate/data-model.md`) that asserts a trivial true condition and includes a short comment that scoring/adapter tests will live under `tests/` later
- [x] T004 Verify locally that `python -m pip install -e ".[dev]"` and `pytest` exit 0 from repository root (per `specs/001-main-ci-gate/quickstart.md`)

**Checkpoint**: Foundation ready — workflow and docs work can begin

---

## Phase 3: User Story 1 - Block Unsafe Merges to Main (Priority: P1) 🎯 MVP

**Goal**: CI job `ci` runs pytest and reports clear pass/fail; maintainer docs explain optional branch protection requiring check `ci`

**Independent Test**: Break `tests/test_ci_smoke.py` → job fails with visible log; restore it → job succeeds. Open `docs/ci-branch-protection.md` and confirm it names check `ci` and states enforcement is optional until enabled.

### Implementation for User Story 1

- [x] T005 [US1] Create `.github/workflows/ci.yml` with job id/name `ci` on `ubuntu-latest`, steps: checkout, setup Python 3.12, `pip install -e ".[dev]"`, `pytest`; fail the job on non-zero pytest exit; MUST NOT use `actions/upload-artifact` or coverage upload actions (per `specs/001-main-ci-gate/contracts/ci-workflow.md`)
- [x] T006 [P] [US1] Write `docs/ci-branch-protection.md` covering Settings → Branches for `main`, require status check `ci`, note that until enabled merges may still be allowed, and that at least one workflow run may be needed before `ci` appears in the check list (per `specs/001-main-ci-gate/contracts/branch-protection-docs.md`)
- [x] T007 [US1] Update `README.md` with a short “CI” section linking to `docs/ci-branch-protection.md` and stating that PRs/pushes to `main` run the `ci` check (pass/fail in Checks, no report artifacts)

**Checkpoint**: User Story 1 delivers a failable gate plus protection instructions (MVP)

---

## Phase 4: User Story 2 - Fast Feedback on Every Relevant Change (Priority: P2)

**Goal**: Safety pipeline starts automatically on pull requests targeting `main` and on pushes to `main` with no manual trigger

**Independent Test**: Open or update a PR to `main` and confirm workflow starts without `workflow_dispatch`; push to `main` (or simulate via PR merge path) and confirm a run is recorded for the commit.

### Implementation for User Story 2

- [x] T008 [US2] Ensure `.github/workflows/ci.yml` `on:` includes both `pull_request` with `branches: [main]` and `push` with `branches: [main]` exactly as specified in `specs/001-main-ci-gate/contracts/ci-workflow.md` (add any missing event; do not remove job `ci`)
- [x] T009 [US2] Add a brief “When CI runs” subsection to `README.md` documenting automatic triggers (PR → `main`, push → `main`) and that contributors do not need a manual start step (SC-001)

**Checkpoint**: Triggers and contributor-facing docs match FR-004 / User Story 2

---

## Phase 5: User Story 3 - Safe Checks Without Live Agent Spend (Priority: P3)

**Goal**: Gate stays deterministic and secret-free: fixtures/smoke only, no provider secrets, safe for public fork PRs

**Independent Test**: Inspect `.github/workflows/ci.yml` for absence of `${{ secrets.* }}` provider keys and live API steps; confirm job still only runs checkout/setup/install/pytest.

### Implementation for User Story 3

- [x] T010 [US3] Audit and harden `.github/workflows/ci.yml`: remove any provider secret references if present; add a top-of-file comment that this workflow MUST NOT call live agents or require LLM API keys (FR-006, FR-007, SC-005)
- [x] T011 [P] [US3] Extend `docs/ci-branch-protection.md` with a “Secrets & fork PRs” note: safety gate needs no provider secrets; fork PRs must not be given LLM credentials for this check
- [x] T012 [US3] Add a one-line note in `tests/test_ci_smoke.py` docstring that future scoring/adapter tests MUST use scrubbed fixtures only (no live provider calls) to keep CI aligned with FR-006 / FR-008

**Checkpoint**: All three stories satisfied without introducing eval.yml or artifact uploads

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Align repo docs with quickstart and confirm contract completeness

- [x] T013 [P] Cross-check `.github/workflows/ci.yml` against `specs/001-main-ci-gate/contracts/ci-workflow.md` (triggers, job name `ci`, no artifacts, Python 3.12, pytest)
- [x] T014 [P] Cross-check `docs/ci-branch-protection.md` against `specs/001-main-ci-gate/contracts/branch-protection-docs.md`
- [x] T015 Walk `specs/001-main-ci-gate/quickstart.md` locally (install + pytest) and fix any path/command drift in README or docs
- [x] T016 Confirm FR-009/FR-010: repository has no eval workflow and no coverage/artifact upload steps in `.github/workflows/ci.yml`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational — MVP
- **User Story 2 (Phase 4)**: Depends on Foundational; practically after T005 so `ci.yml` exists (extends triggers/docs)
- **User Story 3 (Phase 5)**: Depends on Foundational; practically after T005 so workflow exists to audit
- **Polish (Phase 6)**: After desired user stories complete

### User Story Dependencies

- **US1 (P1)**: After Phase 2 — creates workflow + protection docs
- **US2 (P2)**: After US1 workflow file exists — trigger completeness + README
- **US3 (P3)**: After US1 workflow file exists — secrets/fork safety (can parallelize with US2 on different files where marked [P])

### Parallel Opportunities

- T006 can proceed in parallel with T005 once Phase 2 is done (different files)
- After T005: T008 (ci.yml triggers) vs T011 (docs) can partially overlap carefully; prefer sequential edits to `ci.yml` (T005 → T008 → T010)
- T011 [P] and T012 [P] can run in parallel (different files)
- T013 [P] and T014 [P] can run in parallel during Polish

---

## Parallel Example: User Story 1

```bash
# After Phase 2, in parallel:
Task: "T005 [US1] Create .github/workflows/ci.yml ..."
Task: "T006 [P] [US1] Write docs/ci-branch-protection.md ..."
# Then:
Task: "T007 [US1] Update README.md ..."
```

---

## Parallel Example: User Story 3

```bash
# After T010 (ci.yml audit), in parallel:
Task: "T011 [P] [US3] Extend docs/ci-branch-protection.md ..."
Task: "T012 [P] [US3] Add docstring note in tests/test_ci_smoke.py ..."
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup  
2. Complete Phase 2: Foundational (smoke test)  
3. Complete Phase 3: US1 (workflow + branch-protection docs + README link)  
4. **STOP and VALIDATE**: Intentionally fail smoke test on a PR; confirm `ci` goes red; restore and confirm green  
5. Optionally enable branch protection later per docs  

### Incremental Delivery

1. Setup + Foundational → local pytest green  
2. US1 → remote `ci` check exists (MVP)  
3. US2 → automatic PR/push triggers documented and verified  
4. US3 → secret-free / no-live-call guarantees documented  
5. Polish → contract and quickstart alignment  

### Parallel Team Strategy

With multiple contributors: one owns `.github/workflows/ci.yml` (T005 → T008 → T010); another owns `docs/ci-branch-protection.md` + README (T006, T007, T009, T011) after T005 lands.

---

## Notes

- [P] = different files, no incomplete-task dependencies  
- Do not add `eval.yml`, object storage, or artifact uploads in this feature  
- Keep status check name stable as `ci` for branch protection  
- Commit after each task or logical group  
- Suggested MVP scope: Phases 1–3 (through US1)  
