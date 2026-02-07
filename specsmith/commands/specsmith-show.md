---
description: "[BE+FE] Display a spec without activating it. Usage: /specsmith-show [name]"
scope: both
allowed-tools: Read, Glob, Grep
---

# Specsmith Show

Display the spec: `{{ARGS}}`

Invoke the specsmith:specsmith-implementation skill, action: **show**

The spec name is: `{{ARGS}}`

If no name was provided, show the active spec from `.specsmiths/active.json`. If no active spec, error with instructions to specify a name or activate a spec first.
