---
name: ADW Test
description: Run the validation suite on a pull request, fix failures and push the fixes to its branch.

on:
  slash_command:
    name: adw_test
    events: [pull_request_comment]

permissions:
  contents: read
  issues: read
  pull-requests: read

imports:
  - shared/adw-base.md
  - shared/adw-project.md

timeout-minutes: 45
max-turns: 250

safe-outputs:
  push-to-pull-request-branch:
  add-comment:
    max: 1
  noop:
---

# ADW: test

Work on pull request #${{ github.event.issue.number }}. Its branch is already checked out.
The text that triggered this run was:

"${{ steps.sanitized.outputs.text }}"

Take the `issue_class` from the branch name.

## 1. Test

Follow `.claude/commands/test.md` to run the validation suite.

- For each failing test, follow `.claude/commands/resolve_failed_test.md` with that test's
  JSON result as the argument, then run the suite again.
- Make at most 4 attempts in total. If tests still fail after the fourth, stop fixing and
  report the remaining failures.
- Commit any fixes on the checked-out branch. Do not create a new branch.

## 2. Push

If you committed fixes, call `push_to_pull_request_branch` to deliver them.

## 3. Report

In the summary comment, list every test with its result. Put failures first and include
the command that reproduces each one.
