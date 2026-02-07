---
description: "[BE+FE] Edit an existing implementation spec. Usage: /specsmith-edit <name>"
scope: both
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, TodoRead, TodoWrite, Task, WebSearch, WebFetch, AskUserQuestion
---

# Specsmith Edit

**Do NOT use EnterPlanMode.** Specsmith IS the planning process.

Edit an existing spec with name: `{{ARGS}}`

Invoke the specsmith:specsmith-implementation skill, action: **edit**

The spec name is: `{{ARGS}}`

If no name was provided, use the active spec from `.specsmiths/active.json`. If no active spec, show available specs and ask which one to edit.
