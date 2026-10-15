---
name: instant-factory-setup
description: Sets a product repo up for the Instant Factory blueprint. It inspects the repo, CI, branch protection, git history and GitHub access, asks the maintainer only what's missing, opens the setup pull request with the repo's factory charter, and proves the controls work before any production work. Use it from the blueprint's setup init routine, or by hand to set up or redo setup.
metadata:
  adapted-from: https://github.com/mikolalysenko/instant-factory/blob/main/BOOTSTRAP.md
  original-author: Mikola Lysenko
---
# Instant Factory setup

This is Mikola Lysenko's [bootstrap](https://github.com/mikolalysenko/instant-factory/blob/main/BOOTSTRAP.md), adapted to Fabrica. The big difference: **Fabrica already made the routines** when someone ran `fabrica apply`, and it serves their instructions live from the blueprint. So setup doesn't create routines or write agent instructions into the repo. It gets the repo ready, writes down the repo's facts in a charter, and proves the controls.

Until setup is done, every routine stops on its own: with no charter and no readable stop control, the `factory-ground-rules` skill treats the factory as stopped.

Load the `factory-ground-rules` skill too. Setup creates what it describes: the stop control, the labels, the ledger and the charter.

The result is:

- a **setup pull request** that adds the charter (and CI, if the repo had none), waiting for a person's approval,
- a **setup issue** labeled `factory:setup` with the evidence from the acceptance runs,
- a **report** to the person: what changed, what was proven, the gaps, the shutdown delay and the steps left.

**Preserve existing work and safeguards.** Don't remove workflows, checks, labels, protections or bots that are already there. Add to them.

## 1. Inspect first

Look before you ask. Nothing in this step writes anything.

- **GitHub access.** Install `gh` if it's missing. If it isn't logged in, ask the person to run `gh auth login` (in Claude Code: `! gh auth login`). Find out who you act as (`gh api user`) and what you can do (`gh api repos/OWNER/REPO --jq .permissions`). The routines' access is a separate thing from yours: in Claude routines it comes from the Claude GitHub App or `/web-setup`, and GitHub event triggers need the App. Treat `repo_access_denied` as a problem to solve before anything else.
- **CI.** Which workflows exist, what they run, and whether they pass on the default branch. Find the repo's real validation commands (type check, lint, tests, build: only the ones it really has, since not every project has all four) and run them yourself. Run them again when something fails in a way that looks flaky. Write down every failure that was already there.
- **Branch protection.** What rulesets or classic protection the default branch has, and whether the repo's plan allows them. Public repos get protection on every plan. Private repos need GitHub Pro (personal accounts) or Team or Enterprise (organizations); moving a private repo to a free organization doesn't help. See [GitHub's rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches).
- **Secrets in history.** Cloud routines clone the repo, and making it public publishes all of it. Scan the whole history (with `gitleaks` or `trufflehog` if you have them, otherwise by searching `git log -p` for key and token patterns). **Never show a secret's value**, only where it is. **Never rewrite history without the person's explicit approval.** If you find one, the person needs to rotate it.
- **The Fabrica line.** The root `AGENTS.md` (and `CLAUDE.md`, if there is one) must start with Fabrica's line, or Fabrica won't serve the repo.
- **What's already there.** Labels, open issues and pull requests, `CODEOWNERS`, other bots and automations.

## 2. Ask once

Ask the person, in **one round** if you can, for only what you couldn't find:

- **Budget or throughput goal**, for agent usage and for GitHub Actions minutes.
- **Approvers**: the GitHub logins whose approval counts as human approval. And who else is **trusted** to feed work into the factory.
- **Routine identity**: which GitHub account the routines act as (see step 3), with the consequences explained before they choose.
- **Branch protection**, if it isn't available: upgrade the plan, change the repo's visibility, or accept that the factory can't run unattended. **Ask before any paid upgrade, visibility change or transfer**, and don't do it yourself.
- **QA areas**: which product areas, workflows or user journeys most need bug hunting, and how to run the product to test them.
- **The stop control**: suggest a repository Actions variable, `FACTORY_STATE`, since only admins can change one and collaborators can read it. If the routine identity can't read Actions variables, agree on another place that the routines can read and only people can change.

**If you can't ask** (nobody answers, or you're running headless): don't guess. Open the setup issue (labeled `factory:setup`) listing your findings and questions, and end. Someone finishes setup later by using this skill in an interactive session.

## 3. Identity and approval

Routines act as the GitHub account of whoever owns them in the runtime. Confirm who that is and what it can do, and explain the consequences before the person picks:

- **GitHub doesn't let an account approve its own pull requests.** The approver has to be a different account from the routine identity: either a separate person, or a dedicated account for the routines. Keep the approver's credentials away from the agents. A person may do the merge by hand, but that still needs the same checks and a separate approval. Merging by hand is not a way around approval. See [GitHub's review requirements](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews).
- **Prefer a routine identity without admin rights.** Protection should apply to admins too, with no bypass for the routine identity. That may stop the person from pushing straight to the default branch: **confirm that before changing protection.** Agents must not be able to change the rules that constrain them.
- **A shared identity can't tell agent actions from human ones.** Role markers (see the ground rules) trace the work, but never count as approval. The factory's own review names the commit it reviewed, and it doesn't replace the person's approval.

## 4. Put the controls in place

In this order:

1. **The stop control, set to `stopped`**, before anything else. For the suggested variable: `gh variable set FACTORY_STATE --body stopped --repo OWNER/REPO`. If you don't have admin rights, give the person the command and wait until it's done.
2. **Branch protection** on the default branch, as the person agreed. Prefer a ruleset that:
   - requires a pull request, with at least one approving review,
   - dismisses stale approvals when new commits are pushed, and requires approval of the most recent push,
   - requires the verified CI job as a status check (never weakened to get setup through),
   - requires conversations to be resolved (in case the agents can't see review threads),
   - blocks force pushes and deletion,
   - has no bypass for the routine identity.

   Show the person the exact settings and get a yes before applying them. Give them the command if you can't apply it yourself.
3. **Labels**: every label in the ground rules' table that the repo doesn't have yet.
4. **The ledger**: an issue titled "Factory ledger", labeled `factory:ledger`, pinned. Its body has the starting capacity plan. Start modest, and fit it to the budget and to how much the approvers can review:

   ```markdown
   ## Capacity plan

   | Role | State | Daily limit | Open PRs | Notes |
   |---|---|---|---|---|
   | qa | active | 3 issues | – | |
   | performance | active | 2 issues | – | |
   | ci | active | 1 PR | 1 | |
   | architecture | active | 1 PR | 1 | |
   | implementation | active | 2 PRs | 2 | |
   | pr-review | active | 10 reviews | – | |
   | merge | active | – | – | |
   | manager | active | – | – | |
   ```

5. **The setup issue**, labeled `factory:setup`, where the acceptance runs report.
6. **Issues for failures that were already there**, labeled `factory:ci`, one per failure.

## 5. Open the setup pull request

On a branch named `<prefix>setup/instant-factory`, labeled `factory:setup`:

- `.agents/skills/factory-charter/SKILL.md`, filled in from this skill's `charter-template.md`. Facts only: the process stays in the blueprint.
- A minimal CI workflow, **only if the repo had none**, running the validation commands you verified in step 1.
- Fabrica's line at the top of `AGENTS.md` (and `CLAUDE.md`), if it was missing.

Nothing else. After it's open, **avoid pushing to it** unless you have to: every commit makes its review stale.

## 6. Prove it before anything runs unattended

Ask the person to change the stop control and to start routines by hand (Claude routines have a **Run now** button; other runtimes have something similar) as you go. Check every result in GitHub yourself.

1. **Stopped.** With the stop control at `stopped`, run `implementation` and `manager`. Confirm they wrote nothing.
2. **Unreadable.** Make the stop control unreadable for a moment (for the variable: delete it), run `implementation` again, and confirm it wrote nothing. Put it back to `stopped`.
3. **Acceptance.** Set the stop control to `acceptance` and run each routine once. Each one does only its **Acceptance check** and reports on the setup issue. Confirm:
   - every role could read the repo, its issues and its pull requests, and write a comment,
   - `pr-review` reviewed the setup pull request's **current** head commit, by its SHA,
   - `merge` found that the setup pull request is blocked, and why (no human approval yet). **Nobody merges the setup pull request during these tests**,
   - `manager` paused one role in the capacity plan, and that role, run afterwards, wrote nothing; then the manager (or you) set it back to active.
4. **A stale review.** If you have to push to the setup pull request anyway, or the person agrees to one small commit: after the push, run `merge` (it must report that the factory review is for an old commit) and then `pr-review` (it must review the new head).
5. **Back to stopped.** Set the stop control to `stopped` again.

Anything you couldn't prove stays off: say so in the report, and leave it out of the capacity plan (mark the role `paused`).

## 7. Report

Post the report on the setup issue, and tell the person in plain words:

- what changed in the repo and in its settings,
- the evidence for each acceptance check (links to the comments and runs),
- the gaps, and what they mean,
- the **shutdown delay**: schedules run at most hourly, turning a schedule off doesn't stop a run that's already going, and a running agent stops at its next write check,
- how to stop the factory (the stop control) and how to turn the routines off directly if the manager can't run (the runtime's own toggle, or `fabrica destroy <factory>` in the home repo),
- the steps left: approve and merge the setup pull request, check that the next runs read the merged charter, and only then set the stop control to `running`.

Production work starts only when the person sets the stop control to `running`.
