# AI Developer Workflow (ADW) System

ADW automates software development by integrating GitHub issues with Claude Code CLI to classify issues, generate plans, implement solutions, and create pull requests.

## Two Ways to Run ADW

| | Automated | Manual |
|---|---|---|
| Where it runs | GitHub Actions, through [GitHub Agentic Workflows](https://github.github.com/gh-aw/) | Your machine |
| How you start it | Comment a slash command on an issue or pull request | Run a script in this folder |
| Defined in | `.github/workflows/adw-*.md` | `adws/*.py` |
| Progress reporting | A start and finish status comment, plus one summary comment | A comment on the issue at every step |
| Pull request | Opened as a draft when the run finishes | Opened after the planning step |
| End-to-end tests | Skipped | Included |

Both modes use the same prompt templates in `.claude/commands/`, so a change to a template
affects both.

This document covers the manual scripts. See
[Automated Mode](#automated-mode-github-agentic-workflows) for the slash commands.

## Quick Start

### 1. Set Environment Variables

```bash
export GITHUB_REPO_URL="https://github.com/owner/repository"
export ANTHROPIC_API_KEY="sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
export CLAUDE_CODE_PATH="/path/to/claude"  # Optional, defaults to "claude"
export GITHUB_PAT="ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"  # Optional, only if using different account than 'gh auth login'
```

### 2. Install Prerequisites

```bash
# GitHub CLI
brew install gh              # macOS
# or: sudo apt install gh    # Ubuntu/Debian
# or: winget install --id GitHub.cli  # Windows

# Claude Code CLI
# Follow instructions at https://docs.anthropic.com/en/docs/claude-code

# Python dependency manager (uv)
curl -LsSf https://astral.sh/uv/install.sh | sh  # macOS/Linux
# or: powershell -c "irm https://astral.sh/uv/install.ps1 | iex"  # Windows

# Authenticate GitHub
gh auth login
```

### 3. Run ADW

```bash
cd adws/

# Process a single issue manually (plan + build)
uv run adw_plan_build.py 123

# Process a single issue with full pipeline (plan + build + test)
uv run adw_plan_build_test.py 123

# Run individual phases
uv run adw_plan.py 123     # Planning phase only
uv run adw_build.py        # Build phase only (requires existing plan)
uv run adw_test.py 123     # Test phase only

# Run continuous monitoring (polls every 20 seconds)
uv run adw_triggers/trigger_cron.py
```

## Script Usage Guide

### adw_plan_build.py - Complete Workflow

Executes the complete ADW workflow for a specific GitHub issue by chaining the planning and building phases.

```bash
# Basic usage
uv run adw_plan_build.py 456

# What it does:
# 1. Fetches issue #456 from GitHub
# 2. Creates feature branch
# 3. Classifies issue type (/chore, /bug, /feature)
# 4. Generates implementation plan
# 5. Implements the solution
# 6. Creates commits and pull request
```

### adw_plan_build_test.py - Full Pipeline

Executes the complete ADW pipeline including testing for a specific GitHub issue.

```bash
# Basic usage
uv run adw_plan_build_test.py 456

# With specific ADW ID
uv run adw_plan_build_test.py 456 a1b2c3d4

# What it does:
# 1. Runs planning phase (adw_plan.py)
# 2. Runs implementation phase (adw_build.py)
# 3. Runs testing phase (adw_test.py)
# 4. All phases are chained via piped state
```

### adw_plan.py - Planning Phase

Handles the initial planning phase of the workflow.

```bash
# Basic usage
uv run adw_plan.py 456

# What it does:
# 1. Fetches issue details from GitHub
# 2. Classifies issue type
# 3. Creates feature branch
# 4. Generates implementation plan
# 5. Commits plan and creates PR

# Outputs state JSON for chaining with build phase
```

### adw_build.py - Implementation Phase

Implements the solution based on an existing plan.

```bash
# Basic usage (requires existing plan)
uv run adw_build.py

# Can also be chained:
uv run adw_plan.py 456 | uv run adw_build.py

# What it does:
# 1. Finds existing plan from state or by searching
# 2. Implements the solution based on plan
# 3. Commits implementation
# 4. Pushes changes and updates PR
```

### adw_test.py - Test Phase

Runs the application test suite and reports results.

```bash
# Basic usage
uv run adw_test.py 456

# Skip E2E tests
uv run adw_test.py 456 --skip-e2e

# What it does:
# 1. Fetches issue details (if not in state)
# 2. Runs application test suite
# 3. Reports results to issue
# 4. Creates commit with test results
# 5. Pushes changes and updates PR
```

### adw_triggers/trigger_cron.py - Automated Monitoring

Continuously monitors GitHub for new issues or "adw" comments.

```bash
# Start monitoring
uv run adw_triggers/trigger_cron.py

# Processes issues when:
# - New issue has no comments
# - Latest comment on any issue is exactly "adw"

# Example log output:
# 2024-01-15 10:30:45 - Starting ADW cron trigger
# 2024-01-15 10:30:46 - Issue #123 has no comments - processing
# 2024-01-15 10:30:47 - Issue #456 - latest comment is 'adw' - processing
```

## Automated Mode (GitHub Agentic Workflows)

The webhook server has been replaced by GitHub Agentic Workflows. GitHub starts the run
itself, so there is no server to host and no tunnel to keep open.

Write the command as the first word of an issue body or comment:

| Command | Where | Result |
|---|---|---|
| `/adw_plan` | Issue | Draft pull request containing the plan |
| `/adw_plan_build` | Issue | Draft pull request with the plan and implementation |
| `/adw_plan_build_test` | Issue | As above, plus the validation suite with automatic fixes |
| `/adw_build` | Pull request | Implements the plan and pushes to the pull request branch |
| `/adw_test` | Pull request | Runs the validation suite and pushes any fixes |

Only users with admin, maintainer or write access can trigger a run.

### Setup

```bash
gh extension install github/gh-aw --pin v0.89.21
gh aw secrets set ANTHROPIC_API_KEY
```

Then, in the repository settings under Actions > General, allow GitHub Actions to create
pull requests. Without that setting the run opens an issue in place of the pull request.

### Files

```
.github/workflows/
├── shared/adw-base.md           # Engine, tools and rules shared by all five
├── shared/adw-project.md        # Runtimes, network access and dependency installation
├── adw-plan.md
├── adw-build.md
├── adw-test.md
├── adw-plan-build.md
├── adw-plan-build-test.md
└── *.lock.yml                   # Generated by `gh aw compile`; do not edit
```

Each `.md` file has YAML frontmatter followed by the prompt. After changing frontmatter,
run `gh aw compile --strict` and commit the regenerated `.lock.yml` file. Changes to the
prompt take effect without recompiling.

### How It Differs from a Manual Run

- **The agent cannot push.** It commits locally inside a sandbox. GitHub applies the
  commits after the run, once they pass an automated threat check.
- **Pull requests are drafts** and are opened when the run finishes.
- **Some files are held for review.** A pull request that changes a dependency manifest
  (`pyproject.toml`, `package.json`, lock files), `README.md`, `.claude/` or `.github/` is
  still created, but it receives a change-request review listing those files.
- **End-to-end tests are skipped.** They drive a running application through a browser,
  which the workflows do not provision. Run them manually with `uv run adw_test.py <issue>`.
- **Limits apply to each run**: a time limit and a turn limit set in the workflow
  frontmatter, and a spending cap of 1000 AI credits (10 US dollars) by default.

### Useful Commands

```bash
gh aw status                 # List workflows and whether they are enabled
gh aw logs adw-plan-build    # Download logs for recent runs
gh aw audit <run-id>         # Detailed report for one run
```

## Project Configuration

ADW learns about the project it works on from `.claude/adw_project.md`: the relevant
files, how to add a dependency, the validation commands and how to run the application.
The prompt templates read that file, so they need no changes from project to project.

The test phase runs the commands listed under `Validation Commands` in that file. With no
commands listed, the test phase reports that no test results were found.

## How ADW Works

1. **Issue Classification**: Analyzes GitHub issue and determines type:
   - `/chore` - Maintenance, documentation, refactoring
   - `/bug` - Bug fixes and corrections
   - `/feature` - New features and enhancements

2. **Planning**: `sdlc_planner` agent creates implementation plan with:
   - Technical approach
   - Step-by-step tasks
   - File modifications
   - Testing requirements

3. **Implementation**: `sdlc_implementor` agent executes the plan:
   - Analyzes codebase
   - Implements changes
   - Runs tests
   - Ensures quality

4. **Integration**: Creates git commits and pull request:
   - Semantic commit messages
   - Links to original issue
   - Implementation summary

## Common Usage Scenarios

### Process a bug report
```bash
# User reports bug in issue #789
uv run adw_plan_build.py 789
# ADW analyzes, creates fix, and opens PR
```

### Run full pipeline
```bash
# Complete pipeline with testing
uv run adw_plan_build_test.py 789
# ADW plans, builds, and tests the solution
```

### Run individual phases
```bash
# Plan only
uv run adw_plan.py 789

# Build based on existing plan
uv run adw_build.py

# Test the implementation
uv run adw_test.py 789
```

### Enable automatic processing
```bash
# Start cron monitoring
uv run adw_triggers/trigger_cron.py
# New issues are processed automatically
# Users can comment "adw" to trigger processing
```

### Trigger from GitHub without running anything locally
```
# Comment on issue #789:
/adw_plan_build_test
# GitHub Actions plans, builds and tests, then opens a draft PR
```

## Troubleshooting

### Environment Issues
```bash
# Check required variables
env | grep -E "(GITHUB|ANTHROPIC|CLAUDE)"

# Verify GitHub auth
gh auth status

# Test Claude Code
claude --version
```

### Common Errors

**"Claude Code CLI is not installed"**
```bash
which claude  # Check if installed
# Reinstall from https://docs.anthropic.com/en/docs/claude-code
```

**"Missing GITHUB_PAT"** (Optional - only needed if using different account than 'gh auth login')
```bash
export GITHUB_PAT=$(gh auth token)
```

**"Agent execution failed"**
```bash
# Check agent output
cat agents/*/sdlc_planner/raw_output.jsonl | tail -1 | jq .
```

### Debug Mode
```bash
export ADW_DEBUG=true
uv run adw_plan_build.py 123  # Verbose output
```

## Configuration

### ADW Tracking
Each workflow run gets a unique 8-character ID (e.g., `a1b2c3d4`) that appears in:
- Issue comments: `a1b2c3d4_ops: ✅ Starting ADW workflow`
- Output files: `agents/a1b2c3d4/sdlc_planner/raw_output.jsonl`
- Git commits and PRs

### Model Selection
Edit `adw_modules/agent.py` line 129 to change model:
- `model="sonnet"` - Faster, lower cost (default)
- `model="opus"` - Better for complex tasks

### Modular Architecture
The system uses a modular architecture with composable scripts:

- **State Management**: `ADWState` class enables chaining workflows via JSON piping
- **Git Operations**: Centralized git operations in `git_ops.py`  
- **Workflow Operations**: Core business logic in `workflow_ops.py`
- **Agent Integration**: Standardized Claude Code CLI interface in `agent.py`

### Script Chaining
Scripts can be chained using pipes to pass state:
```bash
# Chain planning and building
uv run adw_plan.py 123 | uv run adw_build.py

# Chain full pipeline
uv run adw_plan.py 123 | uv run adw_build.py | uv run adw_test.py

# Or use the convenience script
uv run adw_plan_build_test.py 123

# State is automatically passed between scripts
```

### Output Structure
```
agents/
├── a1b2c3d4/
│   ├── sdlc_planner/
│   │   └── raw_output.jsonl
│   └── sdlc_implementor/
│       └── raw_output.jsonl
adw_modules/
├── agent.py
├── data_types.py
├── github.py
├── git_ops.py
├── state.py
├── utils.py
└── workflow_ops.py
```

## Security Best Practices

- Store tokens as environment variables, never in code
- Use GitHub fine-grained tokens with minimal permissions
- Set up branch protection rules
- Require PR reviews for ADW changes
- Monitor API usage and set billing alerts

## Technical Details

### Core Components
- `adw_modules/agent.py` - Claude Code CLI integration
- `adw_modules/data_types.py` - Pydantic models for type safety
- `adw_modules/github.py` - GitHub API operations
- `adw_modules/git_ops.py` - Git operations (branching, commits, PRs)
- `adw_modules/state.py` - State management for workflow chaining
- `adw_modules/workflow_ops.py` - Core workflow operations (planning, building)
- `adw_modules/utils.py` - Utility functions
- `adw_plan_build.py` - Main workflow orchestration (plan & build)
- `adw_plan_build_test.py` - Full pipeline orchestration (plan & build & test)
- `adw_plan.py` - Planning phase workflow
- `adw_build.py` - Implementation phase workflow
- `adw_test.py` - Testing phase workflow

### Branch Naming
```
{type}-{issue_number}-{adw_id}-{slug}
```
Example: `feat-456-e5f6g7h8-add-user-authentication`
