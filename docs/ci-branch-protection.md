# Branch protection for `main` (optional enforcement)

The safety gate workflow (`.github/workflows/ci.yml`) always runs on pull requests and pushes to `main` and reports a status check named **`ci`**.

Until you enable branch protection, a failing `ci` check is still **visible** on the PR, but GitHub **may still allow merge**. Enabling the rule below is a recommended one-time maintainer step; it is not automated by this repository.

## Preconditions

1. `.github/workflows/ci.yml` is on the default branch (or the PR that adds it has run at least once).
2. At least one workflow run has completed so the **`ci`** check appears in GitHub’s status-check list.

## Enable required checks on `main`

1. Open the repository on GitHub.
2. Go to **Settings → Branches**.
3. Click **Add branch protection rule** (or use **Rules → Rulesets** if your UI shows that).
4. Set **Branch name pattern** to `main`.
5. Enable **Require status checks to pass before merging**.
6. In the status checks search box, select **`ci`**.
7. Optionally enable **Require a pull request before merging** (recommended, not required for this feature).
8. Save the rule.

After this is enabled, merges into `main` are blocked while `ci` is failing or missing for the latest commit.

## Secrets & fork PRs

- This safety gate needs **no** provider/LLM API secrets.
- Do **not** grant fork pull requests access to LLM credentials for the `ci` check.
- Live agent evaluation belongs in a separate future workflow, not this gate.

## Troubleshooting

- If **`ci` does not appear** in the status-check picker, push a commit that triggers the workflow (open or update a PR to `main`), wait for the run to finish, then return to branch protection settings.
- A **cancelled** run does not count as success; re-run the workflow from the Actions tab if needed.
