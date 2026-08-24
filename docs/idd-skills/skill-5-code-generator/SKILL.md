---
name: idd-code-generator
description: Generates production-ready React/TypeScript components for any Couchbase Capella UI feature using Claude Code. Reads real source files to extract actual DS tokens, component APIs, and patterns. Capella RBAC rules, destructive guards, and code generation rules are fixed. Feature-specific components and file paths are derived from the intent statement at runtime.
product: Couchbase Capella
chain: consumes skill-1-intent-builder output + skill-4-ui-component as reference · outputs to git branch
runtime: Claude Code (terminal) OR Claude Project chat — see two workflows below
version: 1.5 — added a §Handoff section (this skill had none): Gate 3 is engineering-owned, and every correction found by running the code must be written back into the intent, tagged, before merge — the leg whose absence caused a fixed bug to be reintroduced on regeneration
---

# IDD Skill 5 — Code Generator

## What this skill does

Generates typed React/TypeScript components for any Capella UI feature
described in an intent statement, grounded in the real codebase.

The engineer reviews everything before any file is committed.

**Two paths — same output quality, different setup:**

| | Path A — Claude Project chat | Path B — Claude Code terminal |
|---|---|---|
| API key needed | ❌ No — uses claude.ai subscription | ✅ Yes — console.anthropic.com |
| File navigation | Engineer pastes source files | Claude Code reads repo directly |
| File writing | Engineer pastes output into repo | Claude Code writes to disk |
| Setup time | ~5 min | ~15 min (install + key) |
| Best for | Getting started · demos · no tooling | Regular engineering workflow |

Start with Path A. Move to Path B when the team is set up with Claude Code.

---

## What is fixed (Capella product context)

**RBAC enforcement — always:**
```tsx
// CORRECT — absent for restricted roles
const canManage = role === 'org-owner' || role === 'project-owner';
{canManage && <Button>Create</Button>}

// WRONG — never use disabled for access control in Capella
<Button disabled={!canManage}>Create</Button>
```

**Destructive guard — always:**
Any delete/rotate/restore → typed confirmation modal.
Confirm button disabled until input matches entity name exactly.
Never closes on backdrop click.
Always shows impact summary.

**Token rule — always:**
All colors via CSS variables from the real tokens file.
Never hardcode hex values.
Never use `any` types.

**Session start checklist — always:**
Before writing any code, read and report on all grounding files.
Never skip the checklist.

---

## What is generic (derived from intent statement at runtime)

- Which components to generate (from §Scope of the intent statement)
- The file paths (from the existing codebase structure + feature name)
- The columns, fields, and hierarchy (from §Scope)
- The scale requirements (from §Constraints)
- The immutable fields and their treatment (from §Constraints)
- The dangerous defaults and their treatment (from §Constraints)
- The RBAC experience per role (from §Who/RBAC)
- The routes (from §Navigation model)

---

## Inputs

| Input | Path A (Project chat) | Path B (Claude Code) |
|---|---|---|
| `intent-[feature]-v1.0.md` | Already in Project Knowledge | In `docs/intents/` in the repo |
| Source files | Engineer pastes into chat | Claude Code reads from repo |
| `CLAUDE.md` | Not needed | At repo root |
| UI component (Skill 4) | Reference — paste key sections | Reference — in the repo |

---

## Path A — Claude Project chat (no API key required)

### When to use Path A
- Getting started with IDD
- Demo or walkthrough sessions
- No Claude Code installed yet
- Engineer wants to review output before setting up tooling

### Step 1 — Gather the source files

The engineer finds and copies the content of these files from the codebase.
Paste them into the Project chat in Step 2.

```
Files to gather (find the closest match in your repo):

1. The existing page component for the screen being extended
   e.g. src/pages/Databases/Buckets/index.tsx

2. The Button component
   e.g. src/components/Button/Button.tsx

3. The Modal component (or note if it doesn't exist)
   e.g. src/components/Modal/Modal.tsx

4. The auth / role context
   e.g. src/contexts/AuthContext.tsx or src/hooks/useRole.ts

5. The design tokens / CSS variables file
   e.g. src/styles/tokens.css or src/styles/variables.css

6. The relevant type definitions
   e.g. src/types/bucket.ts or src/types/index.ts

7. The API service for this domain
   e.g. src/api/buckets.ts or src/services/buckets.ts
```

### Step 2 — Open the Claude Project and paste this prompt

```
Run IDD Skill 5: Code Generator (Path A — Claude Project).

The intent statement for this feature is already in Project Knowledge.

I am pasting the relevant source files below so you can ground the
generated code in the real codebase. Read them before generating anything.

--- SOURCE FILE 1: [filename and path] ---
[PASTE FILE CONTENTS]

--- SOURCE FILE 2: [filename and path] ---
[PASTE FILE CONTENTS]

--- SOURCE FILE 3: [filename and path] ---
[PASTE FILE CONTENTS]

[continue for all files gathered in Step 1]

---

Now complete the session start checklist:
For each pasted file, report:
- What you found: key exports, type names, CSS variable names, role string values
- Any gap: something the intent statement needs that isn't in the pasted files

Do not generate any code until the checklist is complete and I confirm.
```

### Step 3 — Token extraction

```
From the design tokens file I pasted:
Extract all CSS variables needed for this feature:
- Text colors (primary / secondary / tertiary / disabled / info / danger / warning)
- Background colors (primary / secondary / info / danger / warning)
- Border colors and border-radius values
- Status/indicator colors
- Button variants and their exact className or prop syntax
- Spacing values from the existing screen I pasted

Output as a reference table: variable name | value | used for
```

### Step 4 — Generate components

Use the same component prompts as Path B (see §Step 5 below).
The only difference: reference "the [component] I pasted" instead of
"the [component] you found in the repo."

### Step 5 — Gap report

Same gap report prompt as Path B (see §Step 6 below).

### Step 6 — Save the output

The engineer copies each generated component from the chat and creates
the files manually in the repo at the paths Claude suggested.

```bash
# Create directories
mkdir -p src/[path suggested by Claude]

# Create each file
# Paste the content Claude generated
```

### Step 7 — Branch and commit

Same as Path B (see §Step 8 below).

---

## CLAUDE.md — generic version (Path B only)

Place this file at the repo root. It is Claude Code's instruction set.
It is Capella-specific but feature-generic — works for any Capella UI feature.

```markdown
# CLAUDE.md — Capella Frontend · Intent-Driven Development

## 1 — What you are doing

You are implementing a Capella UI feature defined in an intent statement.
Find the intent statement at: docs/intents/

Read the intent statement before touching any code.
The intent statement tells you what to build, what the RBAC rules are,
what the scale constraints are, and where the feature boundary is.

## 2 — Session start checklist (mandatory — complete before writing any code)

- [ ] Find and read the intent statement in docs/intents/
- [ ] Find the existing page/component for the screen being extended
      (look in the routes file and the pages directory)
- [ ] Find and read the real Button component — note the prop types and variants
- [ ] Find and read the real Modal component — note the prop types (or flag as missing)
- [ ] Find and read the auth/role context — note the exact role string values
- [ ] Find and read the design tokens file — note the CSS variable names
- [ ] Find and read the router file — confirm which routes exist
- [ ] Find the existing API service for this domain — note the endpoint patterns
- [ ] Find the existing type definitions — note the interface names

Report what you found for each item before generating any code.
If something is missing, flag it as a gap — do not invent it.

## 3 — What to build

Read §Navigation model and §Scope from the intent statement.
Generate the components described there.
Match the naming convention of existing components in the codebase.

## 4 — RBAC rules (non-negotiable for Capella)

Read §Who/RBAC from the intent statement for the experience per role.

Four roles: read the exact string values from the auth context you found.
Excluded controls: ABSENT (conditional render) — not DISABLED.
- CORRECT: {canManage && <Button>Action</Button>}
- WRONG:   <Button disabled={!canManage}>Action</Button>

## 5 — Design system rules

Use the CSS variables you found in the tokens file. Never hardcode hex.
Match the exact className / styled-component / Tailwind pattern used in existing components.
Match the density (spacing, padding) of the existing screen being extended.

## 6 — Interaction rules (always apply to any Capella feature)

Destructive actions (delete, rotate, restore):
- Always require a typed confirmation modal
- Confirm button disabled until input matches entity name exactly
- Never closes on backdrop click
- Always shows downstream impact summary

Async states (always include all four):
- Loading: skeleton rows/cards
- Success: toast notification
- Error: inline error with retry
- Empty: empty state with appropriate CTA

## 7 — Output format (required on every generated file)

Header comment:
/**
 * [ComponentName]
 * Intent: [intent filename] §[section]
 * Route:  [route from intent §Navigation model]
 * RBAC:   [roles that see this component]
 */

After all components: state transitions table
| User action | Result | Condition |

After state transitions: gap report
| Item | Status | Action needed |

## 8 — What NOT to do

- Do not hardcode hex colors
- Do not use `any` types
- Do not use disabled for access control
- Do not modify existing files (generate new files only)
- Do not invent API endpoints (flag as gaps)
- Do not commit, push, or create branches unless explicitly asked
- Do not add npm packages without flagging first
```

---

## Path B — Claude Code terminal (requires API key)

### When to use Path B
- Regular engineering workflow once Claude Code is set up
- Larger features with many components
- When you want Claude Code to write files to disk directly
- When you want automated branch creation

### Step 1 — Place files in the repo

```bash
# Intent statement goes in the repo
mkdir -p docs/intents
cp /path/to/intent-[feature]-v1.0.md docs/intents/

# CLAUDE.md goes at repo root
cp /path/to/CLAUDE.md .
```

### Step 2 — Start Claude Code

```bash
cd /path/to/capella-repo
claude
```

### Step 3 — Session start prompt (always first)

```
Read CLAUDE.md at the repo root.
Then complete the session start checklist in §2 of CLAUDE.md.

For each checklist item:
- Find the actual file
- Read it
- Report: the exact path, the key exports/types/variable names, any gaps

Do not write any code until the checklist is complete and I confirm.
```

Review the report. Confirm before proceeding.

### Step 4 — Token extraction

```
From the design tokens file you found:
Extract all CSS variables needed for this feature:
- Text colors (primary / secondary / tertiary / disabled / info / danger / warning)
- Background colors (primary / secondary / info / danger / warning)
- Border colors and border-radius values
- Status/indicator colors
- Button variants and their exact className or prop syntax
- Spacing values from the existing screen being extended

Output as a reference table: variable name | value | used for
```

### Step 5 — Generate components

Read §Scope from the intent statement to know what to generate.
Generate one component at a time. Review each before the next.

**For each component, use this prompt pattern:**

```
Generate [ComponentName] for the [feature name] feature.

Read:
- §Scope in docs/intents/intent-[feature]-v1.0.md for what this component does
- §Who/RBAC for the RBAC rules
- §Constraints for scale, immutable fields, destructive actions, dangerous defaults
- The real [existing component] you found for the prop types and patterns to match

Requirements:
- Use the real [Button/Modal/etc] component API you found (exact prop names)
- Use CSS variable names from the tokens file you found
- Implement RBAC: [canManage check] — absent not disabled
- [Any feature-specific requirement from §Constraints]

Follow the header comment format from CLAUDE.md §7.
```

### Step 6 — Gap report

```
Generate the gap report for this session.

For each item, status is: exists / missing / needs confirmation

1. API endpoints used in the hook/components
   (list each one, confirm it exists in the API service file you found)
2. Routes in the intent statement — do they exist in the router?
3. Design tokens referenced — do they all exist in the tokens file?
4. Component types that need adding to existing type definitions
5. npm packages that would need to be added (list with justification)

Format: table with Item | Status | File/Location | Action needed
```

### Step 7 — Write files (after engineer review)

```
Write all generated files to disk.

Use paths that match the existing codebase structure you found.
Do not modify any existing files.
Report each file: full path + line count.
```

### Step 8 — Create branch and commit (after engineer approval)

```bash
# Feature branch — name from intent statement feature slug
git checkout -b feature/idd-[feature-slug]

# Stage new files only
git add [paths reported by Claude Code]

# Commit with intent traceability
git commit -m "feat([scope]): [feature name] (IDD)

Intent: docs/intents/intent-[feature]-v1.0.md

- [Component 1]: [what it does]
- [Component 2]: [what it does]
- [Hook]: [what it manages]

RBAC: absent not disabled for restricted roles
[Scale note if applicable]
[Any other constraint from §Constraints]"

git push origin feature/idd-[feature-slug]
```

---

## Engineer review checklist

Before approving any generated file:

- [ ] All imports reference real paths that exist in the codebase
- [ ] All CSS tokens match real variable names from the tokens file
- [ ] Role values match the real values from the auth context
- [ ] No `any` types anywhere
- [ ] No `disabled` props used for access control
- [ ] API endpoints either exist or are in the gap report
- [ ] Destructive modal: typed input, no backdrop close, impact summary shown
- [ ] Dangerous defaults: correct default state per §Constraints
- [ ] Skeleton, empty, error, and success states all present
- [ ] For multi-role features: the role scenario reference doc exists and every row is traceable to the actual code, not restated from the intent's draft Who/RBAC

---

## Time estimate

| Phase | Path A (Project chat) | Path B (Claude Code) |
|---|---|---|
| Setup | 5 min (gather files) | 15 min (install + key) |
| Session start + checklist | 10 min | 15 min |
| Token extraction | 10 min | 10 min |
| Components (varies by feature) | 30–60 min | 30–60 min |
| Gap report | 5 min | 5 min |
| Engineer review | 60–90 min | 60–90 min |
| Write / commit | 20 min (manual paste) | 15 min (Claude Code writes) |
| **Total** | **~2.5–3.5 hours** | **~2.5–3.5 hours** |

Both paths produce the same output quality.
Path B saves ~5 min on file writing. Path A saves ~10 min on setup.

vs 3–4 days building from scratch.

---

## Reference example

`examples/bucket-scope-collection.md` — the complete Claude Code session
for the Bucket/Scope/Collection feature. Shows the checklist report,
token extraction output, component generation prompts, and gap report.

---

## Slash commands (Path B — Claude Code)

Once CLAUDE.md is in the repo root, these commands are available
in every Claude Code session. No copy-pasting prompts.

```
/idd-intent "rough feature seed"   → Skill 1 — intent statement
/idd-brief intent-[feature].md     → Skill 2 — design brief
/idd-prototype intent-[feature].md → Skill 3 — throwaway HTML prototype
/idd-ui-component intent-[feature].md [screen] → Skill 4 — real component
/idd-code intent-[feature].md      → Skill 5 — components + tasks.md
/idd-archive [feature-slug]        → archive shipped feature
/idd-status                        → show all active features
```

---

## tasks.md — generated after every Skill 5 run

After generating all components, Claude Code fills in
`docs/idd-skills/skill-5-code-generator/tasks-template.md`
and saves the result as `tasks-[feature-slug].md` in the
feature component directory.

The tasks.md covers:
- Hook / state management tasks
- Component implementation tasks
- RBAC wiring verification
- API integration tasks
- Routing confirmation tasks
- Gap resolution tasks (from gap report)
- Design alignment tasks
- Quality gate checklist
- Archive checklist (run when feature ships)

Engineer checks off tasks as they complete integration.
PM can see progress without asking.

---

## Role scenario reference — generated after tasks.md, for any multi-role feature

If the feature has more than one RBAC-distinct experience (per §Who/RBAC in
the intent statement — most features that aren't purely single-role), also
generate a **verified role scenario reference**: a role × screen matrix
checked against the code just written, not the intent's speculative
Who/RBAC section (which was drafted before any code existed and may have
drifted during implementation).

This is different from tasks.md's "RBAC wiring" checkbox — that just
confirms the work was done; this document records *what the actual
conditional logic does*, so a PM, QA engineer, or support person can answer
"if role X is enabled, what does the screen show" without re-reading every
component.

**Prompt:**

```
Generate the role scenario reference for this feature.

For each screen/surface generated in this session, and for each
RBAC-distinct role/permission-state combination named in §Who/RBAC:
- Read the actual conditional logic just written (not the intent's
  draft — the real code)
- State exactly what renders, is absent, or is disabled in that state
- Note any role-combination case (e.g. two roles held together) and
  what the combined state does, per any combination matrix in §Who/RBAC

Format as one table per screen/surface: State | Result.
Cross-check every row against the file/line it came from before
including it — do not restate the intent's Who/RBAC section verbatim.

Save as docs/[feature-slug]-role-scenarios.md.
```

Save alongside `tasks-[feature-slug].md`, in the same location. Skip this
output for single-role or no-RBAC-branching features — it only earns its
keep when there's more than one state to disambiguate.

---

## Handoff

**This is Gate 3 — Engineering owns it, not PM.** Someone has to run the app.
"Compiles" is not "verified", and neither is "the Storybook story looked right".

**Goes to:**

1. **Engineer** — runs the app and clicks through the *real* feature, not the
   story. Works the gap report. Checks that the generated code landed where
   §Navigation model said it would — in the reference run it compiled, worked,
   and was still on the wrong page. Decides whether this merges.
2. **PM** — reads the role scenario reference and the state transitions table;
   confirms each role's real behaviour matches §Who/RBAC. Decides whether any
   gap is an accepted follow-up or a blocker.
3. **Designer** — compares the running feature against the Storybook story
   signed off in Skill 4; flags anything the container wiring changed.

**Done when:** every gap-report item is resolved or is an accepted,
ticketed follow-up with an owner; the tasks.md quality gate is checked off; and
at least one person has run the feature and exercised each RBAC role.

**Back into the intent — this is the leg that has actually failed before.**
Every correction found by running the code goes into the intent statement
before the branch merges, tagged `[CRITICAL]` or `[CORRECTED]`, with the
version bumped. A fix that lives only in a code comment does not survive the
next `/idd-code` run: regenerating bucket-scope-collection reintroduced a bug
the first pass had already found and fixed (using the bucket's Capella ID
instead of its cluster-native name for proxy-service calls), because that fix
had never been written back here. If you are about to fix something in the
generated code, ask whether the *intent* was wrong — and if it was, fix it in
both places.

---

## Archive pattern — after feature ships

```
/idd-archive [feature-slug]
```

Moves:
```
docs/intents/active/intent-[feature]-v[n].md
→ docs/intents/archive/[YYYY-MM-DD]-[feature-slug]/
```

Also moves tasks.md into the archive folder.
Prompts to update capella-core-url-map.md with any new routes.
Prints unresolved open questions so nothing gets lost.

PR description must include:
```
Intent: docs/intents/archive/[date]-[slug]/intent-[slug]-v[n].md
```

Every PR is traceable to its intent statement. Forever.
