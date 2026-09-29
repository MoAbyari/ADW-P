---
# Project-specific setup for the ADW agentic workflows.
#
# This is the one workflow file to edit when you add ADW to a project. It tells GitHub
# Actions how to prepare the project before the agent starts. After editing it, run
# `gh aw compile --strict` and commit the regenerated .lock.yml files.
#
# The values below suit the ADW repository itself, which needs only Python.

# Domains the agent may reach. `defaults` covers basic infrastructure only. Add an
# ecosystem for each package manager the project uses, for example `python` or `node`.
network:
  allowed:
    - defaults

# Language runtimes to install. Add what the project needs. Python is listed because the
# validation commands in .claude/adw_project.md use it.
runtimes:
  python:
    version: "3.12"
#   uv:
#     version: "latest"
#   node:
#     version: "24"

# Commands that install the project's dependencies. They run after the pull request branch
# is checked out, so the dependencies match the code under test. Secrets are not available
# here.
# pre-agent-steps:
#   - name: Install dependencies
#     run: npm ci
---
