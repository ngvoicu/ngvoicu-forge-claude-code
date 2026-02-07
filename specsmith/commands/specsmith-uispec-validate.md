---
description: "[FE] Validate UI code against .uispec/ specs with PASS/FAIL checks. Usage: /specsmith-uispec-validate"
scope: frontend
allowed-tools: Read, Glob, Grep, LS
---

# Specsmith UISpec Validate

Validate that existing UI code follows `.uispec/` specs. Runs 6 PASS/FAIL checks across all implemented endpoints: API contract, shapes, states, design tokens, accessibility, form validation.

Invoke the specsmith:specsmith-uispec skill, action: **validate**

Reports summary table with detailed failures and suggested fixes.
