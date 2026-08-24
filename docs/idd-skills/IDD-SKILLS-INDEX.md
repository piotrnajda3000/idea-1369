# IDD Skills — Throughline Pipeline
**Intent-Driven Development · Couchbase Capella**
**Version:** 2.4 · Five skills · Capella fixed · Feature generic

> v2.1 restores a fast, throwaway prototype stage (Skill 3) ahead of the
> real-component UI mock (now Skill 4, was Skill 3 in v1.x–v2.0) and code
> generation (now Skill 5, was Skill 4). v2.2 adds a pacing rule to Skill 1
> (Intent Builder): batch and draft-with-source for code-groundable
> dimensions, one-at-a-time only where no default exists. v2.3 adds a
> verified role scenario reference as a standard Skill 5 output, for any
> feature with more than one RBAC-distinct experience. v2.4 renames Skill 4
> from "UI Mock" to "UI Component" (`/idd-mock` → `/idd-ui-component`) — it
> has generated real, shipping code since v2.0, "mock" undersold what it
> does. See "Version
> history" below for why.

---

## Design principle

**Capella product context is fixed across all features.**
The 4 RBAC roles, DS tokens, URL map, production chrome, 11 generation
rules, and destructive guard pattern never change.

**Feature-specific content is elicited or read at runtime.**
Routes, columns, interaction patterns, scale limits, immutable fields,
and dangerous defaults come from the intent statement — which is produced
fresh for each feature by Skill 1.

A new PM can run these skills for any Capella UI feature without
modifying the skill files. The only input that changes is the feature seed.

---

## The pipeline

```
PM types a rough seed for any Capella feature
              ↓
┌─────────────────────────────────────────────────────────┐
│ SKILL 1: Intent Builder                                 │
│ Fixed:   4 RBAC roles, RBAC rule, DS, chrome, guards   │
│ Generic: feature name, routes, columns, scale, fields   │
│ Output:  intent-[feature]-v1.0.md                      │
│ Time:    15–20 min                                      │
└───────────────────┬─────────────────────────────────────┘
                    │ intent statement → Project Knowledge
         ┌──────────┴──────────┐
         ↓                     ↓
┌────────────────┐   ┌─────────────────────────────────────┐
│ SKILL 2:       │   │ SKILL 3: Prototype                  │
│ Design Brief   │   │ Fixed:   chrome (approximated), DS  │
│                │   │          tokens (hand-copied), RBAC │
│ Fixed: RBAC    │   │ Generic: screens, pattern, columns, │
│ model, chrome  │   │          scale — from intent        │
│ Generic: all   │   │ Output:  mock-[feature]-vN.html     │
│ content from   │   │ Time:    8–12 min per screen        │
│ intent         │   │ Throwaway — not real code            │
│ Time: ~80 min  │   └─────────────────────────────────────┘
└────────────────┘             │
         │                     │ PM/Designer/UX iterate fast — cheap to
         │                     │ redo entirely; shape gets decided here
         └──────────┬──────────┘
                    │ shape settled · intent updated if it changed
                    ↓
┌─────────────────────────────────────────────────────────┐
│ SKILL 4: UI Component                                    │
│ Fixed:   DS tokens/components (real, via                │
│          @couchbasecloud/ui-platform), RBAC, 11 rules   │
│ Generic: screens, pattern, columns, scale — from intent  │
│ Output:  real .tsx component + .stories.tsx (Storybook)  │
│ Time:    8–12 min per screen                             │
└───────────────────┬─────────────────────────────────────┘
                    │ designer reacts in Storybook · component ships as-is
                    ↓
┌─────────────────────────────────────────────────────────┐
│ SKILL 5: Code Generator                                 │
│ Fixed:   RBAC enforcement, destructive guard, token     │
│          rule, session start checklist, CLAUDE.md       │
│ Generic: components, file paths, fields — from intent   │
│ Output:  typed React/TS container + gap report           │
│ Time:    ~3 hours (vs 3–4 days build)                   │
└───────────────────┬─────────────────────────────────────┘
                    │ engineer reviews → approves
                    ↓
             branch + PR + merge
                    │
                    ↓
             updated source repo
                    └── feeds next feature's Skill 1
```

Skill 5 extends the same component/story files Skill 4 wrote — it does not
rebuild from Skill 3's HTML prototype. Skill 3's only job is to get
the PM/Designer/UX aligned on shape before any real code exists; once that
happens, Skill 4 builds the real thing directly from the (possibly updated)
intent statement.

---

## Quick reference

| Skill | Trigger phrase | Where | Time |
|---|---|---|---|
| 1 — Intent Builder | `Run IDD Skill 1: Intent Builder. Feature seed: [seed]` | Claude Project chat | 15–20 min |
| 2 — Design Brief | 6 prompts (see SKILL.md) | Claude Project chat | ~80 min |
| 3 — Prototype | `Run IDD Skill 3: Prototype.` | Claude Code or Claude Project chat | 8–12 min/screen |
| 4 — UI Component | `Run IDD Skill 4: UI Component.` | Claude Code, real Storybook | 8–12 min/screen |
| 5 — Code Generator | Step-by-step Claude Code session | Terminal | ~3 hours |

---

## Grounding files — required in Claude Project Knowledge

These are Capella-specific and shared across all features.
Upload once. They are used across the five skills.

| File | Skills | What it provides |
|---|---|---|
| `capella-ds-index.json` | 1, 2, 3, 4, 5 | DS tokens, CSS variables, component variants, RBAC role definitions |
| `capella-core-ia.md` | 2, 3, 4, 5 | Layout templates A/B/C-flat/C-split, density standards |
| `capella-core-url-map.md` | 1, 2, 3, 4, 5 | All routes, URL parameters, component file locations |
| `capella-org-control-center.html` | 3, 4, 5 | Real production component patterns |
| `capella_system_prompt_v2.md` | 3 | 11 rules (goes in Project Instructions, not Knowledge) |
| `CLAUDE.md` | 5 only | Code generation rules (goes in repo root) |
| `intent-[feature]-v1.0.md` | 2, 3, 4, 5 | Added per feature after Skill 1 completes |

---

## What each actor does

| Actor | Skill | Produces | Time saved |
|---|---|---|---|
| PM | Skill 1 | Intent statement | Manual spec → 20 min coaching |
| PM | Skill 2 | Design brief | 3–4 weeks email chain → 80 min |
| PM + Designer | Skill 3 | Throwaway HTML prototype + UX iteration | Blank Figma → reacting to a mock, cheap to redo |
| Designer + Engineer | Skill 4 | Real component + Storybook story, design-approved | Design-approved mock IS the shipping code |
| Engineer | Skill 5 | Reviewed container + wired data, ready to commit | 3–4 days build → 3 hours review |

---

## The one rule that runs through all five skills

**The intent statement is the source of truth.**

If the prototype or mock conflicts with the intent → update the intent, then regenerate.
If the code conflicts with the intent → engineer flags it, PM updates intent, re-run Skill 5.
If UX suggests a change → PM scores it against intent outcomes, updates intent, regenerates.
If engineering discovers a constraint → add to §Constraints, version the intent, regenerate.

The intent statement is never bypassed. Every decision traces back to it.

---

## File structure

```
idd-skills/
├── IDD-SKILLS-INDEX.md                    ← this file
│
├── skill-1-intent-builder/
│   ├── SKILL.md                           ← generic coaching methodology + prompt
│   └── examples/
│       └── bucket-scope-collection.md     ← reference example output
│
├── skill-2-design-brief/
│   ├── SKILL.md                           ← 6 generic prompts + assembly guide
│   └── examples/
│       └── bucket-scope-collection.md     ← reference example output
│
├── skill-3-prototype/
│   ├── SKILL.md                           ← throwaway HTML prototype methodology
│   └── examples/                          ← (empty until first feature goes through it)
│
├── skill-4-ui-component/
│   ├── SKILL.md                           ← generic real-component generation
│   └── examples/
│       └── bucket-scope-collection.md     ← reference example session
│
└── skill-5-code-generator/
    ├── SKILL.md                           ← generic Claude Code workflow
    ├── CLAUDE.md                          ← generic repo instruction set
    └── examples/
        └── bucket-scope-collection.md     ← reference example session
```

---

## Adding a new Capella feature

1. Open Claude Project chat
2. Paste Skill 1 prompt with your feature seed
3. Coach the intent statement (15–20 min)
4. Upload `intent-[feature]-v1.0.md` to Project Knowledge
5. Run Skill 2 prompts for the design brief (~80 min)
6. Run Skill 3 to generate the throwaway HTML prototype (8–12 min per screen); iterate fast with PM/Designer/UX until shape is settled
7. Update the intent statement if the prototype changed scope
8. Run Skill 4 to generate the real component + Storybook story from the settled shape (8–12 min per screen)
9. Iterate with UX using Skill 4's comparison prompt for any remaining real-component-level pattern questions
10. Open terminal, run Claude Code with Skill 5 (~3 hours with review)
11. Create branch, commit, open PR

The skills files never change. Only the intent statement changes per feature.

---

## Version history

| Version | Date | Change |
|---|---|---|
| 1.0 | July 2026 | Initial four-skill pipeline, Bucket/Scope/Collection specific |
| 1.1 | July 2026 | Capella fixed, feature generic. Examples folder added. |
| 1.2 | July 2026 | Slash commands, tasks.md, archive pattern — parity with OpenSpec |
| 2.0 | July 2026 | Skill 3 (UI Mock) pilot: real Storybook components replace the throwaway HTML mock, to stop Skill 4 (then Code Generator) from finding new DS bugs late |
| 2.1 | July 2026 | Restored the throwaway prototype as its own stage (new Skill 3), ahead of the real-component mock (renumbered to Skill 4) and code generation (renumbered to Skill 5) — the real-component approach solved DS-fidelity bugs but made early shape iteration more expensive than it should be; five skills now split those two jobs cleanly |
| 2.2 | July 2026 | Skill 1 (Intent Builder) v1.2: added a "batch-and-draft vs. interrogate" pacing rule — sourced drafts + batched confirms for code-groundable dimensions (scale, immutable fields, dangerous defaults, route/pattern), one-at-a-time only where no default exists (Problem, Desired Outcome, RBAC per-role experience, feature boundary). No reduction in coverage — the quality gate gained a 9th item requiring every draft be sourced, not silently assumed |
| 2.3 | July 2026 | Skill 5 (Code Generator) v1.4: added a verified role scenario reference (role × screen matrix, checked against the actual generated code, not the intent's speculative draft) as a standard output alongside tasks.md, for any feature with more than one RBAC-distinct experience. Originated as an ad hoc doc for the Billing Admin/Viewer + Org Read Only feature; promoted to a standard deliverable |
| 2.4 | July 2026 | Renamed Skill 4 from "UI Mock" to "UI Component" (`/idd-mock` → `/idd-ui-component`, `skill-4-ui-mock/` → `skill-4-ui-component/`) — naming only, no behavior change. This skill has generated real, production-primitive components since v2.0; "mock" undersold what it does once v2.1 gave the actual throwaway-mock job to Skill 3 |

---

## v1.2 additions — parity with OpenSpec

Three additions that bring IDD to parity with OpenSpec's developer experience:

### 1 — Slash commands (via CLAUDE.md)

```
/idd-intent     → Skill 1
/idd-brief      → Skill 2
/idd-prototype  → Skill 3
/idd-ui-component → Skill 4
/idd-code       → Skill 5 (also generates tasks.md)
/idd-archive    → archive shipped feature
/idd-status     → show active features
```

No copy-pasting prompts. One command per skill.
CLAUDE.md in the repo root registers all commands automatically.

### 2 — tasks.md (equivalent to OpenSpec's tasks.md)

Generated by Skill 5 after every code generation run.
Template: docs/idd-skills/skill-5-code-generator/tasks-template.md

Covers: hook tasks, component tasks, RBAC wiring, API integration,
routing, gap resolution, design alignment, quality gate, archive checklist.
Engineer tracks progress. PM sees status without asking.

### 3 — Archive pattern (equivalent to OpenSpec's /opsx:archive)

```
/idd-archive [feature-slug]
```

Moves intent + tasks from docs/intents/active/ → docs/intents/archive/[date]-[slug]/
Updates capella-core-url-map.md with new routes.
Every PR links back to its archived intent statement.

---

## How IDD compares to OpenSpec

| | OpenSpec | IDD |
|---|---|---|
| Entry point | Developer with code idea | PM with product problem |
| Slash commands | ✅ /opsx:propose etc | ✅ /idd-intent etc (v1.2) |
| tasks.md | ✅ Generated per change | ✅ Generated per feature (v1.2) |
| Archive | ✅ /opsx:archive | ✅ /idd-archive (v1.2) |
| RBAC-aware | ❌ Not in scope | ✅ First-class — 4 roles |
| Design loop | ❌ Not in scope | ✅ PM → brief → prototype → real mock → decision |
| Product grounding | ❌ Generic | ✅ Capella DS + URL map + IA |
| UX iteration | ❌ Not in scope | ✅ Concept A vs B + outcome scoring |

OpenSpec is generic — works for any project.
IDD is opinionated — correct for Capella specifically.
They are complementary, not competitive.
