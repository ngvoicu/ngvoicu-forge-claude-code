---
description: "List all specs with interactive picker. Usage: /specsmith-list"
scope: both
allowed-tools: Read, Glob, Grep, AskUserQuestion
---

# Specsmith List

List all available specs with interactive selection.

Invoke the specsmith:specsmith-implementation skill, action: **list**

Display all specs from `.specsmiths/` and use AskUserQuestion to let the user pick which spec to work on. After selection, offer actions: show, edit, switch, archive.
