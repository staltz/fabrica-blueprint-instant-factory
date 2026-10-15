---
description: Set the repo up for the Instant Factory, and prove its controls before any production work
trigger: { on: init }
# Setup asks the maintainer questions, so it runs in a session a person
# starts and can talk to: a Claude Code session on the web, or a Codex task.
runtimes: [claude-code-cloud, codex]
skills: [instant-factory-setup, factory-ground-rules, factory-charter]
connectors: [github]
side_effects:
  - github.issue_open
  - github.issue_comment
  - github.issue_edit
  - github.push
  - github.open_pr
  - github.repo_settings
models:
  - { model: claude-opus-5-5, effort: xhigh }
  - default
---
You're setting up an **Instant Factory** on this repo: a few scheduled agents, each with one standing objective, that keep improving the repo behind a stop control, independent review and human approval. The idea and the procedure are Mikola Lysenko's ([instant-factory](https://github.com/mikolalysenko/instant-factory)).

Fabrica has already created the factory's routines. They do nothing until setup is done, because without a charter and a readable stop control they stop on their own.

Follow the `instant-factory-setup` skill from start to end. It also tells you when to load `factory-ground-rules`.

Change repo settings (the stop control, branch protection, labels) only after the person has said yes to each change. If you can't talk to a person in this session, don't guess: open the setup issue with your findings and questions, and stop there.
