# Install & Prime

## Read
.env.sample (never read .env)
.claude/adw_project.md

## Read and Execute
.claude/commands/prime.md

## Run
- Run `uv run adws/adw_tests/health_check.py` to check the environment variables, the git repository, the GitHub CLI and the Claude Code CLI

## Report
- Output the work you've just done in a concise bullet point list.
- List every error and warning from the health check, with the fix for each.
- If `./.env` does not exist, instruct the user to fill out `./.env` based on `./.env.sample`
- If `.claude/adw_project.md` still describes the ADW repository itself, instruct the user to replace its values with those of their own project
