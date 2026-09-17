# agent-eval-platform

AI agent evaluation and regression-testing platform that benchmarks agents across task success, tool usage, latency, token usage, and cost, while comparing versions and detecting performance regressions.

## CI

Pull requests and pushes to `main` automatically run the **`ci`** GitHub Actions check (Python 3.12 + pytest). No manual start step is required.

- **Pass/fail** shows under the PR **Checks** tab and in the Actions log. This feature does **not** upload coverage or other report artifacts.
- The workflow does **not** call live agents and does **not** need provider API secrets.

### When CI runs

| Event | Branch | Starts automatically? |
|-------|--------|------------------------|
| Pull request opened/updated | targeting `main` | Yes |
| Push | to `main` | Yes |

### Optional: block merges until `ci` is green

See [docs/ci-branch-protection.md](docs/ci-branch-protection.md) for how to require the **`ci`** status check on `main` in GitHub Settings. Until that rule is enabled, failed checks are still visible but merges may still be allowed.
