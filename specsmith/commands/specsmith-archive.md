---
description: "[BE+FE] Archive a completed or shelved spec. Usage: /specsmith-archive <name>"
scope: both
allowed-tools: Read, Edit, Write, Glob
---

# Specsmith Archive

Archive spec: `{{ARGS}}`

Invoke the specsmith:specsmith-implementation skill, action: **archive**

The spec name is: `{{ARGS}}`

If no name was provided, use the active spec. If no active spec, list specs and ask which one to archive.
