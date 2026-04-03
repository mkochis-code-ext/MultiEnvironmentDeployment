# MultiEnvironmentDeployment

An example of a multi-environment deployment process using GitHub Actions. This repo demonstrates CI/CD pipelines with sequential environment promotion (Dev → Test → Prod), approval gates, and branch protection best practices.

## Pipelines

### CI (`CI.yml`)

Runs on every pull request targeting `main`. Builds the code and validates changes before merge. This pipeline is required to pass as part of the branch policy before a PR can be merged.

### CD - Dev (`CD-Dev.yml`)

Manually triggered workflow that deploys to the **Dev** environment only.

### CD - Test (`CD-Test.yml`)

Manually triggered workflow that deploys to the **Test** environment only.

### CD - Prod (`CD-Prod.yml`)

Manually triggered workflow that deploys to the **Prod** environment only.

### CD - Multi-Environment (`CD-MultiEnv.yml`)

The main deployment pipeline. Triggers automatically on push to `main` (after a PR merge) or via manual dispatch. Deploys sequentially through all three environments: **Dev → Test → Prod**. Each environment goes through a Build, Approve, and Deploy stage.

### Deploy Template (`deploy-template.yml`)

A reusable workflow called by all CD pipelines. For a given environment it runs three jobs:

1. **Build** – Builds the code for the target environment.
2. **Approve** – Waits for manual approval via the `<env>-approve` environment protection rule.
3. **Deploy** – Deploys to the target environment after approval.

## Required GitHub Environments

Six GitHub environments must be configured in **Settings → Environments**:

| Environment | Purpose | Protection Rules |
|---|---|---|
| `dev` | Dev secrets/config | None |
| `test` | Test secrets/config | None |
| `prod` | Prod secrets/config | None |
| `dev-approve` | Approval gate for Dev deployment | Required reviewers |
| `test-approve` | Approval gate for Test deployment | Required reviewers |
| `prod-approve` | Approval gate for Prod deployment | Required reviewers |

The `-approve` environments should have **required reviewers** configured so that deployments pause and wait for a team member to approve before proceeding.

## Branch Protection Rules

The following branch protection rules should be configured on the `main` branch:

- **Require a pull request before merging** – Direct pushes to `main` are not allowed. All changes must go through a pull request.
- **Require approvals** – At least one reviewer must approve the pull request before it can be merged.
- **Require status checks to pass before merging** – The **CI** workflow must pass before the pull request can be merged. Add `CI` as a required status check.

These policies ensure that all changes are reviewed and validated before reaching `main`, which then triggers the full multi-environment deployment pipeline.
