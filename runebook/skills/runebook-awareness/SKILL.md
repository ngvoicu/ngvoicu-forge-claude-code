---
name: runebook-awareness
description: "Assertive awareness of the .runebook/ directory for living documentation. Triggers on: 'how does X work', 'what does this endpoint do', 'what does this service do', 'what does this job do', 'what does this integration do', 'what does this page do', 'what props does X accept', 'what does this hook do', 'check the runebook', 'document this', 'update the docs', 'explain the X system', 'give me an overview of X', 'walk me through X', 'how do these pieces fit together', or when a .runebook/ directory exists and code changes are made. Automatically reads relevant entries and guides before work and updates them after changes — no prompting."
---

# Runebook Awareness

## Trigger Phrases

Activate this skill when the user says anything matching these patterns:

**Documentation queries:**
- "how does X work", "what does X do"
- "what does this endpoint do", "what does this service do"
- "what does this job do", "how does the X job work"
- "what does this integration do", "how does the X integration work"
- "what does this page do", "how does the X page work", "what does this route render"
- "what props does X accept", "how does the X component work", "what variants does X have"
- "what does this hook do", "how does the X hook work", "what does useX return"
- "explain the flow for X", "how does the X flow work"
- "what calls X", "what depends on X"
- "check the runebook", "look it up in the runebook"
- When any of these are detected, read the relevant `.runebook/` entry and present it

**Guide queries (high-level "how does it work" questions):**
- "how does authentication work", "explain the payment system", "walk me through onboarding"
- "give me an overview of X", "I'm new to this codebase", "help me understand X"
- "what's the architecture for X", "how do these pieces fit together"
- When a question is about a feature domain rather than a specific component, check `guides/` first — guides provide the narrative overview, component entries provide the details
- If a matching guide exists, present it; if not, check if component entries can answer the question directly

**Documentation updates:**
- "update the docs", "document this", "add this to the runebook"
- "the runebook is stale", "docs are out of date"
- When any of these are detected, invoke `/runebook-update`

**Auto-detection (assertive):**
- When `.runebook/` directory exists in the project root — always activate
- Before ANY code implementation work — read relevant entries
- After ANY code changes — update affected entries automatically

## Before ANY Code Work

When `.runebook/` exists and you're about to implement, modify, or debug code:

1. **Identify affected components**
   - From the user's request, determine which endpoints, services, jobs, integrations, pages, components, or hooks will be touched
   - Map file paths to runebook entries via `source_files` frontmatter
   - Use path-based heuristics when `source_files` match is ambiguous:
     - `app/**/page.*`, `pages/**` (excluding `pages/api/`) → pages
     - `components/**`, `ui/**`, `shared/**` → components
     - `hooks/**`, `composables/**` → hooks

2. **Read relevant entries automatically**
   - Read each affected entry from `.runebook/<type>/<name>.md`
   - Also check if a guide covers this domain — read `guides/*.md` frontmatter to see if any guide's `covers` list includes the affected entries. If so, skim the guide for context about how the component fits into the bigger picture.
   - Surface key details to yourself (do not dump the entire entry to the user unless asked):
     - Current behavior and business logic
     - Dependencies and cross-references
     - Edge cases and error handling
     - Recent changelog entries
     - How this fits into the broader system (from guides)
   - Use this context to inform your implementation

3. **Note gaps**
   - If a component you're about to modify has no entry, note it: "No runebook entry exists for this component — I'll create one after making changes."
   - Do not block work on missing documentation

## After ANY Code Changes

**This is assertive mode — update automatically, do not ask permission.**

After completing code changes (new features, bug fixes, refactors), follow the update procedure from the `runebook-implementation` skill, action: **update** (without name argument — auto-detect from changes).

Key behavioral rules:
- **Identify** which files were created, modified, or deleted and which runebook entries cover them
- **Update** affected entries — re-read source, update sections, preserve human context, append changelog
- **Create** entries for new components using the appropriate template
- **Update guides** if any affected entries are covered by a guide — update the narrative sections to reflect the new reality, preserve human-written sections
- **Update `index.md`** if new entries were created or metadata changed
- **Announce briefly**: "Updated runebook: `endpoints/create-user`, `services/auth-service` + guide: `authentication`" — don't dump full entries

## Documentation Queries

When the user asks about how something works:

1. Search `.runebook/` for matching entries:
   - By name (fuzzy match across all type directories)
   - By tag (grep frontmatter for matching tags)
   - By source file (grep frontmatter for the file path)
2. Read and present the matching entry
3. If the entry seems stale (based on `updated` date vs current code), mention it: "Note: this entry was last updated on X. The source has changed since — let me verify it's still accurate."

## Integration with Other Plugins

**With specsmith:** When implementing a spec, read relevant runebook entries during Phase 3 (implementation) to understand the current state of components being modified. After each implementation step, update the affected runebook entries.

**With uispec:** When syncing UI specs, check runebook endpoint entries for consistency with the API documentation in uispec.

## Proactive Suggestions

When the conversation suggests runebook-related needs but no runebook exists:
- User describes a complex system with many components
- User asks "how does X work" and you have to scan code to answer
- User is onboarding to an unfamiliar codebase

Suggest: "This codebase would benefit from a runebook — structured documentation that stays in sync with code. Want me to set one up? `/runebook-init`"
