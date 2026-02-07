---
description: "Permanently delete a spec and all companion files. Usage: /specsmith-drop <name>"
scope: both
allowed-tools: Read, Write, Bash, Glob, AskUserQuestion
---

# Specsmith Drop

Delete spec: `{{ARGS}}`

Invoke the specsmith:specsmith-implementation skill, action: **drop**

The spec name is: `{{ARGS}}`

If no name was provided, list specs and ask which one to delete. Always confirm with the user before deleting.
