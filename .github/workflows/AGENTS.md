# GitHub Actions Workflows (Agent Catalog)

Authoritative, agent-facing catalog of workflows in this repository. Use this when loading or modifying
workflows and keep it in sync with the files in this directory.

For a human-readable overview, see [README.md](README.md).

## Workflow catalog

- **[check-pr-comment.yml](check-pr-comment.yml)**: Extracts annotations from failed `Check` workflow runs and posts them
  as a PR comment.
- **[check.yml](check.yml)**: Linting and quality gates via actionlint and pre-commit.
- **[cogni-ai-agent.yml](cogni-ai-agent.yml)**: Logic for the Cogni AI Agent.
- **[copilot-setup-steps.yml](copilot-setup-steps.yml)**: Environment setup utility.
- **[devcontainer-ci.yml](devcontainer-ci.yml)**: Build/test devcontainer and required tools/packages.

## Details

### check-pr-comment.yml

- Purpose: provides PR feedback for failed `Check` workflow runs by extracting job annotations
  and posting them as a comment. Manages the `check-error` label.
- Triggers: `workflow_run` (after `Check` completions), `workflow_call`.
- Note: when invoked via `workflow_call`, callers should pass the needed `workflow_run`
  context explicitly via inputs (run_id, head_sha, conclusion, run_url) to ensure
  correct behavior.
- Permissions: `actions: read`, `checks: read`, `issues: write`, `pull-requests: write`.

### check.yml

- Purpose: run actionlint and pre-commit to enforce workflow and repo standards.
- Triggers: `push`, `pull_request`, `schedule`, `workflow_call`, `workflow_dispatch`,
  `workflow_run` (after `Cogni AI Agent` completions).
- Bot-PR support: `workflow_run` trigger enables checks on PRs created by bots,
  since normal `pull_request` events don't trigger for bot actors.
- Reusable: `uses: Cogni-AI-OU/.github/.github/workflows/check.yml@main`.
- Jobs: `actionlint`, `link-checker`, `pre-commit`.

### cogni-ai-agent.yml

- Purpose: provides the underlying logic to run the Cogni AI Agent.
- Inputs: `agent` (default `cogni-ai-architect`), `model`, `prompt`, `issue_number`,
  `version_agents`, `version_instructions`, `version_skills`.
- Triggers: `issue_comment`, `pull_request_review_comment`, `workflow_call`, `workflow_dispatch`.
- Details: Installs Python dependencies from `.devcontainer/requirements.txt` and calls the
  `Cogni-AI-OU/cogni-ai-agent-action` to process instructions. A post-run `summary` job generates
  an AI summary of the agent's actions.
- Concurrency: Only one run per issue/PR/branch at a time; new runs are queued (no auto-cancel).
- Permissions: `contents: write`, `id-token: write`, `issues: write`, `pull-requests: write`.

### copilot-setup-steps.yml

- Purpose: utility workflow for setting up the environment.
- Triggers: `push` and `pull_request` on `copilot-setup-steps.yml` or `.devcontainer/requirements.txt`.
- Details: Checks out repo, sets up Python 3.12, restores cache, and installs dependencies.
- Permissions: `contents: read`.

### devcontainer-ci.yml

- Purpose: build and validate the dev container; ensure required tools and Python packages exist.
- Inputs: `required_commands` (defaults to common CLI tools), `required_python_packages`
  (defaults to ansible, ansible-lint, docker, molecule, pre-commit, uv).
- Triggers: pull_request/push affecting `.devcontainer/` or this workflow; weekly schedule;
  `workflow_call`.
- Permissions: callers must grant `packages: write` when pushing images to GHCR.
- Reusable: `uses: Cogni-AI-OU/.github/.github/workflows/devcontainer-ci.yml@main`.

## Synchronized Configuration

The following configuration values should be reviewed for consistency:

### Model options list

The `model` input options for `workflow_dispatch` in `cogni-ai-agent.yml` define the supported AI models.

### Agent options list

The `agent` input options for `workflow_dispatch` in `cogni-ai-agent.yml` define the available specialized agents.

## Notes

- Follow the GitHub workflows instructions (available in the runtime instructions catalog)
  when editing workflow files (ordering, formatting, validation).
- Keep this catalog updated when workflows are added, removed, or renamed.
