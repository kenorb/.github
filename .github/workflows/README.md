# GitHub Workflows and Actions

This directory contains GitHub Actions workflows, agent prompts, and related configuration.

## Workflows

### Check Workflow (`check.yml`)

The `check.yml` workflow runs on pull requests, pushes, and weekly schedule to
ensure code quality and correctness.

Jobs:

- **actionlint**: Validates GitHub Actions workflow files
- **link-checker**: Checks for broken links in Markdown files using Lychee
- **pre-commit**: Runs pre-commit hooks for code formatting and linting

#### Link Checker

The link checker job uses [Lychee](https://github.com/lycheeverse/lychee) to
scan all Markdown files for broken links. It includes caching to avoid rate
limits and can be configured via `.lycheeignore` at the repository root to
exclude specific URLs or patterns.

**Local Testing**: You can test links locally with the configured
`markdown-link-check` pre-commit hook:

```bash
# Install from requirements.txt
pip install -r .devcontainer/requirements.txt

# Check a single file
pre-commit run markdown-link-check --files path/to/file.md

# Check all Markdown files
pre-commit run markdown-link-check -a
```

The hook uses `.markdown-link-check.json` and checks both local file references
and remote URLs before you push changes.

#### Using Check as a Reusable Workflow

You can use the Check workflow in your repository by referencing it via `workflow_call`:

```yaml
---
name: Check
on:
  pull_request:
  push:
  schedule:
    - cron: 0 0 * * 1  # Run every Monday at 00:00 UTC
  workflow_dispatch:
jobs:
  check:
    uses: Cogni-AI-OU/.github/.github/workflows/check.yml@main
    with:
      submodules: 'false'  # Set to 'true' or 'recursive' if repository uses submodules
```

### Cogni AI Agent Workflow (`cogni-ai-agent.yml`)

The `cogni-ai-agent.yml` workflow provides the underlying logic to run the Cogni AI Agent. It runs on issue
comments, pull request review comments, and manual triggers (`workflow_dispatch`). It installs Python dependencies
from `.devcontainer/requirements.txt` and calls the `Cogni-AI-OU/cogni-ai-agent-action` to process natural language
instructions and automate repository tasks using selected LLM models.

### Copilot Setup Steps Workflow (`copilot-setup-steps.yml`)

The `copilot-setup-steps.yml` workflow is a utility workflow that checks out the repository, sets up Python 3.12,
restores the Python user site cache, and installs dependencies from `.devcontainer/requirements.txt`. It is triggered
on pushes and pull requests that modify the workflow file or the requirements file.

### Devcontainer CI Workflow (`devcontainer-ci.yml`)

**Purpose**: It ensures that all required command-line tools (e.g., `docker`, `gh`, `pre-commit`) and Python packages
(e.g., `ansible-lint`, `molecule`) are properly installed and functional inside the devcontainer. It runs on changes
to the `.devcontainer` directory, on a weekly schedule, and can also be used as a reusable workflow (requires
`packages: write` permission for the caller).

#### Using Devcontainer CI as a Reusable Workflow

```yaml
jobs:
  devcontainer:
    uses: kenorb/.github/.github/workflows/devcontainer-ci.yml@main
    permissions:
      contents: read
      packages: write  # Required for pushing to GitHub Container Registry
```

## Workflow Templates

The `../workflow-templates/` directory contains reference workflows that are not
actively executed but are preserved for future use or copying to other
repositories. These templates can be customized and moved to the `workflows/`
directory when needed.

## Agent Prompts

The `../prompts/` directory contains ready-to-use prompts for AI agents to perform
common repository management tasks. For agent-loading guidance and catalog, see
[../prompts/AGENTS.md](../prompts/AGENTS.md). For human-oriented details, see
[../prompts/README.md](../prompts/README.md).

## MCP Configuration

The `../mcp-config.json` configuration provides GitHub Copilot access to built-in tools:

- **Repository & Code:** `get_file_contents`, `search_code`, `search_repositories`, `list_branches`, `list_commits`
- **Issues & PRs:** `get_issue`, `list_pull_requests`, `create_pull_request`
- **Actions:** `list_workflows`, `list_workflow_runs`, `get_job_logs`

## Problem Matchers

GitHub Actions problem matchers automatically annotate files with errors and
warnings in pull requests, making it easier to identify and fix issues.

### Available Matchers

- **actionlint-matcher.json**: Captures errors from actionlint workflow linting
- **pre-commit-matcher.json**: Captures errors from pre-commit hooks

### Pre-commit Problem Matcher

The pre-commit problem matcher supports two output formats:

1. **Generic format** (`file:line:col: message`): Used by flake8, actionlint,
   and other tools that provide column information
2. **No-column format** (`file:line message`): Used by markdownlint and other
   tools that only provide line numbers

Note: Some hooks like yamllint and ansible-lint already output GitHub Actions
annotations directly and don't need the problem matcher.

### Configuration

Problem matchers are registered in the `check.yml` workflow
before running the corresponding tools.

### Using Matchers in Reusable Workflows

When using the `check.yml` workflow as a reusable workflow (via `workflow_call`),
the matcher files are automatically provided from this repository. You don't need
to copy the matcher files to your repository.

If you want to use custom matcher files, you can specify them using the inputs:

```yaml
jobs:
  check:
    uses: kenorb/.github/.github/workflows/check.yml@main
    with:
      actionlint-matcher-path: .github/custom-actionlint-matcher.json
      pre-commit-matcher-path: .github/custom-pre-commit-matcher.json
```

If these inputs are not provided, the workflow will automatically use the default
matcher files from this repository.
