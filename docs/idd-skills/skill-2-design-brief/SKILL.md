---
name: idd-design-brief
description: Generates a complete designer-ready brief from any Capella IDD intent statement. Produces 6 outputs — user flows, RBAC matrix, edge cases, competitive references, lo-fi layout concepts, and pre-answered open questions. Capella DS and RBAC context is fixed. All feature-specific content is read from the intent statement at runtime.
product: Couchbase Capella
chain: consumes skill-1-intent-builder output · feeds skill-3-prototype
runtime: Claude Project chat
version: 1.2 — added a §Handoff section (this skill had none): who owns which Prompt 6 decision, and the rule that scope/RBAC answers go back into the intent, not just into the brief
---

# IDD Skill 2 — Design Brief

## What this skill does

Takes the intent statement from Skill 1 and generates everything a designer
needs to start wireframing — for any Capella UI feature. Six prompts, each
producing one section of the brief. Run them in sequence or selectively.

All feature-specific content (routes, screens, interaction pattern, scale limits,
immutable fields) is read directly from the intent statement. Nothing is hardcoded
in these prompts.

The RBAC matrix (Prompt 2) is the highest-value output — run this one live in demos.

---

## What is fixed (Capella product context)

**RBAC model — always Capella's 4 roles:**
Org Owner / Project Owner / Data Writer / Data Reader.
The matrix always covers all 4 × all screens defined in the intent statement.

**Design system — always Capella DS:**
Template references (A / B / C-flat / C-split) come from capella-core-ia.md.
Layout constraints come from the same file.

**Chrome — always Capella production:**
Any layout concept must be compatible with Capella's dark top nav, cluster context
bar, and tab structure. Concepts that require layout changes incompatible with
the existing chrome are flagged, not recommended.

**Absent not disabled — always the implementation rule:**
Every RBAC matrix output ends with this rule stated explicitly.

---

## What is generic (read from intent statement at runtime)

- Feature name and screens
- Routes (from §Navigation model in intent)
- Interaction pattern (from §Scope in intent)
- Scale limits (from §Constraints in intent)
- Immutable fields (from §Constraints in intent)
- Dangerous defaults (from §Constraints in intent)
- Open questions (from §Open Questions in intent)
- Desired outcomes (from §Desired Outcome in intent)

---

## Inputs

| Input | Required | Description |
|---|---|---|
| `intent-[feature]-v1.0.md` | Yes | In Project Knowledge — all feature context lives here |
| `capella-ds-index.json` | Yes | In Project Knowledge |
| `capella-core-ia.md` | Yes | In Project Knowledge |
| `capella-core-url-map.md` | Yes | In Project Knowledge |

---

## Output — 6 sections assembled into design brief

| # | Section | Designer uses it for |
|---|---|---|
| 1 | User flows | Mapping screens and decision points |
| 2 | RBAC matrix | Knowing how many views to build per screen |
| 3 | Edge cases | Wireframe scope — every state accounted for |
| 4 | Competitive references | Adopt / adapt / avoid decisions |
| 5 | Lo-fi layout concepts | 2–3 directions to react to, not copy |
| 6 | Open questions pre-answered | No clarification emails |

---

## Prompt 1 — User flows

```
Run IDD Skill 2, Prompt 1: User Flows.

Read the intent statement for this feature from Project Knowledge.

Generate step-by-step user flows based on the §Navigation model,
§Who/RBAC, and §Desired Outcome sections of the intent statement.

Cover all 4 Capella roles: Org Owner, Project Owner, Data Writer, Data Reader.
Use the role definitions from capella-ds-index.json in Project Knowledge.

For each role:
- Pre-condition: what state is the user in before they start?
- Primary flow: numbered steps from entry point to success state
- Decision points: where does the flow branch? (mark with ▲)
- Error paths: what can go wrong and what does the user see?
- Exit: where does the flow end and what is the user's next natural action?

Validate every route against capella-core-url-map.md.
Flag any step where a route does not exist (mark with ⚠️ ROUTE GAP).
Flag any step where an API endpoint is assumed but not confirmed (⚠️ API GAP).

Format: one flow per role. Decision points marked ▲. Gaps marked ⚠️.
```

---

## Prompt 2 — RBAC matrix ← run this live in demos

```
Run IDD Skill 2, Prompt 2: RBAC Experience Matrix.

Read the intent statement for this feature from Project Knowledge.
Use the §Who/RBAC and §Navigation model sections.

Generate the RBAC experience matrix for this feature.

PART 1: Role × Screen table
- Rows: one per role (Org Owner / Project Owner / Data Writer / Data Reader)
- Columns: Role | Screen/Route | Sees | Can do | Absent (never disabled) | Empty state
- Cover every route and screen defined in §Navigation model of the intent statement

PART 2: Design implications
- How many distinct view states must be designed?
- Which controls are conditionally rendered absent vs always present?
- What RBAC banners or indicators are needed per role?
- What does the empty state look like per role?
- Is there any role that needs a completely different layout (not just hidden controls)?

End with: the "absent not disabled" implementation rule and what it means
for the React component — conditional render, not disabled prop or CSS opacity.
```

---

## Prompt 3 — Edge cases and states

```
Run IDD Skill 2, Prompt 3: Edge Cases and UI States.

Read the intent statement for this feature from Project Knowledge.
Use §Scope, §Constraints, and §Who/RBAC sections.

Generate the complete UI state inventory. This defines wireframe scope.

1. VALIDATION STATES
   Based on §Constraints — what fields have validation? What are the invalid states?
   What prevents form submission?

2. DESTRUCTIVE ACTION STATES
   Based on §Constraints — what can be deleted? What is the impact summary per entity type?
   What is the typed confirmation modal content?
   Is there a hard block condition?

3. SCALE / PAGINATION STATES
   Based on §Constraints scale limits — what does the list look like at max capacity?
   What triggers search? What does "no results" look like?
   What does the "show more" pattern look like?

4. ASYNC STATES
   Loading: what skeleton pattern per screen/level?
   Success: what toast or confirmation?
   Error: what does an API error look like inline?
   Empty: what does this screen look like on a brand new cluster?

5. RBAC EDGE CASES
   Based on §Who/RBAC — what happens if a restricted role navigates directly
   to a management URL? What do they see?

Flag which states are commonly missed in wireframes.
```

---

## Prompt 4 — Competitive references

```
Run IDD Skill 2, Prompt 4: Competitive References.

Read the intent statement for this feature from Project Knowledge.
Use §Scope and §Desired Outcome to understand the interaction pattern.

Based on the interaction pattern described in the intent statement,
analyse how comparable products handle a similar pattern.

For each relevant reference:
- What pattern do they use?
- What works well for the Capella use case?
- What breaks down for the Capella use case?
- Adopt / Adapt / Avoid — with reason tied to the intent statement's desired outcomes

End with a clear recommendation for the pattern that best serves:
- The desired outcomes from §Desired Outcome
- The scale constraints from §Constraints
- Compatibility with Capella's Template A/B/C-flat/C-split from capella-core-ia.md

Do not recommend patterns that require layout changes incompatible with
the existing Capella chrome (dark top nav, cluster context bar, tab structure).
```

---

## Prompt 5 — Lo-fi layout concepts

```
Run IDD Skill 2, Prompt 5: Lo-Fi Layout Concepts.

Read the intent statement for this feature from Project Knowledge.
Read capella-core-ia.md for template constraints.

Generate 2–3 distinct layout concepts for the primary screen
defined in §Navigation model of the intent statement.

For each concept:
- Name and one-line description
- ASCII wireframe showing the key layout decision
- Template assignment from capella-core-ia.md (A / B / C-flat / C-split)
- How it handles the interaction pattern from §Scope
- How it handles the scale constraints from §Constraints
- Trade-offs: what it does well, what it sacrifices

Requirements:
- At least one concept must be conservative — fits existing Capella patterns
- At least one concept should be the most ambitious direction UX would want
- All concepts must be compatible with Capella's production chrome

End with: which concept best satisfies the §Desired Outcome from the intent
statement, and why. Frame this as a PM product decision, not a design opinion.
```

---

## Prompt 6 — Open questions pre-answered

```
Run IDD Skill 2, Prompt 6: Open Questions Pre-Answered.

Read the §Open Questions section of the intent statement from Project Knowledge.

For each PM decision:
- What does the intent statement imply the answer should be?
- What additional information is needed to confirm?
- What the designer cannot finalise until this is resolved?

For each Design decision:
- State 2–3 design options
- What §Desired Outcome says should guide the choice
- Explicitly hand it back: "This is yours to own — the intent says [X], the rest is your call"

For each Engineering decision:
- What the designer needs from engineering before wireframes can finalise
- Which wireframe states are blocked until this is answered
- Who should answer it (frontend / backend / PM + engineering)

End with: a prioritised list of the 3 questions that must be answered before
design starts, and which can be resolved during design.
```

---

## Assembling the brief

After running all 6 prompts, assemble into:

```markdown
# Design Brief — [Feature Name from intent §Navigation model]
Intent statement: intent-[feature]-v1.0.md
Date: [date]
Designer: [name]
PM: [name]

## 1. User Flows
[Prompt 1 output]

## 2. RBAC Matrix
[Prompt 2 output]

## 3. Edge Cases & States
[Prompt 3 output]

## 4. Competitive References
[Prompt 4 output]

## 5. Layout Concepts
[Prompt 5 output]

## 6. Open Questions
[Prompt 6 output]
```

Save as `design-brief-[feature-slug]-v1.md`.

---

## Handoff

Not a gate — this stage produces a working document, not a sign-off. But it is
where the three roles first split up, so say the split out loud.

**Goes to:**

1. **Designer** — owns §5 Layout Concepts and every Design decision Prompt 6
   handed back. Decides the visual and interaction direction; takes it either
   to wireframes or straight to Skill 3. This is the "yours to own" half of
   Prompt 6 — the brief deliberately stops short of deciding it.
2. **PM** — answers the prioritised 3 questions Prompt 6 flagged as
   blocking-before-design-starts. Decides anything that moves the feature
   boundary.
3. **Engineer** — answers the Engineering questions from Prompt 6, and says
   which wireframe states stay blocked until backend confirms.

**Done when:** the 3 must-answer-before-design questions each have a named
owner and an answer, and the Design decisions have been explicitly handed to
the designer rather than pre-decided in the brief.

**Back into the intent:** any answer that changes §Scope, §Who/RBAC, or
§Constraints goes into the *intent statement*, version bumped — not only into
this brief. The brief is a derived document; the intent is the source of truth,
and Skills 3, 4, and 5 all read the intent, not this file. An answer that lives
only in the brief is invisible to every stage after this one.

---

## Time

| Prompt | Time |
|---|---|
| 1 — User flows | 8–10 min |
| 2 — RBAC matrix | 5–8 min |
| 3 — Edge cases | 8–10 min |
| 4 — Competitive refs | 10–12 min |
| 5 — Layout concepts | 12–15 min |
| 6 — Open questions | 10–12 min |
| Assembly | 10–15 min |
| **Total** | **~75–85 min** |

---

## Reference example

`examples/bucket-scope-collection.md` — completed design brief for the
Bucket/Scope/Collection feature. Use it to calibrate output depth.
