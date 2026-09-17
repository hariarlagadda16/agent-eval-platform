# Data Model: Main Branch CI Safety Gate

This feature does not introduce application persistence. Entities are conceptual records produced by the CI host and documented policy.

## Entities

### Safety Pipeline Run

Represents one execution of the safety workflow for a git commit (and optionally a pull request).

| Field | Description | Rules |
|-------|-------------|-------|
| `commit_sha` | Git commit being checked | Required; immutable for the run |
| `trigger` | `pull_request` or `push` | Required |
| `target_branch` | Branch receiving the change | Must be `main` for this feature’s triggers |
| `workflow_name` | Workflow identifier | `ci` (file: `.github/workflows/ci.yml`) |
| `job_name` | Status check name | `ci` (stable name for branch protection) |
| `overall_status` | `success` \| `failure` \| `cancelled` | `cancelled` MUST NOT count as success for merge readiness |
| `started_at` / `finished_at` | Run timestamps | Provided by CI host |
| `log_url` | Link to workflow logs | Used for FR-003 / SC-003 diagnosis |

**Lifecycle**: `queued` → `in_progress` → (`success` \| `failure` \| `cancelled`). Re-run creates a new run for the same or new SHA.

### Required Check

A verification step that must succeed for overall success.

| Field | Description | Rules |
|-------|-------------|-------|
| `name` | Human-readable step | e.g. `Install dependencies`, `Run tests` |
| `kind` | `placeholder_smoke` \| `unit` \| `contract` \| `lint` \| `types` | Bootstrap uses `placeholder_smoke`; later kinds added when tooling exists |
| `status` | `success` \| `failure` \| `skipped` | Any non-success required check fails the job |
| `evidence` | Log lines / pytest output | No uploaded artifact files in this feature |

**Relationships**: Many Required Checks belong to one Safety Pipeline Run. Overall run succeeds iff every required check succeeds.

### Fixture / Recorded Trace (future-facing)

Scrubbed inputs for scoring/adapter tests. Not required for the smoke placeholder.

| Field | Description | Rules |
|-------|-------------|-------|
| `id` | Fixture identifier | Stable for tests |
| `payload` | Scrubbed content | MUST NOT contain API keys or raw credentials |
| `purpose` | `scoring` \| `adapter_contract` \| other | Used when those tests exist |

### Merge Gate Policy

Documented maintainer configuration (outside repo runtime).

| Field | Description | Rules |
|-------|-------------|-------|
| `protected_branch` | `main` | Fixed for this feature |
| `required_status_checks` | List of check names | MUST include `ci` when enabled |
| `enabled` | Boolean | Optional at first (clarification B); when true, merges blocked until checks pass |

**Relationships**: Policy references Required Check / job name `ci`. Enabling policy does not change pipeline behavior; it only enforces outcomes at merge time.
