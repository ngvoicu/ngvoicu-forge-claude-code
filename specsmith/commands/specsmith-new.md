---
description: "Create a new implementation spec through discovery interview and deep research. Usage: /specsmith-new <name>: <brief>"
scope: both
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, TodoRead, TodoWrite, Task, WebSearch, WebFetch, AskUserQuestion
---

# Specsmith New

**Do NOT use EnterPlanMode.** Specsmith IS the planning process.

Create a new implementation spec.

Invoke the specsmith:specsmith-implementation skill, action: **new**

The args are: `{{ARGS}}`

**Default: one-shot (name + brief together).** If the args contain a colon (e.g. `/specsmith-new auth-refactor: Refactor the auth middleware to support JWT and session tokens`), split on the first colon — everything before it is the spec name, everything after is the initial brief. Skip asking for the feature explanation and go straight to the brief quality gate / analysis with the provided text.

If the args have no colon, treat the entire arg as the spec name and ask the user to explain the feature.

If no args were provided at all, ask the user for both a spec name and feature explanation — ideally in one prompt: "Give me a name and brief for this spec (e.g. `auth-refactor: Refactor the auth middleware to support JWT`)."
