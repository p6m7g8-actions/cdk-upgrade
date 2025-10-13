# Copilot Coding Agent Onboarding

This repository defines a **composite GitHub Action** used to **build, test, and deploy AWS CDK Construct projects**.  
It is **not** itself a CDK Construct or deployable app — it provides reusable CI automation for other repositories that do.

---

## 1. High-level Overview

- **Type:** GitHub Action (composite)
- **Primary entry point:** `action.yml`
- **Purpose:** Automate setup and deployment for repositories containing AWS CDK Constructs.
- **Languages:** YAML, Bash
- **Runtime:** Node.js 20.x on Ubuntu 24.04
- **Tools used:** pnpm, AWS CDK, OpenTofu, AWS OIDC for authentication.

The Action installs the CDK CLI, configures AWS credentials, installs OpenTofu, and executes the consumer project’s `pnpm run deploy` command.  
Consumers must define their own deploy script; this repository only provides the reusable automation.

---

## 2. Build and Validation Instructions

### Environment Setup
All workflows assume a clean Ubuntu 24.04 environment with Node 20.x.

```bash
corepack enable
pnpm install
```
Always run `pnpm install` before building to ensure all dependencies and toolchain versions are installed.

### Composite Action Behavior
`action.yml` runs these steps in order:

1. **Checkout** code (`actions/checkout@v5`)
2. **Setup Node.js** (`p6m7g8-actions/node-setup@main`)
3. **Install CDK CLI**
   ```bash
   npm install --global aws-cdk
   ```
4. **Configure AWS credentials** using OIDC via `aws-actions/configure-aws-credentials@v5.1.0`
5. **Install OpenTofu** (`opentofu/setup-opentofu@v1.0.6`)
6. **Run Deploy**
   ```bash
   pnpm run deploy
   ```
   with environment variables:
   - `CDK_DEPLOY_ACCOUNT`
   - `CDK_DEPLOY_REGION`
   - `AWS_REGION`
   - `TERRAFORM_BINARY_NAME=tofu`

**Preconditions:**  
Consumer repository must define `pnpm run deploy` and provide a valid OIDC role.  

**Postconditions:**  
Successful CDK Construct build and deploy within the consumer repo.

---

## 3. Project Layout and Automation

```
action.yml                   # Composite action definition
.github/workflows/
  ├── build.yml              # Required “Build / build” workflow
  ├── auto-queue.yml         # Enqueues PRs to Merge Queue
  ├── auto-approve.yml       # Auto-approves trusted PRs
  ├── pr-labeler.yml         # Adds contribution and auto-merge labels
  └── pull-request-lint.yml  # Enforces Conventional Commit PR titles
.vscode/settings.json        # Editor/YAML defaults
README.md                    # Repository intro
LICENSE
```

### GitHub Workflows

- **Build (`build.yml`)**
  - Triggers: `pull_request`, `merge_group`, `workflow_dispatch`
  - Required by Merge Queue and must always pass.

- **Auto-queue (`auto-queue.yml`)**
  - Runs after successful Build workflow to enqueue the PR into the Merge Queue.

- **Auto-approve (`auto-approve.yml`)**
  - Auto-approves PRs when authored by trusted users or labeled `auto-approve`.

- **PR Labeler & Linter**
  - Manage PR labels and enforce semantic PR titles.

---

## 4. Validation and Testing

To simulate CI locally:
```bash
act pull_request
```

To verify Merge Queue behavior:
1. Enable “Require merge queue” on the main branch.
2. Require check: `Build / build (pull_request)`.
3. Run:
   ```bash
   gh pr merge <number> -R <owner>/<repo>
   ```
   It should enqueue, not merge, the PR.

---

## 5. Agent Guidance

- This repo is **a GitHub Action**, not a CDK project.
- Never rename or remove the `Build` workflow or job id `build`.
- Use Node.js 20.x, Ubuntu 24.04, and pnpm consistently.
- Test all YAML syntax with `act` or via a sandbox PR before merging.
- Only perform searches or edits outside this document if data here is missing or outdated.
