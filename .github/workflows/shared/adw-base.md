---
# Shared configuration for every ADW agentic workflow. It is the same for every project.
# Imported by the adw-*.md workflows in the parent folder; it has no trigger of its own.
# Project-specific setup (runtimes, network access, dependency installation) lives in
# adw-project.md, next to this file.
engine: claude

tools:
  edit:
  bash: [":*"]
  github:
    toolsets: [default]
  timeout: 600
---

## ADW operating rules

You are running an AI Developer Workflow (ADW) inside GitHub Agentic Workflows. The same
workflows can be run by hand on a developer machine through the scripts in `adws/`. Both
modes share the prompt templates in `.claude/commands/`, so the work you produce here should
look the same as a manual run.

### The project

Read `.claude/adw_project.md` before you start. It lists the project's relevant files, how
to add a dependency, and the validation commands. The prompt templates refer to it.

### Using the prompt templates

When a step says to follow a template, read that file and carry out its instructions.

- The templates were written to be invoked as slash commands with positional arguments.
  Substitute the values given in the step for `$1`, `$2`, `$3` or `$ARGUMENTS`.
- Each template ends with a `Report` section asking for a specific output, in some cases
  "ONLY" a JSON document or a file path. In manual mode a script parses that output. Here,
  keep it as an intermediate result for the steps that follow, then carry on with the next
  step.

### Identifiers

- `adw_id` is `${{ github.run_id }}`. Use it wherever a template asks for an ADW ID.
- `issue_class` is one of `feat`, `bug` or `chore`.

### Git and GitHub

The sandbox has read-only access to GitHub and cannot push. Your changes reach the repository
only through the safe-output tools, which are applied after you finish. For that reason:

- Commit locally. Do not run `git push`, `git pull` or any `gh` command that writes.
- Do not follow `.claude/commands/commit.md`, `.claude/commands/pull_request.md` or
  `.claude/commands/generate_branch_name.md`. They assume push access. Use the conventions
  below instead, which match what those templates produce.
- Commit messages use the form `<agent_name>: <issue_class>: <message>`, where the message
  is present tense, 50 characters or fewer, and has no trailing period. Use `sdlc_planner`
  for the plan commit, `sdlc_implementor` for implementation commits and `test_runner` for
  fixes made while testing.
- Do not add any AI attribution or co-author trailer to commits or pull requests.

### Files that are held for review

A pull request that changes any of the following is still created, but it is held for
manual review. Change them only when the issue or the plan requires it, and list them in
your summary.

- Anything under `.claude/` or `.github/`.
- Dependency manifests and lock files, such as `pyproject.toml` and `package.json`.
- `README.md` and other top-level documentation.

New end-to-end test specs belong in `specs/e2e/`. Never read or create `.env` files.

### End-to-end tests

Do not run the specs in `specs/e2e/`. They drive a running application through a browser,
which these workflows do not provision. State in the summary that end-to-end tests were
skipped and that they can be run manually with `uv run adws/adw_test.py <issue_number>`.
When a plan lists an end-to-end spec among its validation commands, skip that command and
run every other one.

### Reporting

- Finish with exactly one summary comment, posted with the `add_comment` tool. Start it
  with the `adw_id`, then state what was done, what was skipped and anything that failed.
- If you cannot complete the workflow, still post the summary comment and explain where it
  stopped and why.
- If there is nothing to do, call the `noop` tool with a short explanation.
