---
description: "[FE] Detect gaps between .uispec/ specs and UI code. Usage: /specsmith-uispec-detect"
scope: frontend
allowed-tools: Read, Glob, Grep, LS
---

# Specsmith UISpec Detect

Scan the frontend codebase to find gaps between `.uispec/` specs and UI implementation. Runs 5 checks: unimplemented specs, undocumented API calls, shape mismatches, design system violations, missing state handling.

Invoke the specsmith:specsmith-uispec skill, action: **detect**

Suggests `/specsmith-uispec-new <endpoint>` for each gap found.
