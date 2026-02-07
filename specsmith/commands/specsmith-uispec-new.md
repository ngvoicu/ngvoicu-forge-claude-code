---
description: "Create a UI implementation spec from a .uispec/ endpoint. Usage: /specsmith-uispec-new [endpoint][: context]"
scope: frontend
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, TodoRead, TodoWrite, Task, WebSearch, WebFetch, AskUserQuestion
---

# Specsmith UISpec New

**Do NOT use EnterPlanMode.** Specsmith IS the planning process.

Create a UI implementation spec from a `.uispec/` endpoint. Full specsmith workflow: discovery → research → planning → implementation → validation.

Invoke the specsmith:specsmith-uispec skill, action: **new**

The args are: `{{ARGS}}`

**No args:** Show unimplemented endpoints from `.uispec/`, let user pick one.

**With endpoint:** Start full workflow for that endpoint.

**With context (colon syntax):** If args contain a colon (e.g. `/specsmith-uispec-new login: JWT replaced sessions — remove token refresh`), split on the first colon — everything before is the endpoint name, everything after is implementation context that guides decisions.
