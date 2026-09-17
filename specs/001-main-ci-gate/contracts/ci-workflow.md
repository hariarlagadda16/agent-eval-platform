# Contract: CI Safety Workflow

**Path (to implement)**: `.github/workflows/ci.yml`  
**Status check name**: `ci` (job `name` / job id must resolve to a check titled `ci` for branch protection)

## Triggers

| Event | Filter | Required |
|-------|--------|----------|
| `pull_request` | `branches: [main]` | Yes |
| `push` | `branches: [main]` | Yes |

No other events are required for this feature (`workflow_dispatch` optional, not required).

## Job contract

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Job name for status checks | `ci` |
| Secrets | None required; MUST NOT reference provider API secrets |
| Artifacts | MUST NOT call upload-artifact / coverage upload actions |

### Required steps (logical)

1. Checkout repository at the triggering SHA  
2. Set up Python 3.12  
3. Install project with test extras (e.g. `pip install -e ".[dev]"` or equivalent from `pyproject.toml`)  
4. Run `pytest` (fails the job on non-zero exit)

### Outcomes

| Condition | Job conclusion | Notes |
|-----------|----------------|-------|
| All steps succeed | `success` | Satisfies merge readiness when protection enabled |
| Any step fails | `failure` | Logs MUST show which step failed |
| Run cancelled | `cancelled` | MUST NOT be treated as success |

## Non-goals

- Live agent / provider calls  
- Durable eval metric storage  
- Coverage or report artifact uploads  

## Compatibility with future tests

When additional tests are added under `tests/` (unit, contract), the same `pytest` invocation MUST discover and run them without changing the status check name `ci`.
