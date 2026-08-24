---
name: idd-prototype
description: Generates a fast, throwaway, self-contained HTML/CSS mock for a Couchbase Capella feature from its intent statement — a visual approximation of the design system, not real code. Used for early, cheap design iteration with PM/Designer/UX before any real component exists. Capella RBAC model and async-state rules are fixed. All feature-specific content (screens, routes, columns, interactions, scale) is read from the intent statement at runtime.
product: Couchbase Capella
chain: consumes skill-1-intent-builder output · optionally consumes skill-2-design-brief · feeds skill-4-ui-component once design is settled
runtime: Claude Code or Claude Project chat
version: 1.2 — §Handoff restructured to the shared shape (goes to / decides / done when / back into the intent); it is now named as the first half of Gate 2. No change to what this skill generates.
---

# IDD Skill 3 — Prototype

## Why this exists

Skill 4 (UI Component) generates the real presentational component + Storybook
story from the first draft — real `@couchbasecloud/ui-platform` imports, real
typed props, no HTML approximation. That's deliberate: it's what stopped code
generation (Skill 5) from hitting new DS bugs late (wrong `Button` prop shape,
wrong `Anchor` variant, wrong grid API).

But a real component is a heavier thing to redo than a sketch. Once a screen
exists as typed props and real Storybook stories, throwing away a whole
layout direction and trying a different one costs more than it should for a
stage where the actual question is still "does this shape make sense" —
information hierarchy, flow between screens, which pattern to use — not "is
this pixel-perfect against the design system."

This skill is that earlier, cheaper stage. It produces a single self-contained
HTML file that *approximates* the Capella design system with hand-copied CSS
variables — good enough to react to, disposable enough to fully rewrite
without anyone feeling the sunk cost. Once the PM/Designer are aligned on the
shape, Skill 4 rebuilds the agreed direction as the real component that
actually ships.

**Real precedent for this exact format already exists in this repo** at
`docs/mocks/mock-bucket-scope-collection-intent-v2.3-mock-v1.html` (and its
sibling screens in the same directory) — use it as the reference for
structure, the `.mock-controls` role-switcher pattern, and how CSS variables
are hand-copied from `docs/capella-ds-index.json` into a `:root` block.

---

## What this skill does

Generates one self-contained HTML file per screen (or a small set of related
screens) for a Capella feature, from its intent statement — approximating
Capella's chrome, DS tokens, and RBAC-driven visibility with hand-written CSS
and a role switcher, so a PM/Designer/UX reviewer can react to something
concrete without a build step, a running dev server, or committed code.

Nothing this skill produces ships. Its only job is to get a design direction
to "yes, build this for real" as fast as possible — including "no, try a
completely different direction," which should be just as cheap.

---

## What is fixed (Capella product context)

**RBAC model — always Capella's role model:**
- Include a `.mock-controls` role switcher (real precedent: the file cited
  above) covering the roles named in the intent's §Who/RBAC — switching
  updates which controls/sections are visible in the mock
- Excluded controls are hidden entirely when switched to a role that doesn't
  have them — same absent-not-disabled rule as production, even in the
  approximation

**Design system — approximated, not real:**
- Hand-copy the CSS variables this screen actually needs from
  `docs/capella-ds-index.json` into a `:root` block — real color/spacing/type
  values, just not the real component library
- Reproduce production chrome (app bar, breadcrumbs, tab nav) well enough to
  read correctly in context — this is the one stage where full-page chrome
  is worth reproducing, since there's no Storybook decorator supplying it yet
- This is an approximation by design — do not treat mismatches against the
  real DS as bugs to fix here; that fidelity is Skill 4's job, not this one

**Async states — represent all four, cheaply:**
- Loading, Empty, Error, Success can be toggled controls in `.mock-controls`
  (a `<select>` or buttons) rather than separate files — the point is fast
  switching, not separate artifacts

**Destructive guard — represent the shape, not the real dialog:**
- Show the confirm-dialog layout (typed-name-to-confirm, impact summary) as
  part of the same HTML file — it doesn't need to be the real `ConfirmDialog`
  component, just needs to communicate the same information architecture

---

## What is generic (read from intent statement at runtime)

- Which screens exist and how they connect (§Navigation model)
- The interaction pattern, columns/fields/hierarchy (§Scope)
- Scale requirements — represent with enough sample rows to read correctly
  (§Constraints)
- Immutable fields and their visual treatment (§Constraints)
- Dangerous defaults and their UI treatment (§Constraints)
- RBAC experience per role per screen (§Who/RBAC) — drives the role switcher
- Destructive actions and their modal content (§Constraints)

---

## Inputs

| Input | Required | Description |
|---|---|---|
| `intent-[feature]-v[n].md` | Yes | `docs/intents/active/` — all feature context |
| Design brief (Skill 2 output), if one exists | No | Lo-fi concepts to start from instead of a blank page |
| `docs/capella-ds-index.json` | Yes | Real token values to hand-copy — don't invent a color/spacing value |
| `docs/capella-core-ia.md` | Yes | Layout template patterns to approximate |
| `docs/capella-core-url-map.md` | Yes | Route validation |
| `docs/mocks/mock-bucket-scope-collection-intent-v2.3-mock-v1.html` | For reference | Real precedent for structure/conventions |

---

## Output

One self-contained `.html` file per screen (or a small related set), saved to
`docs/mocks/`:

```
docs/mocks/mock-[feature-slug]-v[n].html
docs/mocks/mock-[feature-slug]-v[n]-[screen-name].html   (if multiple screens)
```

Each file is fully self-contained — inline `<style>`, no external build step,
opens directly in a browser. No production imports, no TypeScript.

After the file(s): a short PM decision note in chat — what direction was
taken and why, referencing the intent's §Desired Outcome language, plus any
open questions the prototype surfaced that the intent statement should
capture before Skill 4 begins.

---

## Prompt template

```
Run IDD Skill 3: Prototype.

Read the intent statement at docs/intents/active/[intent-filename].
Read the design brief for this feature if one exists.
Read docs/capella-ds-index.json for real token values — hand-copy what this
screen needs into a :root block, don't invent values.
Use docs/mocks/mock-bucket-scope-collection-intent-v2.3-mock-v1.html as the
structural reference for chrome, the mock-controls role switcher, and how
tokens are copied in.

Before writing the file, output:
  Screens:        [from §Navigation model]
  RBAC scope:     [roles from §Who/RBAC covered by the role switcher]
  Destructive:    [list destructive ops from §Constraints, if any]
  Gaps:           [anything §Scope/§Constraints didn't specify, that had to be assumed]

Then write one self-contained HTML file per screen to docs/mocks/, named
mock-[feature-slug]-v[n]-[screen-name].html.

Requirements:
- Reproduce production chrome (app bar, breadcrumbs, tab nav) well enough to
  read in context
- A .mock-controls role switcher covering every role in §Who/RBAC; switching
  hides controls that role doesn't have (absent, not disabled)
- Represent Loading/Empty/Error/Success as togglable controls, not separate
  files
- Represent destructive-confirm flows with the same information the real
  ConfirmDialog would show (typed-name-to-confirm, impact summary) — doesn't
  need to be the real component
- No build step, no TypeScript, no production imports — this is throwaway

After the file(s), output a short PM decision note: what direction was
chosen, why (against §Desired Outcome), and any open questions the prototype
surfaced.
```

---

## When to iterate vs. move to Skill 4

- **Still iterating on shape** (layout, flow, which pattern, what's on each
  screen) → keep editing this same HTML file, or write a second variant for
  a real A/B comparison. Cheap. No versioning discipline needed beyond the
  filename's own `-v[n]` bump when a round of feedback changes the file
  meaningfully.
- **Shape is settled, now it's about DS fidelity, real component structure,
  interaction correctness against the real DS** → stop here, hand off to
  Skill 4. Skill 4 does not read this HTML file's markup/CSS as a spec — it
  reads the intent statement. This prototype's job was only to get the PM,
  Designer, and Engineer aligned on what to build; if the prototype changed
  the shape of what's being built, update the intent statement first, the
  same one-rule-runs-through-everything discipline as any other stage.

---

## Handoff

**This is the first half of Gate 2 — PM and Design own it jointly.**

**Goes to:**

1. **Designer / UX** — reacts to the HTML file directly (open it, no server
   needed); flags direction changes fast since redoing it costs nothing.
   Decides the screen's shape: layout, flow, which pattern, what's on each
   screen.
2. **PM** — decides whether the shape still matches the problem. Updates the
   intent statement if the prototype changed scope, before Skill 4 begins.
3. **Engineer (Skill 4)** — does not build from this file; builds from the
   (possibly updated) intent statement, using the settled shape as context.
   Decides nothing about shape here — flags anything unbuildable now rather
   than after Skill 4 has produced a real typed component.

**Done when:** all three agree the shape is settled. Shape only — DS fidelity,
real component structure, and interaction correctness are Skill 4's job and are
not reasons to keep iterating here.

**Back into the intent:** the PM writes any shape change into §Scope and
§Navigation model and bumps the version *before* Skill 4 runs. Skill 4 reads
the intent, not this HTML — a shape change that lives only in the prototype
markup will not reach the real component.

---

## Reference example

`docs/mocks/mock-bucket-scope-collection-intent-v2.3-mock-v1.html` and its
sibling screen files in the same directory — the real prototype this feature
went through before Skill 4 rebuilt it as real components.
