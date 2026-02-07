---
name: chisel-ops
description: "Core dispatch logic for chisel operations. Parses operation type, spawns the chisel agent, returns change summary. Actions: execute (perform edit), help (show operations). Triggers on: 'rename X to Y', 'move X to Y', 'replace X with Y', 'change the color', 'update the font', 'fix all references', or when invoked via /chisel command."
user-invocable: false
---

# Chisel Operations

This skill dispatches to the chisel agent for fast, surgical code edits.

## Action: help

Display available operations when `/chisel` is invoked with no arguments:

```
Chisel — Fast Surgical Code Editor

  rename    Rename files, variables, functions, classes, or components
            Updates all references and imports automatically
            Example: /chisel rename UserCard.tsx to ProfileCard.tsx

  move      Move files or directories to new locations
            Updates all import paths and config references
            Example: /chisel move utils/helpers.js to lib/helpers.js

  replace   Find and replace text across the codebase
            Respects word boundaries, warns on large scope
            Example: /chisel replace API_V1 with API_V2

  edit      Make small targeted code changes
            CSS tweaks, config updates, HTML adjustments
            Example: /chisel edit change primary button color to green

  Or just describe what you need:
            /chisel rename all snake_case functions to camelCase in src/
            /chisel update the logo path across all templates
```

---

## Action: execute

Perform an edit operation by spawning the chisel agent.

**Input:** operation type (rename/move/replace/edit/auto-detect) + context

### Step 1: Read the Agent

Read the chisel agent file from `${CLAUDE_PLUGIN_ROOT}/agents/chisel.md`.

### Step 2: Spawn the Agent

Spawn via the Task tool:

```
Task tool:
  subagent_type: general-purpose
  model: haiku
  description: "chisel <operation>"
  prompt: "<contents of ${CLAUDE_PLUGIN_ROOT}/agents/chisel.md>

  ## Task
  <the user's full request with operation type and context>"
```

### Step 3: Return Results

The agent returns a summary of all files modified and what changed in each. Present this to the user.

If the agent asked the user a clarifying question (ambiguous match, large scope preview), relay the question and wait for the user's response before continuing.

### Step 4: Post-Operation Hooks

After the chisel agent returns results, check for cross-plugin updates:

**Runebook update** (if runebook is installed):
- Check if `.runebook/` exists at the project root
- If yes:
  - From the agent's results, identify all files that were modified, renamed, or moved
  - Grep `.runebook/` entry frontmatter for `source_files` matching the OLD paths
    of any renamed or moved files
  - For each matching entry:
    - **Rename/move:** Update `source_files` paths in frontmatter to new locations
    - **Replace:** Re-read affected source files and update entry body if behavior
      descriptions reference the replaced text (function names, class names, endpoint paths)
    - **Edit:** Re-read affected source files and update entry if documented behavior changed
    - Append a changelog line: `- <date> Updated by chisel <operation>: <summary>`
  - Update `.runebook/index.md` if any frontmatter metadata changed
  - If any updated entries are covered by a guide, update the guide's
    `Where Things Live` section with new file paths
  - Announce: "Updated runebook: `<type>/<name>` (source paths updated)"
- If `.runebook/` does NOT exist:
  - Skip silently

**UISpec update** (if UISpec is configured):
- Check if `.uispec/` exists (frontend project) OR `.specsmiths/uispec.json` exists (backend project)
- If yes AND the operation renamed/moved API handler or route files:
  - For **backend projects**: read uispec.json, find frontend path, check if
    `.uispec/endpoints/` contains a spec referencing the old endpoint path,
    update the spec's API section (Method, Path)
  - For **frontend projects**: check if `.uispec/endpoints/*.md` files reference
    the old file paths in UI Guidelines and update those references
  - Announce: "Updated .uispec/ endpoint spec: `<endpoint-name>`"
- If neither exists:
  - Skip silently

---

## Operation-Specific Guidance

When passing context to the agent, include these hints based on operation type:

### rename
- Include: old name, new name, scope (single file vs project-wide)
- Agent will: find all references, update imports, rename co-located files (tests, styles, stories), use `git mv`

### move
- Include: source path, destination path
- Agent will: move file/directory, update all import paths, update config files, use `git mv`

### replace
- Include: old text, new text, scope (specific files/dirs or project-wide)
- Agent will: analyze word boundaries, count matches, preview if >5 files, execute replacement

### edit
- Include: what to change, where (file/component/section)
- Agent will: make the minimal targeted change, preserve formatting, verify no breakage

### auto-detect
- Pass the full natural language request
- Agent will: determine operation type from context and proceed accordingly
