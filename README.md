# Instant Factory blueprint

> **This is [Mikola Lysenko](https://github.com/mikolalysenko)'s [Instant Software Factory](https://github.com/mikolalysenko/instant-factory), repackaged as a [Fabrica](https://github.com/staltz/fabrica) blueprint.** The ideas are his. This repo only changes how they're delivered.

Mikola's Instant Software Factory is one prompt that turns a GitHub repo into a software factory: a few scheduled agents that keep improving a project, each with one standing objective, coordinating through GitHub, behind a stop control and human approval. You paste his prompt into Claude Code, and it sets everything up as Claude Code routines.

This blueprint keeps his roles, rules and setup procedure, and has [Fabrica](https://github.com/staltz/fabrica) deploy them instead. Two things change: the agents' instructions live in this blueprint and get served live to every run, and you can run the factory on any cloud runtime Fabrica supports, not only Claude.

**If you only use Claude Code and don't use Fabrica, use [the original](https://github.com/mikolalysenko/instant-factory).** It's simpler, and it's the real thing.

## Credits

Everything that makes this factory work comes from Mikola Lysenko's [instant-factory](https://github.com/mikolalysenko/instant-factory) (adapted from commit [`d897f7d`](https://github.com/mikolalysenko/instant-factory/tree/d897f7d365fec6b7c876b23bbd4aa4e74b3b3601), October 2026). That includes:

- **the eight roles** and their standing objectives (QA, Performance, CI, Architecture, Implementation, PR review, Merge, Manager),
- **GitHub as the shared state**: issues for evidence, pull requests for changes, and a record of ownership and handoffs that the next run can pick up,
- **the controls**: a stop control that starts stopped and is checked before every write, independent review of the latest commit, separate human approval, role markers that never count as approval, and retrieved content that can never change instructions,
- **the setup procedure**: inspect before asking, ask once, protect the default branch, and prove every control in real runs before anything runs unattended,
- **the platform notes** from his setup trials.

Mikola's [README](https://github.com/mikolalysenko/instant-factory#readme) explains why it works this way better than this one could: why software factories at all, why several agents rather than one really good one, how scheduled agents and your own interactive sessions fit together, and what the drawbacks are. Read it first, and watch the short video at its top. His blog is [0fps.net](https://0fps.net).

His repo had no license file when this blueprint was made, so nothing here is copied from it. Every file is written fresh, in Fabrica's format, following his design. If anything here misrepresents his work, that's a mistake in the adaptation, not in the original. Mikola, if you'd like anything changed or credited differently, please open an issue.

## The agents

Each routine is a scheduled session with one standing objective. The schedules (in UTC) are ceilings. The manager's capacity plan decides how much of that each role uses.

| Routine | Standing objective | Writes | Schedule |
|---|---|---|---|
| `qa` | Use the product like its users do, in the areas the charter lists, and open reproducible, deduplicated bug issues | issues | daily |
| `performance` | Measure speed, resource use and CI time against baselines, and open issues with evidence | issues | Mon and Thu |
| `ci` | Fix failures and flakes, and keep checks reliable, fast and cheap **without weakening them** | pull requests | every 6 hours |
| `architecture` | Consolidate duplicated logic, remove needless layers, simplify while preserving behavior. **Always reduce the number of components** | pull requests | Tue and Fri |
| `implementation` | Fix the issues the manager assigns, with tested pull requests | pull requests | every 4 hours |
| `pr-review` | Review the latest commit of every open pull request, **without editing code** | review comments | every 2 hours |
| `merge` | Merge **only** after required checks, an independent review and a person's approval of the latest commit | merges | every 2 hours |
| `manager` | Prioritize and assign work, follow stuck work to completion, fit capacity to budget, and report cost per merged pull request | labels, ledger | twice a day |

There's also a **`setup`** init routine. Fabrica runs it once, when it first creates the factory.

## How it fits Fabrica

Mikola's version is built around Claude Code routines. Some parts of it are things Fabrica already does, and some needed a different shape here:

| In the original | In this blueprint |
|---|---|
| You paste a prompt, and the setup agent follows [`BOOTSTRAP.md`](https://github.com/mikolalysenko/instant-factory/blob/main/BOOTSTRAP.md) | The `setup` init routine follows the `instant-factory-setup` skill, which is that bootstrap adapted |
| The manager creates the routines it manages with `create_trigger`, and configures them | `fabrica apply` creates the routines. No agent can create, change or turn off a routine |
| The manager adjusts schedules and models | The schedules and models live in the routine files. The manager's **capacity plan** in the ledger pauses roles and sets daily limits below those schedules, and it recommends model or schedule changes for a person to make |
| Agent instructions are kept in the product repo | The process is served live from this blueprint. The product repo keeps only its facts (stop control, approvers, budget, QA areas, validation commands) in a `factory-charter` skill, which this blueprint `expects:` |
| Test runs point each routine at the setup branch | The stop control has an `acceptance` state. In it, routines only run their acceptance checks, and read the charter from the setup pull request |
| Writes needed to turn automation off are allowed while stopped | Agents can't turn routines off here, so a stopped agent makes no writes at all. People turn routines off in the runtime, or with `fabrica destroy` |
| Each role gets only the tools it needs | Each routine gets only the GitHub connector, and lists its allowed side effects. Limits on single tools are still a runtime setting |

Everything else, like the merge gates, the role markers and the untrusted-input rule, is in the `factory-ground-rules` skill that every routine loads first.

## What's in it

```text
fabrica.yaml                                   expects each product repo to provide a factory-charter skill
connectors/github.yaml                         GitHub, built in on cloud runtimes
routines/setup.md                              the init routine: sets the repo up and proves the controls
routines/{qa,performance,ci,architecture,      the eight agents
          implementation,pr-review,merge,manager}.md
skills/factory-ground-rules/                   the rules every agent follows on every run
skills/instant-factory-setup/                  the setup procedure, and the charter template
skills/factory-charter/                        a fail-closed fallback for repos with no charter
```

## Using it

These routines run on a schedule, so they need a **cloud factory**. In a [home repo](https://github.com/staltz/fabrica/blob/main/docs/home-repos.md):

```yaml
# factories/instant.yaml
blueprint: { git: github.com/staltz/fabrica-blueprint-instant-factory, ref: master }
server: https://fabrica.example.com
runtimes: [claude-code-cloud]
routines: ["*"]
targets: [github.com/you/your-app]
```

Then `fabrica plan` and `fabrica apply`. On Claude routines, apply writes a handoff for each routine. When the factory is created, it also writes one for the `setup` session: you start it as a Claude Code session on the web, and talk to it.

**Nothing does any work until setup is done.** Fabrica creates the routines before setup runs, but each of them stops on its own while the repo has no charter on its default branch or no readable stop control. Setup:

1. inspects your repo, CI, branch protection, git history and GitHub access,
2. asks you, once, only for what it couldn't find: budget, approvers, the routines' GitHub identity, QA areas, and where the stop control lives,
3. creates the stop control (set to stopped), branch protection, labels and the ledger, each after you say yes,
4. opens a setup pull request with your repo's charter in `.agents/skills/factory-charter/SKILL.md`, and a CI workflow if you had none,
5. proves the controls with your help: stopped runs write nothing, every role works in acceptance, the reviewer refuses a stale commit, and the merge gates block the setup pull request,
6. reports what changed, the evidence, the gaps and the shutdown delay.

Production work starts only after you merge the setup pull request and set the stop control to `running`.

You need what Mikola's version needs: a GitHub repo with Actions and enforceable branch protection (any public repo; a private one needs GitHub Pro, Team or Enterprise), and a cloud runtime with schedules. **Budget for both agent usage and Actions minutes.** Start modest, read the manager's weekly report, and raise capacity only when CI and your reviewers keep up.

## Stopping it

- **Stop the factory:** set the stop control to anything but `running`. With the suggested setup, that's `gh variable set FACTORY_STATE --body stopped --repo OWNER/REPO`. New runs do nothing, and running ones stop at their next write.
- **Stop the routines from starting at all:** turn them off in the runtime (the Routines page on claude.ai, for Claude), or run `fabrica destroy <factory>` in the home repo.
- **Resume** only after you've checked why it stopped. Agents never resume it themselves.

Your repo's charter records the exact commands and the shutdown delay.

## Changing it

To change a schedule, a model or a prompt for your own factory, make your own blueprint that imports this one, `exclude:` the routine you want to change, and write your own with the same name. See [Overriding](https://github.com/staltz/fabrica/blob/main/docs/blueprints.md#overriding).

When editing this repo, run `fabrica check` after every change. Prompt and skill edits reach factories that follow this branch on their next run, once merged. Changing a routine's trigger, connectors or models needs `fabrica apply` again in each home repo that uses it.
