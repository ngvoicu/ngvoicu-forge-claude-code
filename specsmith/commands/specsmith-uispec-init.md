---
description: "Initialize .uispec/ in the frontend project with API contracts from this backend. Usage: /specsmith-uispec-init"
scope: backend
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, LS, AskUserQuestion
---

# Specsmith UISpec Init

Initialize `.uispec/` in the sibling frontend project. Scans this backend for all API endpoints, creates the directory structure, and generates endpoint specs with suggested implementations.

Invoke the specsmith:specsmith-uispec skill, action: **init**

First-time setup for full-stack workflow. Run this once, then use `/specsmith-uispec-sync` after API changes.
