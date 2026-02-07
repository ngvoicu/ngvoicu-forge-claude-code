---
description: "Create a new implementation spec through discovery interview and deep research. Usage: /specsmith-new <name>"
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, TodoRead, TodoWrite, Task, WebSearch, WebFetch, AskUserQuestion
---

# Specsmith New

**Do NOT use EnterPlanMode.** Specsmith IS the planning process.

Create a new implementation spec.

Invoke the specsmith:specsmith-implementation skill, action: **new**

The spec name is: `{{ARGS}}`

If no name was provided, ask the user for a spec name first.

**One-shot support:** If the args contain a colon (e.g. `/specsmith-new auth-refactor: Refactor the auth middleware to support JWT and session tokens`), split on the first colon — everything before it is the spec name, everything after is the initial brief. Skip asking for the feature explanation and go straight to the brief quality gate / analysis with the provided text.

Otherwise, once you have the name, immediately ask the user to explain the feature — do NOT start creating files or researching until they've described what they want to build.
