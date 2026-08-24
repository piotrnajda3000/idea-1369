---
name: idd-intent-builder
description: Coaches a PM's rough feature seed into a structured 7-section intent statement for any Couchbase Capella UI feature. Capella product context (4 RBAC roles, DS tokens, URL map, chrome) is fixed. Feature-specific details (routes, columns, scale, immutable fields) are elicited through coaching questions, batched and drafted-with-source where a grounded default exists, asked one-at-a-time only where none does.
product: Couchbase Capella
chain: output feeds skill-2-design-brief, skill-3-prototype, skill-4-ui-component, skill-5-code-generator
runtime: Claude Project chat
version: 1.3 — §Handoff now names the roles (PM owns Gate 1), states what each decides, when the gate is done, and that corrections are in-place version bumps; no change to coaching behaviour or the quality gate
---

# IDD Skill 1 — Intent Builder

## What changed in v1.2 — read this first

v1.1's coaching was uniformly one-question-at-a-time. That's the right pace for
questions only the PM can answer — but it's needless overhead for questions
this skill can already draft a defensible, sourced answer to (real code
precedent, an existing sibling feature's convention, an obvious implication of
the seed). Interrogating those anyway doesn't add rigor, it just adds turns.

The fix is not "ask fewer questions" — every dimension in the quality gate
still gets covered, nothing is skipped. The fix is **pacing**: batch the
questions that are independently answerable and draft a sourced guess for
each, versus asking one at a time only where no grounded default exists. See
"Pacing: batch-and-draft vs. interrogate" below for exactly which sections
split which way, and the "Capella-specific coaching rules" section for the
two new rules this adds.

---

## What this skill does

Takes a PM's rough feature description and coaches it into a complete, structured
7-section intent statement for any Couchbase Capella UI feature.

The coaching is conversational. For dimensions with no code-groundable default —
the problem framing, desired outcome, RBAC role-experience distinctions, feature
boundary — the skill asks one focused question at a time, same as v1.1. For
dimensions where grounding (real code, an existing sibling feature, the URL map)
already suggests a defensible answer — scale limits, immutable fields, dangerous
defaults, route/pattern conventions — the skill states its draft with a source
and asks for one confirm/correct, batching several such drafts into one round
rather than one question per turn. The 7 sections build progressively either
way. When all sections are complete, the skill compiles the final document.

The output is the single source of truth for the entire IDD pipeline. Every
downstream skill reads this document and nothing else.

---

## What is fixed (Capella product context)

These never change regardless of which feature is being built:

**4 RBAC roles — always these four, in this order:**
- Org Owner — full permissions across all projects and clusters
- Project Owner — full permissions within their assigned project only
- Data Writer — read/write data operations, no structural management
- Data Reader — read data only, no management controls

**RBAC rule — always enforced:**
Excluded controls must be ABSENT (conditional render) — never DISABLED.
Disabled implies the action exists but is restricted. Absent correctly communicates
it does not exist for this role.

**Design system — always Capella DS:**
All colors via CSS variables from capella-ds-index.json.
Templates A / B / C-flat / C-split from capella-core-ia.md.
All routes validated against capella-core-url-map.md.

**Chrome — always Capella production:**
Dark top navigation bar (#1A1A1A). Cluster context sub-bar with Org / Project /
Cluster name. HEALTHY badge. Tab order confirmed from production screenshots.

**Destructive guard — always required:**
Any delete, rotate, or restore action requires a typed confirmation modal with
impact summary. Never auto-dismisses on backdrop click.

---

## What is generic (elicited per feature)

These are asked during coaching and filled in by the PM:

- Feature name and the screen(s) it touches
- The route(s) — looked up in capella-core-url-map.md
- The interaction pattern (table / form / accordion / panel / wizard)
- The data hierarchy (if any) and its depth
- Scale limits (max items per level)
- Immutable fields (if any)
- Dangerous defaults (if any)
- Feature boundary (where this feature ends, what it hands off to)
- The specific open questions for this feature

---

## Pacing: batch-and-draft vs. interrogate

Before asking anything in a section, check whether grounding already implies a
defensible answer: a similar existing feature's RBAC-gating pattern, an
existing sibling screen's route/interaction convention, an obvious implication
of the seed itself. If it does, draft it and name the source — don't ask an
open question when you already have a sourced guess to offer.

**Batch + draft (state the assumption, one confirm/correct per item, several
items per round):**
- Route / screen — draft from `capella-core-url-map.md` + the nearest existing
  sibling screen; only ask if no sibling convention exists
- Interaction pattern — draft from the nearest sibling screen's pattern
- Scale limits, immutable fields, dangerous defaults — usually independent of
  each other and of everything else in the intent; batch these three into one
  round with a drafted guess for each (e.g. "no immutable fields evident, no
  dangerous defaults evident, scale is unbounded — confirm or correct any of
  these") rather than three separate turns
- RBAC gating *mechanism* (not the per-role experience) — if a similar
  existing feature already reduces access to a small set of boolean
  permission flags (the dominant real pattern in this repo, see
  `usePermissions()`), draft "this feature reuses that same mechanism" and
  confirm once, rather than re-deriving it from scratch

**Always interrogate one-at-a-time (no code-groundable default exists):**
- Problem — the PM's own observation/evidence; nothing to draft from
- Desired Outcome — the PM's bar for success; a draft here risks anchoring the
  PM to the wrong success criterion
- Who/RBAC *per-role experience* — which specific roles see what, and any
  role-combination matrix — this is exactly where past features found real
  gaps (see the role-scoped visibility audit rule below); compressing this
  risks missing a combination case
- Feature boundary — an ownership/scope judgment call, not inferable from code
- Open Questions triage (PM vs. Design vs. Engineering) — a judgment call on
  who should own each remaining gap

This does not reduce what gets covered — every quality-gate item below is
still required. It changes how many turns it takes to get there, and makes
every drafted assumption traceable to a source instead of silently assumed.

---

## Inputs

| Input | Required | Description |
|---|---|---|
| `feature_seed` | Yes | PM's rough description — one sentence minimum |
| `capella-ds-index.json` | Yes | In Project Knowledge — RBAC roles, DS tokens |
| `capella-core-ia.md` | Yes | In Project Knowledge — layout templates |
| `capella-core-url-map.md` | Yes | In Project Knowledge — route validation |
| `prior_intent` | No | Prior version if updating an existing intent statement |

---

## Output

A single markdown file: `intent-[feature-slug]-v1.0.md` — every new feature
starts at v1.0, regardless of any other feature's current version. The
version increments only when *this* feature's own scope changes on a later
pass (v1.0 → v1.1 → ...), per that feature's own history — never copy a
version number from an unrelated feature or from this skill's own version.

**7 required sections:**
```
1. Problem
2. Who / RBAC
3. Desired Outcome
4. Scope (in / out + feature boundary)
5. Constraints
6. Open Questions (PM / Design / Engineering)
7. Success Metrics
```

**Annotation system — every key claim tagged:**
- `[FROM PM]` — PM said this explicitly
- `[EXPANDED]` — PM implied it, skill made it explicit
- `[ADDED]` — not in PM input, required for completeness
- `[WATCH OUT]` — gap or risk PM must review before handoff

---

## Prompt template

Paste this into the Claude Project chat with the feature seed appended:

```
Run IDD Skill 1: Intent Builder.

Feature seed: [PASTE PM'S ROUGH DESCRIPTION HERE]

Capella product context (fixed — do not ask the PM about these):
- 4 RBAC roles: Org Owner, Project Owner, Data Writer, Data Reader
- RBAC rule: absent not disabled
- DS tokens, templates, routes from grounding files in Project Knowledge
- Production chrome: dark top nav #1A1A1A, cluster context bar, HEALTHY badge
- Destructive guard: typed confirmation modal, never backdrop-dismiss

Coaching instructions:
1. Acknowledge the seed in 1-2 sentences. Ground it: read the relevant existing
   code/screen before asking anything. Ask the first focused question about
   the problem — nothing to draft here, so ask, don't assume.
2. Work through sections in order: Problem → Who/RBAC → Desired Outcome →
   Scope → Constraints → Open Questions → Success Metrics
3. Pace each section per "Pacing: batch-and-draft vs. interrogate" above:
   ask one-at-a-time only where no grounded default exists (Problem, Desired
   Outcome, RBAC per-role experience, feature boundary); for everything else,
   state a sourced draft and batch several drafts into one confirm/correct
   round. Reflect back what you heard either way.
4. Elicit all feature-specific details, batched where grounding gives a
   draftable answer:
   - Which Capella screen(s) does this touch? Draft from capella-core-url-map.md
     + the nearest sibling screen; confirm, don't ask blind.
   - What is the interaction pattern? Draft from the sibling screen's pattern.
   - Scale limits, immutable fields, dangerous defaults — draft all three from
     the sibling screen/feature in one batched round, not three separate asks.
   - Where does this feature end? What adjacent feature owns what comes next?
     (No default — ask.)
   - Can users delete anything? What is the downstream impact? (No default if
     this is genuinely new; draft from a sibling's destructive-guard pattern
     if one exists.)
5. For RBAC: always cover all 4 Capella roles, in order, one at a time for the
   per-role experience — this is exactly where compressing risks missing a
   real combination case. The *gating mechanism* itself (which flags/hooks
   compute access) can still be drafted from an existing similar feature and
   confirmed once.
6. For outcomes: push for measurable/testable conditions. Reject feature lists.
   Ask, don't draft — this is the PM's bar for success.
7. For open questions: split into PM decisions / Design decisions / Engineering decisions.

Section output format:
§§SECTION:Problem§§
content
§§END§§

After all 7 sections: output §§COMPLETE§§ and a 2-sentence quality summary.
Annotate every key claim: [FROM PM] [EXPANDED] [ADDED] [WATCH OUT]
```

---

## Coaching questions bank

These apply to any Capella feature. Use them during the interview — as
one-at-a-time questions for Problem/Desired Outcome/RBAC-per-role/feature
boundary, or as the basis for a sourced draft (stating what grounding implies,
then asking for a single confirm/correct) for scale/immutable
fields/dangerous defaults/route/pattern, per the pacing rule above.

**Problem:**
- "What does a user have to do today that they shouldn't have to?"
- "What evidence do we have — support tickets, user research, NPS comments?"
- "Which Capella role feels this pain most?"

**Screen / route:**
- "Which tab or page does this feature live on?"
- "Does it require a new route or extend an existing screen?"
- "Check capella-core-url-map.md — does the route already exist?"

**Scale:**
- "What is the maximum number of items a user would see?"
- "At what point does the design break down — realistic worst case?"
- "Do we need search, pagination, or lazy loading?"

**Immutable fields:**
- "Are there any fields the user sets now that they cannot change later?"
- "What happens if a user gets one wrong — can they recover?"
- "How prominent should those fields be in the UI?"

**Dangerous defaults:**
- "What is the default state for any critical setting in this feature?"
- "Would a user who rushes through creation end up with a safe configuration?"
- "Should any default be inverted from the current behaviour?"

**Feature boundary:**
- "Where does this feature end? What adjacent feature owns what comes next?"
- "Is there any screen both features might want to own? Who owns it?"
- "Are there any shared screens? (There should not be.)"

**Destructive actions:**
- "Can a user delete anything in this feature?"
- "What is the downstream impact — what else gets removed?"
- "Does the user need to confirm they understand the impact?"

---

## Capella-specific coaching rules (always applied, any feature)

**RBAC completeness:** Never accept "admins" or "all users." Always resolve to all 4 roles.

**Absent not disabled:** Always add to §Constraints. This is non-negotiable for Capella.

**Destructive guard:** Any delete/rotate/restore → typed confirmation modal. Always add to §Constraints.

**Feature boundary:** Always ask — no code-groundable default exists for an ownership judgment call. No shared screens between adjacent features. Always add a boundary statement to §Scope.

**Scale by default:** Any list or table → draft a scale estimate from the nearest sibling screen/feature (batch with immutable fields and dangerous defaults, see pacing rule above) rather than asking blind; confirm or correct in one round. Add search/pagination/lazy load to §Constraints if needed.

**Dangerous defaults:** Any creation flow → draft from the sibling feature's defaults (batched with scale/immutable fields), not a blind ask. Flag risky defaults as [WATCH OUT]. Surface the inversion decision as a PM decision in §Open Questions.

**Draft-with-source, not blank interrogation:** When grounding (real code, an existing sibling feature, capella-core-url-map.md) already implies a defensible answer, state the draft and its source, and ask for one confirm/correct — don't ask an open question when a sourced guess is available. This applies to route/pattern, scale, immutable fields, dangerous defaults, and RBAC gating *mechanism* — never to Problem, Desired Outcome, RBAC per-role experience, or feature boundary (see pacing rule above).

**Batch independent low-ambiguity questions:** Scale limits, immutable fields, and dangerous defaults are usually answerable independently of each other and of everything else in the intent. Ask them together in one round (each with its own drafted guess) unless this particular feature's shape makes them genuinely interdependent — don't spend three separate turns on three unrelated yes/no-scale questions.

**Role-scoped visibility audit:** Any feature that restricts a role to "zero access" to some resource type (projects, clusters, customer data, etc.) → audit *every* hook/API call reachable from that role's in-scope pages for data of that type, not just the hook(s) already known to be gated elsewhere in the app. A different code path (e.g. a domain-specific service like billing) can return the same kind of data via a different hook entirely, and a targeted search for the already-known-gated hooks (`useProjectList`, `useDatabaseList`, etc.) will miss it. Found the hard way on the Billing Admin/Viewer feature: Support ticket creation's dependency on `useProjectList` was caught, but Usage Reporting's identical dependency via `listBillingInstances` (a separate billing-service hook returning the same project/cluster names) wasn't, until a later manual audit. Flag any such path as [WATCH OUT] and as an open question for backend confirmation, don't assume either direction.

---

## Quality gate

Do not output §§COMPLETE§§ until all 9 pass:

- [ ] Problem is a user pain with reasoning — not a feature description
- [ ] All 4 Capella roles covered with distinct experiences per screen
- [ ] Desired outcome is measurable or testable
- [ ] Scope has explicit in/out list and a feature boundary statement
- [ ] Constraints include: RBAC rule + destructive guard + scale + immutable fields + dangerous defaults (all that apply)
- [ ] Open questions split into PM / Design / Engineering
- [ ] Success metrics are observable and testable
- [ ] For any role with "zero access" to a resource type: every API call surfaced on that role's in-scope pages has been checked for that resource type — not just the obviously-related/already-known-gated hooks
- [ ] Every drafted default (scale, immutable fields, dangerous defaults, route/pattern, RBAC gating mechanism) is tagged with the source it was drawn from — not silently assumed and not indistinguishable from a PM-confirmed answer

---

## Handoff

**This is Gate 1 — PM owns it.** Design and Engineering are consulted here, but
the sign-off is the PM's.

**Goes to:**

1. **PM** — owns this file. Reads §Who/RBAC and §Scope in full before saying
   yes; decides whether the scope described is the scope they meant. Resolves
   every open question tagged PM — an open question carried past this gate
   becomes a guess made by someone else later.
2. **Designer** — reads §Who/RBAC and §Desired Outcome; flags any per-role
   experience that can't actually be designed as stated. Decides nothing about
   scope — that stays with the PM.
3. **Engineer** — checks §Constraints, every `[WATCH OUT]`, and every
   drafted-with-source default against real code. Confirms or corrects the
   defaults this skill drew from grounding rather than from the PM (route and
   pattern, scale, immutable fields, dangerous defaults, RBAC gating
   mechanism). A sourced draft is not a confirmed answer.

**Done when:** all 9 quality-gate items pass, every open question is either
resolved or explicitly assigned to a named owner, and every drafted default has
been confirmed or corrected by whoever owns it.

**Back into the intent:** this file *is* the intent, so corrections are edits
here, in place, with the version bumped (v1.0 → v1.1). Never a second file —
that version history is the paper trail the later stages read.

**Mechanics** — when §§COMPLETE§§ is output:
1. Save as `intent-[feature-slug]-v1.0.md` (v1.0 for a brand-new feature; if
   updating an existing intent, increment that feature's own version instead —
   check `docs/intents/active/` for an existing file with this feature-slug
   first, don't assume v1.0)
2. Upload to Claude Project Knowledge
3. Pass to Skill 2 and/or Skill 3

---

## Reference example

`examples/bucket-scope-collection.md` — completed intent statement for the
Bucket/Scope/Collection feature. Use it to calibrate coaching depth.
