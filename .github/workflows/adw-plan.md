---
name: ADW Plan
description: Write an implementation plan for a GitHub issue and open a draft pull request containing it.

on:
  slash_command:
    name: adw_plan
    events: [issues, issue_comment]

permissions:
  contents: read
  issues: read
  pull-requests: read

imports:
  - shared/adw-base.md
  - shared/adw-project.md

timeout-minutes: 20
max-turns: 100

safe-outputs:
  create-pull-request:
    draft: true
    preserve-branch-name: true
  add-comment:
    max: 1
  noop:
---

# ADW: plan

Work on issue #${{ github.event.issue.number }}. Fetch its title and body with the GitHub
tools before you start. The text that triggered this run was:

"${{ steps.sanitized.outputs.text }}"

Complete the phases below in order. Stop at the first phase that fails and report it. This
workflow only plans: do not implement anything.

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
`title` and `body` fields. Commit the plan file.

## 4. Pull request

Call `create_pull_request` with the branch you created.

- Title: `<issue_class>: #<issue_number> - <issue title>`
- Body: a summary of the issue, the path of the plan file and the `adw_id`. State that the
  pull request holds the plan only.

Comment `/adw_build` on the pull request afterwards to implement the plan.
