# Research: Main Branch CI Safety Gate

## 1. CI host

**Decision**: GitHub Actions

**Rationale**: Repository is already on GitHub (`hariarlagadda16/agent-eval-platform`). Actions provides status checks that branch protection can consume, runs on PR/push events without extra vendors, and matches prior product discussion.

**Alternatives considered**:
- GitLab CI / CircleCI — would require migrating host or dual setup; rejected.
- Local-only pre-commit hooks — helpful later but do not gate remote merges alone; rejected as sole solution.

## 2. Language / test runner for the placeholder

**Decision**: Python 3.12 + pytest as the CI entrypoint, with one smoke test as the bootstrap placeholder

**Rationale**: The product domain (agent evaluation, scoring, adapters) fits Python. A pytest smoke test is a real required check that can fail later when assertions break, satisfying FR-002/FR-011 better than a no-op `echo`.

**Alternatives considered**:
- Bash-only `exit 0` step — simpler but not a path to scoring/contract tests; rejected as permanent design.
- Node/Jest — viable, but less aligned with typical eval/scoring stacks; deferred unless product stack later chooses Node.
- No package until app exists — leaves CI as empty green; weaker than a smoke test; rejected.

## 3. Workflow shape

**Decision**: Single workflow `.github/workflows/ci.yml`, single job named `ci`, triggers `pull_request` (branches: `main`) and `push` (branches: `main`)

**Rationale**: One status check name simplifies branch-protection docs (FR-005). Fail-fast on any step failure meets FR-003. No matrix until multiple runtimes are required.

**Alternatives considered**:
- Separate lint/test jobs — clearer later at cost of multiple required checks; deferred.
- `workflow_dispatch` only — violates automatic trigger requirements (FR-004); rejected.

## 4. Secrets and fork safety

**Decision**: Do not configure provider secrets for this workflow; do not reference `${{ secrets.* }}` for LLM keys

**Rationale**: Spec FR-006/FR-007 and constitution forbid secrets in artifacts and live calls on the safety gate. Public fork PRs must run without repo secrets.

**Alternatives considered**:
- Optional secrets with `if` guards — adds complexity and risk of accidental live calls; rejected for this feature.

## 5. Artifacts / coverage uploads

**Decision**: No `actions/upload-artifact` (or coverage upload actions) in this feature

**Rationale**: Clarification Option A / FR-010 — pass/fail status and logs only.

**Alternatives considered**:
- Always upload coverage — rejected by clarification.
- Conditional upload when suite grows — deferred to a later feature.

## 6. Branch protection

**Decision**: Document optional setup in `docs/ci-branch-protection.md` (and README link); do not automate org settings via API

**Rationale**: Clarification Option B — enforcement is a maintainer Settings step, not a hard repo-file deliverable. Docs should name the exact check (`ci`) to require.

**Alternatives considered**:
- GitHub API/Terraform to force protection — overkill for a resume/solo repo and may need admin tokens in CI; rejected.
- Treat protection enablement as merge-blocking DoD — rejected in clarify session (chose B over A).

## 7. Out of scope confirmation

**Decision**: No `eval.yml`, no object storage, no baseline regression against live models

**Rationale**: FR-009 and prior architecture conversation (Layer 2 later).

**Alternatives considered**:
- Combined CI+eval workflow — cost/flakiness/secret risk; rejected for main safety gate.
