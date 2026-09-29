---
name: ADW Build
description: Implement the plan held in a pull request and push the result to its branch.

on:
  slash_command:
    name: adw_build
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

# ADW: build

Work on pull request #${{ github.event.issue.number }}. Its branch is already checked out.
The text that triggered this run was:

"${{ steps.sanitized.outputs.text }}"

## 1. Find the plan

The plan is the file this pull request adds under `specs/`, named
`issue-<issue_number>-adw-<adw_id>-sdlc_planner-<name>.md`. Compare the branch with the
default branch to find it. Take the issue number and `issue_class` from the plan and the
branch name.

If the pull request contains no plan, post the summary comment saying that `/adw_plan`
must run on the issue first, and stop.

## 2. Build

Follow `.claude/commands/implement.md` with the path of the plan file as the argument.
Commit the implementation on the checked-out branch. Do not create a new branch.

## 3. Push

Call `push_to_pull_request_branch` to deliver the commits to the pull request.

Comment `/adw_test` on the pull request afterwards to run the test phase.
