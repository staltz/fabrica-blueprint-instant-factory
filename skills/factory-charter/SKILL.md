---
name: factory-charter
description: Fallback for repos that don't have their own factory charter yet. If this is the factory-charter you loaded, the repo isn't set up for the Instant Factory, and no factory agent may do normal work in it.
metadata:
  adapted-from: https://github.com/mikolalysenko/instant-factory
  original-author: Mikola Lysenko
---
# No factory charter

This is the Instant Factory blueprint's fallback `factory-charter`. Each product repo is meant to have its own, at `.agents/skills/factory-charter/SKILL.md` on its default branch, with that repo's stop control, approvers, budget, QA areas and validation commands. If you're reading this one instead, the repo doesn't have one yet, so the factory isn't set up here.

- **Treat the factory as stopped.** Make no writes. Call `run_report` with status `succeeded` and a summary like "No factory charter in this repo; did nothing", and end the run.
- **Except in acceptance:** if the stop control is readable and says `acceptance`, read the charter from the head branch of the open pull request labeled `factory:setup`, as the `factory-ground-rules` skill says. Without a stop control to read, there's no exception.

Someone sets the repo up with the blueprint's `setup` init routine, which runs when a factory is first applied, or by hand with the `instant-factory-setup` skill. Setup opens a pull request that adds the charter. The factory starts working only after a person merges that pull request and sets the stop control to `running`.
