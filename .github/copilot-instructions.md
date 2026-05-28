# Copilot Instructions

## Project Overview

This is the user's special `.github` repository. It provides
default community health files, GitHub Actions workflows, issue/PR templates,
and coding standards that apply across all repositories in the organization.

## Formatting Guidelines

### JSON

Follow the JSON rules in the runtime instructions catalog (dynamically populated at runtime,
e.g., in `~/.instructions/`) or refer to the repository `.editorconfig` configuration.

To test locally, use `jq` for validation or use the VS Code JSON formatter.

### Markdown

Follow the Markdown rules in the runtime instructions catalog, which mirror the repository markdownlint configuration.

To test locally, run via `pre-commit run markdownlint -a` or use the VS Code Markdownlint extension.

### YAML

Follow the YAML rules in the runtime instructions catalog, which mirror the repository `.yamllint` configuration.

Notes:

- Project utilizes Codespaces with config at `.devcontainer/devcontainer.json` and requirements at `.devcontainer/requirements.txt`.
- GitHub Actions run pre-commit checks (`.pre-commit-config.yaml`).
- To verify locally, run `pre-commit run yamllint -a` from the repo root.

### Devcontainer Guidance

Project utilizes Codespaces with config at `.devcontainer/devcontainer.json`
and requirements at `.devcontainer/requirements.txt`.
These rules apply only when the agent is running inside GitHub Codespaces
or the repository's VS Code devcontainer.
In that environment, the container should be treated as the controller
runtime and as the source of required controller-side dependencies.

- Treat the repository devcontainer as the default controller environment
  for local development and testing.
- Keep controller dependency installation in the devcontainer
  configuration so Molecule scenarios can assume those tools are already
  available.
- Do not install controller-side Python dependencies during Molecule runs
  when the agent is already operating inside Codespaces or the repo
  devcontainer.
- Do not create additional Python virtual environments such as `.venv`
  or `venv`; use the existing container Python environment, which should
  already provide the required dependencies.
- If dependencies are missing in a Codespace or devcontainer, update
  `.devcontainer/requirements.txt` or `.devcontainer/devcontainer.json`
  instead of introducing a per-run Molecule install step or a separate
  virtual environment.
- Use `CODESPACES=true` as the primary quick check for GitHub Codespaces.
- Outside Codespaces, use the presence of `/.dockerenv` together with
  the repository `.devcontainer/devcontainer.json` as a practical check
  for the containerized dev environment.

## Project Structure

```text
.
├── .github/
│   ├── ISSUE_TEMPLATE/      # Issue templates (bug reports, feature requests)
│   ├── workflows/           # GitHub Actions workflows
│   ├── copilot-instructions.md
│   └── pull_request_template.md
├── .tours/                   # VS Code guided tours
├── profile/
│   └── README.md             # Organization profile (shown on GitHub org page)
├── AGENTS.md                 # AI agent guidance
├── CODE_OF_CONDUCT.md        # Community standards
└── README.md                 # Repository documentation
```

### Tours

- Keep the `.tours` folder up-to-date (especially `.tours/getting-started.tour`)
  when making significant changes to the codebase.
  Update existing tours or create new ones to reflect changes in project structure,
  workflows, or key files.

## Git Operations

When working with the user interactively (e.g., in an IDE like VS Code):

- **Never create commits or push changes to branches** without explicit user feedback or requests.
- Present the proposed changes, successfully save the edited files, and allow the user to review the diffs locally.
- Let the user drive the Git staging, committing, and pushing processes,
  or wait for them to explicitly instruct you to perform these operations.

## Troubleshooting

### Finding Build Errors

To identify and diagnose the latest build errors:

1. **Reproduce errors locally:**
   - For pre-commit errors: Run `pre-commit run -a` to check all files
   - For specific hooks: Run `pre-commit run <hook-name> -a` (e.g., `markdownlint`, `yamllint`)
   - For actionlint errors: Install actionlint and run it on workflow files

2. **Common error patterns:**
   - **Ansible missing Python modules:** If a module such as `requests` or
     `docker` is installed for the main container Python but Ansible still
     cannot import it, check `ansible --version` to identify the interpreter in
     use. In Codespaces/devcontainers, Ansible may run from a pipx-managed
     environment, so install controller-side libraries there as well, for
     example with `pipx inject ansible -r .devcontainer/requirements-ansible.txt`.
   - **Markdown linting errors:** Check `.markdownlint.yaml` for rules; errors show line numbers
   - **YAML linting errors:** Check `.yamllint` for rules; verify indentation and structure
   - **JSON formatting errors:** Use `jq . <file>` to validate JSON syntax
