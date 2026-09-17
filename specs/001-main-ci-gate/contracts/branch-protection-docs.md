# Contract: Branch Protection Setup (Maintainer)

**Path (to implement)**: `docs/ci-branch-protection.md` (linked from `README.md`)

This is an instructional contract for humans configuring GitHub repository settings. It is not executed by CI.

## Preconditions

- Workflow `.github/workflows/ci.yml` exists and has produced at least one run so the `ci` check appears in the status check list.

## Required instructions content

The doc MUST tell a maintainer to:

1. Open **Settings → Branches → Add branch protection rule** (or Rulesets equivalent) for `main`  
2. Enable **Require status checks to pass before merging**  
3. Select the check named **`ci`**  
4. Optionally enable “Require a pull request before merging” (recommended, not mandatory for this feature)  
5. Save the rule  

## Acceptance of the doc

- A maintainer who has never configured protection can follow the doc without reading the workflow YAML  
- The required check name in the doc matches the job status name from [ci-workflow.md](./ci-workflow.md) (`ci`)  
- The doc states that until the rule is enabled, failed checks are visible but merges may still be allowed (clarification B)

## Out of scope

- Automating settings via GitHub API or Terraform  
- Treating “protection already enabled in the remote repo” as a merge-blocking deliverable for the feature PR itself  
