# pipeline-runner

Manual CI/CD dispatcher that calls the reusable secure pipeline in
`allanvarghese567-lab/pipeline-templates` (pinned to tag `v1`).

## Required setup

### 1. Tag the templates repo

In `pipeline-templates`:

```bash
git tag v1
git push origin v1
```

### 2. Repo variable `ALLOWED_OWNERS`

Settings → Secrets and variables → Actions → Variables → New repository variable

- Name: `ALLOWED_OWNERS`
- Value: comma-separated list of allowed owners, e.g. `allanvarghese567-lab,my-org`

### 3. Repo secret `CROSS_REPO_TOKEN`

Settings → Secrets and variables → Actions → Secrets → New repository secret

- Name: `CROSS_REPO_TOKEN`
- Value: a **fine-grained PAT** (or GitHub App installation token) with:
  - Repository access: the target repos you will build (or all public)
  - Permissions: Contents (read), Metadata (read)

Never give this token write access.

### 4. Cross-repo workflow access (if templates is private)

In `pipeline-templates` → Settings → Actions → General → Access:

- Allow access from repositories in the `allanvarghese567-lab` account (or specific repos).

### 5. GitHub Environments + protection rules

In `pipeline-templates` (or the runner, depending on where you want the gates):

Settings → Environments → New environment

Create `dev`, `staging`, `prod`.

For **staging** and **prod**:

- Required reviewers: add yourself / team
- (Optional) Wait timer, deployment branches filter

The deploy job uses `environment: ${{ inputs.environment }}`, so these rules are enforced automatically.

## How to run

Actions → Pipeline Runner → Run workflow

| Input         | Example                          |
|---------------|----------------------------------|
| target_repo   | `allanvarghese567-lab/my-app`    |
| branch        | `main`                           |
| environment   | `none` / `dev` / `staging` / `prod` |
| build_type    | `npm` or `python`                |

## Security notes

- Concurrency group cancels in-progress runs for the same repo+branch
- Least-privilege job permissions
- Timeouts on all jobs
- No free-form shell commands; only fixed `npm` / `python` paths
- Inputs never appear as `${{ inputs.* }}` inside `run:` scripts
