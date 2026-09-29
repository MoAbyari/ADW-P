# ADW Project Configuration

This file describes the project that ADW works on. The prompt templates in
`.claude/commands/` read it to learn where the code lives and how to validate a change, so
the templates themselves stay the same from project to project.

When you add ADW to a project, replace the values below with that project's own. The values
shipped here describe the ADW repository itself and serve as a working example.

## Relevant Files

- `README.md` - Project overview and setup instructions.
- `adws/**` - The ADW scripts. They are uv single-file Python scripts; run one with
  `uv run <script_name>`.
- `.claude/commands/**` - The prompt templates shared by the manual and automated modes.
- `.github/workflows/*.md` - The GitHub Agentic Workflows definitions for the automated mode.
- `scripts/**` - Utility scripts for cleaning up issues and pull requests.

## Dependencies

How to add a library to this project:

- Python scripts in `adws/` declare their dependencies inline, in the `# /// script` block
  at the top of each file. Add the package name to that block.

## Validation Commands

Run from the project root, in the order listed. Each entry has a name, the command, and
what it proves.

1. **python_syntax_check**
   - Command: `python3 -m py_compile adws/*.py adws/adw_modules/*.py adws/adw_tests/*.py adws/adw_triggers/*.py`
   - Purpose: Validates Python syntax by compiling every ADW source file to bytecode.

2. **shell_syntax_check**
   - Command: `bash -n scripts/clear_issue_comments.sh && bash -n scripts/delete_pr.sh`
   - Purpose: Validates the syntax of the utility shell scripts without running them.

## Application

Used by end-to-end tests, which drive the running application through a browser. Leave
every value as `none` when the project has no application to run.

- Start command: none
- Stop command: none
- Reset command: none
- URL: none

## End-to-End Specs

- Location: `specs/e2e/`
- One Markdown file per test, named `test_<descriptive_name>.md`. The format is described
  in `.claude/commands/test_e2e.md`.
