---
name: specsmith-implementation
description: "Core implementation logic for specsmith commands. Do not invoke directly - use the /specsmith-* commands instead."
user-invocable: false
---

# Spec Smith Implementation

This skill contains the core logic for all specsmith commands. It is invoked by the thin command wrappers.

## How It Works

All state lives in `.specsmiths/` at the project root. Each spec consists of:
- `<n>.md` — the full specification with implementation steps
- `<n>.research.md` — evidence base from the research phase
- `<n>.questions.md` — discovery log (questions asked, answers received, decisions made)

A `.specsmiths/active.json` file tracks which spec is currently active.

## Error Handling

Before executing any action, check for and handle these edge cases:

- **Spec name collisions**: Before creating a new spec with `/specsmith-new`, check if `<n>.md` already exists in `.specsmiths/`. If it does, error: "A spec named `<n>` already exists. Use `/specsmith-edit <n>` to revise it, or choose a different name."
- **Missing .specsmiths directory**: On any command except `/specsmith-new`, check that `.specsmiths/` exists. If not, explain: "No `.specsmiths/` directory found. Run `/specsmith-new <name>` to create your first spec."
- **Corrupted active.json**: When reading `.specsmiths/active.json`, validate that it contains valid JSON and that the referenced spec file actually exists. If broken, offer to rebuild: scan `.specsmiths/*.md` files, list found specs, and ask the user which (if any) should be active. Rewrite `active.json` accordingly.
- **Invalid spec names**: Validate that spec names contain only alphanumeric characters, hyphens, and underscores. If the user provides a name with spaces or special characters, auto-slugify it (e.g. "my cool feature" → "my-cool-feature") and confirm with the user before proceeding.
- **Git merge conflicts**: When reading any spec file, check for `<<<<< HEAD` conflict markers. If found, error: "Spec `<n>` has unresolved git merge conflicts. Please resolve them before continuing." Do not attempt to parse or use the conflicted file.
- **Network failures during research**: If WebSearch or WebFetch tools fail during Phase 2 research, note the failure in `.specsmiths/<n>.research.md` under a `## Research Gaps` section and continue with codebase analysis only. Do not block spec creation on web availability.

---

## Action: new

Create a new implementation spec. This is a multi-phase process that prioritizes understanding the problem deeply before proposing any solution.

### Phase 1: Discovery

**Do NOT research or plan anything yet.** First, deeply understand what the user wants.

**Do NOT use EnterPlanMode.** Specsmith IS the planning process — it handles its own discovery, research, and spec creation. Using plan mode on top of specsmith is redundant.

#### Step 1: Get the User's Explanation

**If a brief was already provided via one-shot** (the command included `name: brief`), skip the prompt — log the provided brief to `.specsmiths/<n>.questions.md` under a `## Brief` section and go straight to Step 1.5.

**If no brief was provided** (name-only or no args), ask the user:

"Explain the feature you want to implement. The more detail you give — flow, constraints, edge cases — the fewer follow-up questions I'll need."

Wait for their response. This is their **brief** — it may be a single sentence or multiple paragraphs. Accept whatever they provide without interrupting.

Log the brief to `.specsmiths/<n>.questions.md` under a `## Brief` section.

#### Step 1.5: Brief Quality Gate

Before spending analysis time, check if the brief has enough substance. A good brief should:
- Mention a **specific behavior or outcome** ("users can reset their password" not just "improve auth")
- Hint at **what triggers it or what problem it solves**
- Be more than a single vague phrase

If it fails these checks (e.g. "make a button", "add caching", "fix the thing"), prompt the user to expand:

"That's a bit thin for me to analyze — could you expand on what this should do? For example: what triggers it, what the expected behavior is, or what problem it solves."

Only proceed to Step 2 once the brief contains enough substance to meaningfully analyze.

#### Step 2: Think, Then Clarify If Needed

After receiving the brief, **analyze it methodically** against these dimensions:

- **Functional requirements:** Is the flow clear? Inputs/outputs? Triggers? What "done" looks like?
- **Edge cases & failure modes:** Missing/malformed input? Concurrency? Upstream failures? Partial failures? Retries?
- **Constraints & boundaries:** What's out of scope? Performance? Auth? Greenfield vs existing data? Compliance?
- **Integration & dependencies:** What systems does this touch? APIs? Database changes? Other teams?
- **User experience (if applicable):** Error messages? Loading states? Accessibility?

If the brief already covers enough to spec this — **don't ask unnecessary questions, just proceed to Phase 2.**

If there are genuine gaps, ambiguities, or unstated assumptions, signal the transition clearly before asking questions:

"I have a few questions before we move to research."

Then ask 3-5 focused follow-up questions. Don't re-ask things the user already covered. Be like a technical architect — only ask what you actually need to know.

After the user answers, **think again**. If more clarification is still needed, do another round. If not, move on. The goal is the minimum number of rounds needed to understand the problem, not a fixed interview process.

Log all Q&A rounds to `.specsmiths/<n>.questions.md`.

#### Question Reference

Use these as a pool to draw from — only ask what's actually unclear from the brief:

**Functional:** What exactly should this do? Walk me through the flow. What are the inputs/outputs? What triggers it? What does "done" look like?

**Edge cases:** What happens when input is missing/malformed/enormous? Concurrent access? Upstream service down/slow? Partial failures? Double-submits? Empty states, first-time use?

**Constraints:** What's explicitly NOT part of this? Performance requirements? Auth/authorization? Existing data or greenfield? Regulatory considerations?

**Integration:** What existing systems does this touch? APIs to call or expose? Database changes? Other teams affected?

**UX:** Error messages for users? Loading states? Accessibility?

#### Logging Format

Log everything to `.specsmiths/<n>.questions.md`:

```markdown
# Discovery: <n>

## Brief
<user's explanation — pasted verbatim>

## Round 1
**Q:** <question addressing a gap in the brief>
**A:** <user's answer>

**Q:** <question addressing a gap in the brief>
**A:** <user's answer>

## Round 2
...

## Decisions Made
- <Decision 1>: <what was decided and why>
- <Decision 2>: ...

## Open Questions
- <Anything still unresolved>
```

Note: Round sections are only added if clarification was needed. If the brief was comprehensive enough, the log may only contain the Brief and Decisions Made sections. If the brief was rejected by the quality gate, log the original brief and the expanded version the user provided.

#### Discovery "Done" Criteria

You have enough to proceed when:
- [ ] You can describe the full happy path in 3-5 sentences
- [ ] You know what "done" looks like (acceptance criteria)
- [ ] You've identified relevant edge cases (5+ for complex features, fewer for simple/narrowly-scoped ones)
- [ ] You know what's explicitly out of scope
- [ ] Integration points and dependencies are mapped

If these are met after reading the brief alone, tell the user: "Your brief covers everything I need. Moving to research." and proceed directly to Phase 2.

If interview rounds were needed, ask: "I think I have enough to spec this. Anything else I should know before I research?"

Only proceed to Phase 2 when:
- The above criteria are met
- Core flow is clear
- Major edge cases are identified
- Boundaries are defined
- The user confirms they're ready (or brief was sufficient)

---

### Phase 2: Deep Research (parallel subagents)

Now that requirements are understood, spawn subagents to research how to build this. Adapt the number and focus of subagents based on what you learned in discovery.

**Lightweight research escape hatch:** If the feature touches only 1-2 files, uses only existing dependencies, and the user explicitly says something like "I know this area, skip the deep research" or "keep it light", limit research to a single codebase analysis subagent only — skip web research, best practices, and library docs agents. Don't offer this proactively — only honor it when the user requests it. Note this in `.specsmiths/<n>.research.md` under a `## Research Scope` section as "Lightweight — codebase analysis only (user-requested)."

#### Grimoire Integration (preferred)

**Check if the grimoire plugin is installed** by looking for grimoire skills/agents (e.g., check if `grimoire/agents/` or the `grimoire:grimoire-research` skill exists).

**If grimoire is available**, use the `grimoire:grimoire-research` skill with action: **research**. Pass the discovery findings (from `.specsmiths/<n>.questions.md`) as the research context. Grimoire provides 6 specialized agents that map directly to the research streams below:

| Specsmith Stream | Grimoire Agent |
|------------------|----------------|
| Codebase Analysis | grimoire-codebase |
| Library Docs & Dependencies | grimoire-docs |
| Best Practices & Pitfalls | grimoire-practices |
| History & Existing Issues | grimoire-history |
| Database Schema Analysis | grimoire-schema |
| API Contract Review | grimoire-contracts |

Grimoire handles agent selection, parallel execution, and synthesis automatically. Its output goes directly to `.specsmiths/<n>.research.md`.

After grimoire completes, proceed to Phase 3.

#### Fallback: Inline Subagents

**If grimoire is NOT installed**, use the inline subagent definitions below.

**Always spawn:**

**Codebase Analysis** (subagent_type: explore, thoroughness: very thorough)
- Map architecture relevant to this spec
- Find existing patterns, conventions, utilities to reuse
- Identify every file that will need changes
- Find related tests, configs, infrastructure
- Check for similar past implementations as reference
- Look for code that contradicts or conflicts with what we want to build

**Spawn if relevant:**

**Best Practices & Pitfalls** (subagent_type: general-purpose) — spawn if unfamiliar domain, security-sensitive feature, or novel architecture
- WebSearch for best practices for this exact type of implementation
- Search for "mistakes to avoid" / "lessons learned" / "gotchas"
- Find security considerations specific to this feature type
- Look for performance benchmarks and recommendations
- Find relevant RFCs, specs, or industry standards
- Search for how well-known projects solve the same problem

**Library Docs & Dependencies** (subagent_type: general-purpose) — spawn if using new/unfamiliar dependencies or considering library alternatives
- WebFetch documentation for libraries being used or considered
- Check current dependency versions and compatibility
- Look for migration guides if upgrades are needed
- Read project docs (CLAUDE.md, README, docs/ folder)
- Check available MCP tools that could be relevant
- Compare alternative libraries/approaches if there's a choice to make

**History & Existing Issues** (subagent_type: explore) — spawn if legacy system, bug fix, or prior attempts exist
- Search git log for related changes and their commit messages
- Find TODOs, FIXMEs, HACKs in relevant areas
- Look for open issues or PRs related to this feature
- Read any existing specs, ADRs, or decision docs
- Check if previous attempts were made and why they were abandoned

**Database Schema Analysis** (subagent_type: explore) — spawn if data model changes are needed
- Analyze current schema and relationships
- Check for existing migrations and patterns
- Identify index requirements and query implications

**API Contract Review** (subagent_type: general-purpose) — spawn if integrating with external services
- Fetch and review external API documentation
- Check rate limits, auth requirements, error formats
- Identify versioning and deprecation concerns

Minimum 1 subagent (codebase), maximum 5. Each must write findings independently. Adjust based on discovery findings — don't spawn irrelevant agents.

Wait for all subagents. Read all findings.

Write research synthesis to `.specsmiths/<n>.research.md`:

```markdown
# Research: <n>

## Codebase Findings
- Architecture overview (relevant parts only)
- Patterns to follow (with file references)
- Files affected (with line-level detail where useful)
- Reusable code found
- Conflicts or technical debt in the area

## Best Practices & Pitfalls
- Recommended approaches (with source URLs)
- Specific mistakes to avoid (with explanations)
- Security considerations
- Performance considerations

## Dependencies & Compatibility
- Library versions and relevant docs
- Compatibility notes
- Alternative approaches considered and why rejected

## Historical Context
- Related git history (commits, PRs)
- Known issues and tech debt
- Prior decisions and rationale

## Key Decisions
For each decision point:
- **Decision:** <what needs deciding>
- **Options:** <what was found>
- **Recommendation:** <which option and why>
- **Trade-offs:** <what you give up>

## Research Gaps
- <Any areas where web research failed or was unavailable>
```

---

### Phase 3: Spec Creation

Using the discovery findings (brief + any interview Q&A) AND research findings, write the full specification to `.specsmiths/<n>.md`.

This is the actual deliverable. It must be detailed enough that someone could implement it without asking any follow-up questions.

```markdown
---
status: draft
created: <ISO date>
updated: <ISO date>
current_phase: 0
total_phases: <n>
research: <n>.research.md
discovery: <n>.questions.md
---

# Spec: <n>

## Overview
<What this does, why it's needed, the approach chosen and why>

## Requirements
### Functional
- <FR-1>: <requirement>
- <FR-2>: <requirement>
...

### Non-Functional
- <NFR-1>: <performance / security / scalability requirement>
...

### Constraints
- <C-1>: <boundary or limitation>
...

## Architecture & Design

### System Context
<How this fits into the broader system. What it touches.>

### Data Model
<New tables, schema changes, migrations. Include column types, constraints, indexes.>

### API Design
<Endpoints, request/response shapes, status codes, error formats. Be specific.>

### Flow
<Step-by-step flow of the main scenario. Include sequence or data flow if complex.>

## Edge Cases & Error Handling

| # | Scenario | Expected Behavior | Implementation Notes | Phase |
|---|----------|-------------------|---------------------|-------|
| E1 | <edge case 1> | <what should happen> | <how to implement> | <phase #> |
| E2 | <edge case 2> | ... | ... | ... |
| E3 | <error condition 1> | ... | ... | ... |
| E4 | <concurrent access> | ... | ... | ... |
| E5 | <partial failure> | ... | ... | ... |

## Implementation

Group steps into phases. Each phase is a shippable increment — it can be built,
tested, and committed independently. The user may stop after any phase and come
back later, or switch to a different spec entirely.

### Phase 1: <Phase title — what this delivers>

- [ ] 1.1. <Step title>
  **What:** <Detailed description>
  **Where:** <Files to create/modify with paths>
  **How:** <Specific approach — patterns, code structure, key functions>
  **Edge cases:** <E1, E3 — references to the table above>
  **Verify:** <Concrete verification — test command, curl, manual check>

- [ ] 1.2. <Step title>
  **What:** ...
  **Where:** ...
  **How:** ...
  **Edge cases:** ...
  **Verify:** ...

**Phase 1 acceptance:** <What must be true for this phase to be considered done — can be a test suite passing, a curl command returning expected output, etc.>

### Phase 2: <Phase title>

- [ ] 2.1. <Step title>
  ...

**Phase 2 acceptance:** ...

### Phase 3: <Phase title>
...

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation | Phase |
|------|-----------|--------|------------|-------|
| <risk> | <H/M/L> | <H/M/L> | <what to do> | <which phase> |

## Out of Scope
- <Explicitly excluded items and why>

## Testing Strategy

### Unit Tests
- <What to test, which files, key assertions>

### Integration Tests
- <What flows to test end-to-end>

### Manual Verification
- <Step-by-step manual test script>

## Rollback Plan
<How to undo this if it goes wrong. Per-phase if phases are deployed independently.>
```

**Quality bar for phases:** Each phase must be a meaningful, shippable increment — not just arbitrary grouping. Think: "if the user stops here and comes back in two weeks, does this phase stand on its own?" Common phasing patterns: (1) data model → API → business logic → error handling → polish, (2) core happy path → edge cases → performance → monitoring, (3) read path → write path → admin/management.

**Quality bar for steps:** Each step must be implementable without further questions. Include file paths, function signatures, patterns to follow from codebase research, specific edge case references from the table, and a concrete verification method. A step like "implement the service" is NEVER acceptable — break it down until each step is a focused, verifiable unit of work.

After writing the spec, present a summary to the user and ask:
1. Does anything look wrong or missing?
2. Should we activate this spec now?

Do NOT set it as active until the user confirms.

---

## Action: list

Display all available specs with interactive picker using AskUserQuestion.

1. Read `.specsmiths/active.json` and glob `.specsmiths/*.md` (exclude `.research.md` and `.questions.md`)
2. For each spec, extract frontmatter to get status, current_phase, total_phases
3. Use AskUserQuestion to let user pick which spec to work on:

```
Which spec would you like to work with?

Options:
- alpha (Phase 2/3, implementing) ← active
- beta (Phase 1/2, paused)
- gamma (draft)
```

4. After selection, offer follow-up actions:

```
What would you like to do with "<selected-spec>"?

Options:
- Show full spec
- Edit spec
- Switch to this spec (make it active)
- Archive spec
```

Show archived specs in a separate "Archived" section at the bottom (collapsed by default).

---

## Action: switch

Switch to a different spec.

1. Save current progress on the active spec — check off completed steps, update `current_phase`
2. Update `.specsmiths/active.json` to point to the new spec
3. Read the new spec and display which phase and step it left off at
4. Update TodoWrite task list with remaining steps from the current phase
5. Do NOT start implementing — show status and wait

---

## Action: resume

Resume the active spec.

1. Read `.specsmiths/active.json` to find the active spec
2. Read the spec file, its `.research.md`, and `.questions.md` for full context
3. Find the current phase and the first unchecked step (`- [ ]`) within it
4. Update TodoWrite task list with remaining steps in the current phase
5. Show which phase and step you're resuming from with full details, and wait for go-ahead

---

## Action: status

Show dashboard of all specs with phase-level detail:

```
Spec Status

→ alpha      Phase 2/3  ██████░░░░  implementing
              Phase 1: Setup & data model ✓
              Phase 2: API endpoints (step 2.3/2.5) ← here
              Phase 3: Error handling & polish

  beta       Phase 1/2  ████░░░░░░  paused
              Phase 1: Core import logic (step 1.2/1.4)
              Phase 2: Validation & edge cases

  gamma      Phase 0/3  ░░░░░░░░░░  draft

Archived:
  old-auth   Phase 3/3  ██████████  archived (2025-01-15)
```

---

## Action: archive

Move a completed or shelved spec to archived status without deleting:

1. Set `status: archived` and `archived_date: <ISO date>` in the spec's frontmatter
2. If archiving the active spec, set `"active": null` in `.specsmiths/active.json`
3. `/specsmith-list` and `/specsmith-status` show archived specs in a separate section (collapsed by default)
4. Archived specs can still be viewed with `/specsmith-show <n>`

---

## Action: unarchive

Restore an archived spec to its previous status:

1. Remove `archived_date` from frontmatter
2. Set `status` back to the appropriate value based on progress:
   - `draft` if `current_phase: 0`
   - `implementing` if partially complete
   - `complete` if all phases done
3. Does NOT automatically make it the active spec — use `/specsmith-switch <n>` for that

---

## Action: drop

Remove a spec and all its companion files (.research.md, .questions.md). Confirm before deleting. If dropping the active spec, clear `.specsmiths/active.json`.

---

## Action: edit

Revise an existing spec. Specs are living documents — they evolve as you learn more during implementation, when requirements change, or when you realize something was missed.

1. Read the spec file, its `.research.md`, and `.questions.md` for full context
2. Ask the user what they want to change. Common edit scenarios:

**Adding/changing requirements mid-flight:**
- Add the new requirements to the Requirements section
- Trace the impact: which phases/steps/edge cases are affected?
- Add, modify, or reorder steps as needed
- If a new phase is needed, insert it in the right position and renumber subsequent phases
- Update the edge case table if new scenarios are introduced
- Append new Q&A to `.questions.md` under a new `## Revision: <date>` section

**Reworking a phase that isn't right:**
- Discuss what's wrong — wrong approach? Missing steps? Too granular? Too vague?
- Rewrite the phase's steps while preserving any already-completed steps (`- [x]`)
- Update the phase acceptance criteria
- If already-completed steps are now invalid, mark them with `⚠️ REWORK` and uncheck them

**Splitting or merging phases:**
- If a phase is too big, split it — create new phase headers, redistribute steps, add acceptance criteria for each
- If phases are too granular, merge them — combine steps under one header, write a single acceptance
- Renumber all step references (e.g. `2.1` becomes `3.1` if a phase was inserted before it)

**Adding edge cases discovered during implementation:**
- Add rows to the edge case table with the right phase assignment
- Add handling to the relevant steps (existing or new)
- If the edge case affects an already-completed step, add a new fix step in the current phase

**Updating research after learning something new:**
- Append to `.research.md` under a `## Updated: <date>` section
- Don't overwrite original research — it's the historical record
- Reference new findings in the relevant spec steps

3. After edits, show a diff summary: what was added, changed, removed, and which phases are affected
4. Update `updated` timestamp in frontmatter

**Rules for editing:**
- Never silently change completed steps — if a done step needs rework, flag it explicitly
- Preserve the discovery and research history — append, don't overwrite
- If edits are so large they basically rewrite the spec, suggest creating a new spec instead and archiving the old one
- Always show the user what changed before finishing

---

## Action: show

Display a spec without activating it. Read `.specsmiths/<n>.md` and show the full contents. Useful for reviewing a spec before deciding to switch to it or edit it. If `<n>` is omitted, show the active spec.

---

## Action: implement

Start implementation of the active spec. This action bridges planning and execution.

1. Read `.specsmiths/active.json` to find the active spec
2. If no active spec, error: "No active spec. Use `/specsmith-switch <name>` to activate one first."
3. Read the spec file, its `.research.md`, and `.questions.md` for full context
4. Find the current phase and first unchecked step
5. **Read runebook context** (if runebook is installed):
   - Check if `.runebook/` exists at the project root
   - If yes:
     - From the spec, collect file paths mentioned in the **Where:** fields
       of steps in the current phase
     - Grep `.runebook/` entry frontmatter for `source_files` matching those paths
     - Read each matching entry to understand current documented behavior,
       dependencies, edge cases, and recent changelog entries
     - Check if a guide covers this domain (grep `guides/*.md` `covers` field)
     - Hold this context for use during implementation — do not dump to user
     - Note components with no entry for creation after steps complete
   - If `.runebook/` does NOT exist:
     - Skip silently
6. Create TodoWrite tasks for all remaining steps in the current phase
7. Begin implementing the first unchecked step

---

### CRITICAL: Step Completion Protocol

**After completing EVERY step, you MUST:**

1. **Edit the spec file** to mark the step as done:
   - Change `- [ ] X.Y. Step title` to `- [x] X.Y. Step title`

2. **Update frontmatter** in the spec file:
   - `updated: <current ISO date>`
   - If all steps in current phase are done: increment `current_phase`
   - If all phases are done: set `status: complete`

3. **Announce completion** to the user:
   - "✓ Step X.Y complete. Moving to X.Z..."
   - Or at phase boundary: "✓ Phase X complete. [acceptance criteria passed]"

4. **UISpec auto-sync** (if the completed step created or modified an API endpoint):
   - Check if the step's **Where:** references route/controller/handler files,
     or the step's **What:** mentions creating/modifying an API endpoint
   - If yes AND `.specsmiths/uispec.json` exists (frontend is configured):
     - Auto-run the sync logic from `specsmith:specsmith-uispec` skill action `sync`
     - Append to the step completion announcement:
       "📋 Synced API changes to .uispec/ in <frontend-project>"
   - If yes but `.specsmiths/uispec.json` does NOT exist:
     - Skip silently — this is a backend-only project
   - If the step didn't touch API endpoints, skip silently

5. **Runebook update** (if affected files are tracked by runebook):
   - Check if `.runebook/` exists at the project root
   - If yes:
     - Identify which files were created or modified in this step
       (from the step's **Where:** field)
     - Grep `.runebook/` entry frontmatter for `source_files` matching those paths
     - For each matching entry: re-read the source files, update the entry body
       (behavior, dependencies, request/response shapes — whatever changed),
       preserve human-written context, append a changelog line:
       `- <date> Updated during specsmith step X.Y (<spec-name>)`
     - If a new component was created with no existing entry, create one
       using the appropriate runebook template
     - If any updated entries are covered by a guide (check `covers` frontmatter
       in `guides/*.md`), update the guide's auto-derivable sections
     - Append to the step completion announcement:
       "Updated runebook: `<type>/<name>`, `<type>/<name>`"
   - If `.runebook/` does NOT exist:
     - Skip silently — runebook is not installed

**DO NOT proceed to the next step until the current step is marked done in the file.**

---

**Within a phase:**
1. Read the step details, edge case references, and **Verify** criteria
2. Implement following the **How** guidance and patterns from research
3. Handle the referenced edge cases
4. Run the **Verify** check — fix issues before marking done
5. **Follow the Step Completion Protocol above** — mark step done, update frontmatter, announce

If interrupted or asked to switch, complete the current step first or explicitly note where you stopped.

**At phase boundary:**
1. When all steps in a phase are checked off, run the **Phase acceptance** check
2. If acceptance passes, announce phase completion and ask:
   - Continue to next phase?
   - Switch to a different spec?
   - Stop here for now?
3. If any steps in this phase created or modified API endpoints:
   - If `.specsmiths/uispec.json` exists: API changes were already auto-synced per-step. Include in summary:
     "📋 API changes from this phase have been synced to .uispec/."
   - If `.specsmiths/uispec.json` does NOT exist: skip — backend-only project.
4. Update `current_phase` in frontmatter
5. If this was the last phase, set `status: complete`

**Phase completion checklist:**
- [ ] All steps in phase show `- [x]`
- [ ] Phase acceptance criteria verified
- [ ] `current_phase` incremented in frontmatter
- [ ] `updated` timestamp refreshed
- [ ] User notified of phase completion

The user can stop after any phase. They can `/specsmith-switch` to work on something else and come back later. The phase structure ensures there's always a clean stopping point.

---

## State File: `.specsmiths/active.json`

```json
{
  "active": "alpha",
  "last_switched": "<ISO date>"
}
```

If no spec is active, `"active"` is `null`. The current phase and step are tracked in each spec's own frontmatter, not here — this file only tracks which spec is active.

## Critical Behaviors

- **Discovery is mandatory** — never skip the interview. Ask until you truly understand the problem.
- **Research is expected** — always run at least a codebase analysis subagent. Only skip web research subagents if the feature is narrowly scoped AND the user explicitly requests lightweight research.
- **Never auto-implement** — always wait for explicit go-ahead (use `/specsmith-implement` to start)
- **Save before switching** — always persist progress before changing context
- **Re-read everything on resume** — spec + research + questions, restore full context
- **Specs are the source of truth** — never rely on conversation memory
- **Specs are living documents** — when the user says "actually let's change X" or "I realized we also need Y" during implementation, treat it as an implicit `/specsmith-edit`. Update the spec file, don't just implement the change ad-hoc. The spec must always reflect reality.
- **Commit `.specsmiths/` to git** — specs must survive across sessions
- **No vague steps** — every step must be implementable without follow-up questions
- **Every step has verification** — nothing is done until verified
- **Edge cases are first-class** — they go in the spec table AND get assigned to specific steps
- **Completed work is sacred** — never silently modify checked-off steps. If they need rework, flag them explicitly and get confirmation.
