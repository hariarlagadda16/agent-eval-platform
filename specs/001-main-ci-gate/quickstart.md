# Quickstart: Validate Main Branch CI Safety Gate

## Prerequisites

- GitHub repository with Actions enabled  
- Permission to push a branch and open a PR to `main` (or push to `main` in a solo workflow)  
- Local Python 3.12 optional (for running the smoke test offline)

## What you should have after implementation

- `.github/workflows/ci.yml` (see [contracts/ci-workflow.md](./contracts/ci-workflow.md))  
- `pyproject.toml` + `tests/test_ci_smoke.py`  
- `docs/ci-branch-protection.md` (see [contracts/branch-protection-docs.md](./contracts/branch-protection-docs.md))  
- README link to the branch-protection doc  

## Local smoke check

On macOS/Homebrew Python, use a virtualenv first (GitHub Actions does not need this):

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e ".[dev]"
pytest
```

**Expected**: pytest exits 0; smoke test passes.

## Remote validation scenarios

### 1. Automatic run on PR (SC-001)

1. Push a branch and open a PR targeting `main`.  
2. Open the PR **Checks** tab.

**Expected**: Workflow starts without a manual trigger; job `ci` appears with pass or fail.

### 2. Passing change (User Story 1)

1. Ensure smoke test passes locally.  
2. Open/update PR to `main`.

**Expected**: `ci` concludes **success**.

### 3. Failing change (SC-002)

1. Temporarily break the smoke test (e.g. `assert False`).  
2. Push to the PR.

**Expected**: `ci` concludes **failure**; log shows the failing test/step within the Actions UI (SC-003).

### 4. No secrets / no live calls (SC-005)

1. Confirm the workflow YAML does not reference provider secrets.  
2. Run CI on a PR from a fork or without repo secrets configured.

**Expected**: Job still runs pytest; no steps call external LLM APIs.

### 5. Optional branch protection (SC-004 / clarification B)

1. Follow `docs/ci-branch-protection.md`.  
2. Require check `ci` on `main`.  
3. Open a PR with a failing test.

**Expected**: Merge is blocked until `ci` is green.  
Before enabling the rule, a red `ci` is still visible but merge may remain allowed.

### 6. Placeholder → real tests (SC-007)

1. With only the smoke test, confirm CI passes.  
2. Add a deliberately failing unit test under `tests/`.  
3. Push.

**Expected**: Same job name `ci` fails; no rename of the status check required.

## Out of scope for this quickstart

- Running live agent evaluations  
- Uploading coverage artifacts  
- Verifying durable object-storage of eval metrics  
