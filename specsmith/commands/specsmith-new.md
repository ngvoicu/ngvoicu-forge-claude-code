---
description: "Create a new implementation spec through discovery interview and deep research. Usage: /specsmith-new <name>"
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, TodoRead, TodoWrite, Task, WebSearch, WebFetch, AskUserQuestion
---

# Specsmith New

**Do NOT use EnterPlanMode.** Specsmith IS the planning process.

Create a new implementation spec.

Invoke the specsmith:specsmith-implementation skill, action: **new**

The spec name is: `{{ARGS}}`

If no name was provided, ask the user for a spec name first. Once you have the name, immediately ask the user to explain the feature — do NOT start creating files or researching until they've described what they want to build.
