---
name: factory-charter
description: This repo's facts for the Instant Factory, like its stop control, approvers, budget, QA areas, validation commands and branch prefix. Every factory agent reads it at the start of a run.
---
# Factory charter

Every agent of the [Instant Factory blueprint](https://github.com/staltz/fabrica-blueprint-instant-factory) reads this at the start of each run. It holds facts about this repo only. The process lives in the blueprint.

Only people change this file, through a reviewed pull request.

## Stop control

- **Where:** <for example: the repository Actions variable `FACTORY_STATE` on OWNER/REPO>
- **Read it with:** `<for example: gh api repos/OWNER/REPO/actions/variables/FACTORY_STATE --jq .value>`
- **Values:** `running` means work, `acceptance` means setup checks only, and anything else (including a failed read) means stopped.
- **Who can change it:** <for example: repo admins only; the routine identity can read it but not write it>
- **To stop the factory:** `<for example: gh variable set FACTORY_STATE --body stopped --repo OWNER/REPO>`
- **Shutdown delay:** <how long until every agent has stopped writing: runs already in progress finish their current step and stop at their next write check, so up to the longest run, about N minutes>
- **If the manager can't run, or to stop routines from starting at all:** <how to turn the routines off directly: the runtime's own toggle (like the Routines page on claude.ai), or `fabrica destroy <factory>` in the home repo>

## People

- **Approvers** (their approving review on the latest commit is the human approval): @<login>, …
- **Trusted** (their issues and pull requests enter the factory without `factory:accepted`): @<login>, …
- **Routine identity** (the GitHub account the routines act as): @<login>. <Whether it's shared with a person, and what that means.>

## Budget and capacity

- **Budget:** <money per week or month, for agent usage and for GitHub Actions minutes>
- **Throughput goal:** <for example: about 5 merged pull requests a week>
- **Review capacity:** <how many pull requests a week the approvers can review>
- **CI capacity:** <Actions minutes per month, concurrency limits, slow jobs worth knowing about>

## QA areas

| Area | What users do there | How to run it for testing |
|---|---|---|
| <area> | <workflows and user journeys> | <commands, URLs, test accounts from fixtures> |

## Validation

- **Run before every pull request:** `<commands, like the type check, lint, tests, build: only the ones this repo really has>`
- **Required checks on the default branch:** <check names, exactly as branch protection lists them>
- **Merge method:** <squash, merge or rebase>
- **Known failures at setup:** <links to the issues setup opened, or "none">

## Performance

- **What to measure, and how:** <benchmarks, page loads, bundle size, memory, CI job times, with the commands>
- **Baselines issue:** <link to the issue labeled `factory:performance` that keeps the baselines>

## Branches

- **Default branch:** `<main>`
- **Branch prefix:** `<claude/>`

## Gaps accepted at setup

- <anything setup couldn't prove or fix, and what that means; or "none">
