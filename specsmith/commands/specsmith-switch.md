---
description: "Switch to a different spec (make it active). Usage: /specsmith-switch <name>"
scope: both
allowed-tools: Read, Edit, Write, Glob, Grep, TodoWrite
---

# Specsmith Switch

Switch to spec: `{{ARGS}}`

Invoke the specsmith:specsmith-implementation skill, action: **switch**

The spec name is: `{{ARGS}}`

If no name was provided, list available specs and ask which one to switch to.
