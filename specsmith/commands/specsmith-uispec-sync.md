---
description: "Sync API changes to frontend's .uispec/ directory. Usage: /specsmith-uispec-sync"
scope: backend
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, LS
---

# Specsmith UISpec Sync

Detect backend API changes and sync them to the frontend's `.uispec/`. Updates `openapi.yaml`, endpoint specs, component patterns, and runs validation.

Invoke the specsmith:specsmith-uispec skill, action: **sync**

Requires prior `/specsmith-uispec-init`. Preserves all frontend-owned sections (UI Guidelines, design-system.md, etc.).
