---
name: ADW Plan Build
description: Plan and implement a GitHub issue, then open a draft pull request.

on:
  slash_command:
    name: adw_plan_build
    events: [issues, issue_comment]

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
  create-pull-request:
    draft: true
    preserve-branch-name: true
  add-comment:
    max: 1
  noop:
---

# ADW: plan and build

Work on issue #${{ github.event.issue.number }}. Fetch its title and body with the GitHub
tools before you start. The text that triggered this run was:

"${{ steps.sanitized.outputs.text }}"

Complete the phases below in order. Stop at the first phase that fails and report it.

## 1. Classify

Follow `.claude/commands/classify_issue.md` with the issue title and body as the argument.
If the issue is not a chore, bug or feature, post the summary comment saying so and stop.

## 2. Branch

Create a local branch from the current commit named
`<issue_class>-issue-<issue_number>-adw-<adw_id>-<concise-name>`, where the concise name is
three to six lowercase words joined by hyphens.

## 3. Plan

Follow the planning template that matches the classification:
`.claude/commands/chore.md`, `.claude/commands/bug.md` or `.claude/commands/feature.md`.
Its arguments are the issue number, the `adw_id` and the issue as JSON with `number`,
`title` and `body` fields. Commit the plan file on its own.

## 4. Build

Follow `.claude/commands/implement.md` with the path of the plan file as the argument.
Commit the implementation.

## 5. Pull request

Call `create_pull_request` with the branch you created.

- Title: `<issue_class>: #<issue_number> - <issue title>`
- Body: a summary of the issue, the path of the plan file, the `adw_id`, a checklist of
  what was done and the key changes.

Comment `/adw_test` on the pull request afterwards to run the test phase.
