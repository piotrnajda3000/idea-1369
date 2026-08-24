---
name: idd-ui-component
description: Generates a real, production-primitive UI component for a Couchbase Capella feature from its intent statement — a presentational React component plus a Storybook story file, not a mock or approximation of any kind. Capella RBAC model and async-state rules are fixed. All feature-specific content (screens, routes, columns, interactions, scale) is read from the intent statement at runtime.
product: Couchbase Capella
chain: consumes skill-1-intent-builder output + skill-3-prototype as settled reference · parallel with skill-2-design-brief · feeds skill-5-code-generator
runtime: Claude Code, against the real cp-ui-v3 repo and its real Storybook instance
version: 2.3 — §Handoff restructured to the shared shape (goes to / decides / done when / back into the intent); named as the second half of Gate 2, with the rule that behavioural corrections go back into the intent and purely visual ones stay in the PR. No behavior change to generation.
---

# IDD Skill 4 — UI Component

## What changed from v1.1 — read this first

v1.1 generated a single self-contained HTML file that *approximated* the Capella
design system with hand-written CSS. It was fast and needed no build step, but
it was a translation — not the real thing. That gap is exactly why real code
generation (then Skill 4, now Skill 5) kept finding new bugs the HTML mock
never could (wrong `Button` prop shape, wrong `Anchor` emphasis variant, wrong
grid API) — the mock was never actually built from `@couchbasecloud/ui-platform`.

v2.0 closed that gap by making this skill write real production code from the
first draft, and v2.1 (this version) keeps that, but no longer treats the
throwaway HTML mock as merely an exception case — that fast, cheap iteration
stage is back as its own skill, **skill-3-prototype**, which runs *before*
this one. The division of labor now:

- **Skill 3 (Prototype)** is where shape gets decided — layout, flow, which
  pattern to use — cheaply, in disposable HTML, with the PM/Designer/UX.
- **Skill 4 (this skill)** starts once that shape is reasonably settled, and
  builds the real thing directly — no second translation step, no risk of the
  real component silently drifting from an approved-but-approximate mock.

Concretely, this skill:

- Writes the actual **presentational** React component(s) for the
  feature — real imports from `@couchbasecloud/ui-platform` and this repo's
  own `components/`, real TypeScript prop types, no hooks, no data fetching.
  Data and callbacks arrive as props/args only.
- Alongside each component, writes a `*.stories.tsx` file using this
  repo's real, established pattern (548 existing story files as precedent —
  see `src/pages/database/buckets/components/database-buckets-list/database-buckets-list.stories.tsx`
  for a feature-level, non-atomic example) — one story per RBAC role, per
  async state, per edge case, using this repo's real fixture utilities
  (`mocks/permission`, `mocks/utils/wrap-with-rbac`, `mocks/responses/*`).
- Skill 5's job shrinks accordingly: it adds a thin **container** that calls
  the real hooks (`usePermissions`, `use-proxy-scope-list`, etc.) and passes
  real data down as props into the *same* presentational component this skill
  wrote — extending the file, not rewriting it. The component that got
  reviewed in Storybook is the component that ships.
- There is no custom "role switcher" widget here — RBAC states are
  expressed as separate stories using real permission-shaped fixtures
  (`permissionMock` overrides via `wrap-with-rbac`), reviewed by picking a
  story in Storybook's sidebar, same as any other component in this repo.
  (Skill 3's HTML prototype does use a role-switcher widget — that's fine
  there, since nothing it produces ships.)
- There is no full-page chrome to reproduce (dark nav, breadcrumb, HEALTHY
  badge) — Storybook's global decorator (`.storybook/preview.tsx`,
  `StorybookProvider`) already supplies real app context. Feature review
  happens at the component/screen-section level, matching how this repo's
  Storybook is actually organized today (no page-level stories exist as
  precedent — don't invent one; review the feature's real components in
  isolation, the way `database-buckets-list.stories.tsx` does for the bucket
  list).
- Versioning changes too: **this is no longer a new file per revision.** The
  component and story files are living artifacts — edit them in place as
  design feedback comes in, the same living-document discipline already used
  for the intent statement itself. Git history *is* the version history; a
  reviewer reads the diff, not a `v2` filename.

If real DS fidelity or component-structure feedback surfaces a shape problem
(not just a token/variant fix), that's a sign the feature needs to go back to
Skill 3 and settle the shape question there — don't keep iterating shape
inside a real component; that's the expensive way to do it.

---

## What this skill does

Generates the real presentational component(s) for a Capella UI feature
described in an intent statement, plus a Storybook story file exercising every
RBAC role view, async state, and scale/edge case named in that intent — all
read from the intent statement at runtime, all built from real
`@couchbasecloud/ui-platform` and repo components.

The designer runs Storybook (`npm run storybook:watch`, port 6006) and reacts
to the real component. The engineer (Skill 5) wires real data into the same
files. The PM reviews the same PR diff everyone else does.

---

## What is fixed (Capella product context)

**RBAC model — always Capella's role model:**
- Express each of the 4 relevant personas (Org Owner / Project Owner / Data
  Writer / Data Reader) as its own story, built from a real permission fixture
  — start from `mocks/permission`'s `permissionMock` and override only the
  fields the intent's §Who/RBAC table says differ for that role/screen.
- Excluded controls are absent (conditional render), never disabled — same
  rule as always, now enforced by the real component's own conditional JSX,
  not a mock-only convention.

**Design system — inherited automatically, not hand-copied:**
Because the component imports real `Button`, `Dialog`, `ConfirmDialog`,
`Text`, `Anchor`, `DataGridWrapper`, etc. from `@couchbasecloud/ui-platform`
and this repo's `components/`, correct tokens/variants/spacing come for free.
There is nothing to "match against `capella-ds-index.json`" anymore at this
stage — that index remains the reference for *which* variant name is real
(don't invent a `Button` variant), but there's no CSS to hand-write.

**Async states — always all four, expressed as separate stories:**
- Loading — a story (or a `loading` arg) showing the real skeleton/loading
  state the component renders
- Empty — a story with empty fixture data
- Error — a story with an error-shaped fixture
- Success — implicit in the `Default` story; toast behavior is described in
  the state transitions table, not literally shown in a static story

**Destructive guard — always required, same as production:**
Any delete/rotate/restore control renders the real `ConfirmDialog` with
`confirmationValue` + `isCaseSensitive`, never `enableOutsideClickClose`. If
the component under this skill includes a destructive action, its story must
include a story state with the confirm dialog open.

---

## What is generic (read from intent statement at runtime)

- Which existing component directory this belongs in (find the sibling
  screen/feature it extends under `src/pages/**` — match its naming
  convention, don't invent a new top-level location)
- Which template/layout it composes with (Template A / B / C-flat / C-split —
  from §Navigation model) — the *component* itself typically does not own
  chrome, so this mostly informs prop shape (e.g. does it need a `readonly`
  prop the way `BucketForm` does)
- The interaction pattern, columns/fields/hierarchy (from §Scope)
- Scale requirements — search, pagination, lazy load (from §Constraints) —
  express as fixture arrays sized to match (e.g. a 20-item fixture array for
  a "paginate at 20" constraint, not just 2–3 sample rows)
- Immutable fields and their visual treatment (from §Constraints)
- Dangerous defaults and their UI treatment (from §Constraints)
- RBAC experience per role per screen (from §Who/RBAC) — one story per role
- Destructive actions and their modal content (from §Constraints)

---

## Inputs

| Input | Required | Description |
|---|---|---|
| `intent-[feature]-v[n].md` | Yes | `docs/intents/active/` — all feature context |
| `docs/capella-ds-index.json` | Yes | Real component/variant names — don't invent one |
| `docs/capella-core-ia.md` | Yes | Layout template patterns |
| `docs/capella-core-url-map.md` | Yes | Route validation |
| The real sibling component(s) under `src/pages/**` | Yes | Naming convention, existing prop patterns to match |
| `src/mocks/permission.ts`, `src/mocks/utils/wrap-with-rbac.ts`, `src/mocks/responses/*` | Yes | Real fixture utilities — reuse, don't reinvent |
| A running Storybook instance (`npm run storybook:watch`) | For review, not generation | Needed by the reviewer, not by this skill while writing files |

---

## Output

For the primary screen/feature:
- The real presentational component file(s), in the correct existing
  directory (e.g. `src/pages/database/buckets/components/<feature>/<name>.tsx`)
  — typed props, no hooks, no data fetching, callbacks passed in as props
- A co-located `<name>.stories.tsx` following this repo's real pattern:
  - `Default` story — the common-case fixture
  - One story per RBAC role that has a visibly different experience per
    §Who/RBAC (skip roles with an identical view to `Default`)
  - `Empty`, `Loading`, `Error` stories as applicable
  - A story with any destructive-confirmation dialog open, if applicable
- Header comment on the component file (same format as Skill 5 uses):
  ```tsx
  /**
   * [ComponentName]
   * Intent: [intent filename] §[section]
   * Route:  [route from intent §Navigation model]
   * RBAC:   [roles that see this component]
   * Note:   presentational only — Skill 5 adds the data-fetching container
   */
  ```

After the files: output the state transitions table (Action | Result |
Condition) and a short gap report (anything the intent didn't specify that
had to be assumed for the fixture data — flag it, don't silently invent it).

---

## Prompt template — primary screen

```
Run IDD Skill 4: UI Component.

Read the intent statement at docs/intents/active/[intent-filename].
Read the grounding files (capella-ds-index.json, capella-core-ia.md,
capella-core-url-map.md).
Find the existing sibling component(s) this feature extends under
src/pages/** and match its directory/naming convention exactly.

Before writing any code, output:
  Component path: [where this will live, matching an existing sibling]
  Route:          [from §Navigation model in intent]
  RBAC scope:     [roles from §Who/RBAC that see this component]
  Reused DS:      [real component names from @couchbasecloud/ui-platform used]
  Destructive:    [list destructive ops from §Constraints, if any]
  Gaps:           [anything §Scope/§Constraints didn't specify, that a fixture had to assume]

Then write:
1. The presentational component — real imports, typed props, no hooks, no
   data fetching. Data and callbacks arrive as props only.
2. Its story file — real fixtures via mocks/permission and
   mocks/utils/wrap-with-rbac, one story per RBAC role with a distinct view,
   plus Empty/Loading/Error stories and a destructive-confirm-open story if
   applicable.

Requirements from §Who/RBAC in the intent statement:
- Each role's story reflects the experience defined in §Who/RBAC
- Excluded controls are absent (conditional render) in the component's own
  JSX, never disabled, unless the intent names an explicit exception

Requirements from §Scope in the intent statement:
- Implement the interaction pattern described
- Include all columns/fields/hierarchy levels described
- Fixture data sized to exercise scale requirements from §Constraints

Requirements from §Constraints in the intent statement:
- Mark immutable fields with the visual treatment specified
- Implement dangerous-default UI treatment as specified
- Destructive actions use the real ConfirmDialog, confirmationValue set,
  enableOutsideClickClose left unset

No placeholders, no TODOs. The component must compile and its stories must
render when Storybook is run.

After the files, output the state transitions table:
| User action | Result | Condition |
```

---

## Prompt template — secondary screens / additional components

For features with multiple screens or sub-components (§Navigation model has
more than one route, or §Scope describes nested pieces like row-level
components):

```
Run IDD Skill 4: UI Component — [component name].

The primary component has already been generated.
Now generate [component name], following the same real-component,
fixture-driven approach.

Read §Navigation model / §Scope in the intent statement to understand how
this component is reached and what it contains.

[Same requirements as the primary-screen template above.]

If this component is a form (Template B / a create-style flow):
- Generate all fields from §Scope, with real form components
- Mark all immutable fields from §Constraints
- Implement dangerous defaults from §Constraints
- A story per meaningful validation/error state
```

---

## Prompt template — UX iteration comparison

Use this when the shape question is already settled (that's Skill 3's job)
but UX wants to compare two real-component-level treatments of a pattern
that already exists in this component — e.g. two variants of the same
interaction, not two different screen layouts:

```
Run IDD Skill 4: UX Iteration — Concept A vs Concept B.

Context:
- Current pattern: [describe what the component currently does]
- UX suggestion: [describe what UX proposed]
- Reason from UX: [why they suggested it]

Generate two Storybook stories on the same component (or a temporary
variant prop) — ConceptA and ConceptB — so both are reviewable side by side
in Storybook's sidebar, real components in both.

Alongside, output a short PM decision note (not a UI, just text in chat):
- Score each concept against the desired outcomes from §Desired Outcome in
  the intent statement, using the intent's own outcome language
- ✅ what each concept does well against those outcomes
- ⚠️ what each concept costs (engineering, DS compatibility, scale)
- The decision is against intent outcomes, not aesthetic preference

Once the PM picks a concept, delete the losing story/variant — don't leave
dead code behind.
```

---

## Living-artifact rule (replaces the old versioning table)

Unlike v1.1's `mock-[feature]-v[n].html` filenames, this skill's output is
edited in place as feedback comes in:

- The component and story files carry no version number in their filename —
  they're addressed by their real path, same as any other component in this
  repo
- Each round of design feedback is a normal edit to the same files; the PR
  diff (or, before a PR exists, `git diff`) *is* the change history
- The intent statement's own version still increments when *scope* changes
  (v1.0 → v1.1 etc., per Skill 1) — the component/story files just need to
  stay in sync with whichever intent version is current, referenced in the
  header comment
- If a concept comparison (see above) produces a real pivot, land the winning
  concept in the real files and delete the losing variant — don't keep both
  around as "v1 vs v2"

---

## Handoff

**This is the second half of Gate 2 — Design signs off on the real rendering.**
Don't let it advance to Skill 5 until you'd actually ship this screen.

**Goes to** — the component + story files:

1. **Designer** — runs `npm run storybook:watch`, opens the story, reacts to
   the real thing; flags discrepancies as PR comments or directly in chat.
   Decides whether the real DS rendering matches the intent — this is the
   sign-off the prototype could not give, because the prototype only
   approximated the design system.
2. **Engineer (Skill 5)** — adds a container component that wires real hooks
   (`usePermissions`, the real data hooks) and passes real data as props into
   this same presentational component; the story's fixture props describe
   the exact shape the container needs to produce.
3. **PM** — reviews the same PR everyone else does; state transitions table
   above is the interaction spec for anyone who hasn't opened Storybook.
   Confirms every RBAC role in §Who/RBAC has a story and renders as described.

**Done when:** the designer has opened the story and signed off on the real
rendering, and every RBAC role, async state, and edge case named in the intent
has a story that behaves as the intent says. Reading the component file is not
sign-off — two real bugs in the reference run showed up only by clicking.

**Back into the intent:** any correction that changes *what the feature does* —
a role that turns out to need a different experience, a state nobody had
specified — goes into the intent, version bumped. Corrections that are purely
visual (spacing, token choice, copy) stay in the PR; they don't belong in the
intent and shouldn't inflate its version history.

---

## Reference example

None yet — this is a pilot version. The first feature that goes through this
flow end-to-end should be added here as `examples/<feature>.md`, the same way
`examples/bucket-scope-collection.md` documents the v1.1 HTML-mock flow. Until
then, use `database-buckets-list.stories.tsx` as the closest real precedent
for what a feature-level (not atomic) story in this repo looks like.
