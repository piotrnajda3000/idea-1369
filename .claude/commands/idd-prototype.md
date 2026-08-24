# IDD Prototype — Skill 3 (throwaway HTML)

Run IDD Skill 3: Prototype for a Capella feature screen or set of screens.

Read docs/idd-skills/skill-3-prototype/SKILL.md for methodology: this
generates a fast, self-contained, throwaway HTML/CSS mock — a visual
approximation of the design system, not real code — so PM/Designer/UX can
iterate on shape (layout, flow, which pattern) cheaply before any real
component exists. Real precedent: docs/mocks/mock-bucket-scope-collection-intent-v2.3-mock-v1.html.

Arguments: $ARGUMENTS
Format: [intent-filename] [optional: screen-name]
Example: intent-backup-restore-v1.0.md cluster-backups

Find the intent statement at: docs/intents/active/[intent-filename].
Read the design brief for this feature if one exists.
Read docs/capella-ds-index.json for real token values to hand-copy into a
`:root` block — don't invent values.

Before writing the file, output:
Screens / RBAC scope / Destructive ops / Gaps

Then write one self-contained HTML file per screen to docs/mocks/, named
mock-[feature-slug]-v[n]-[screen-name].html:
- Reproduce production chrome (app bar, breadcrumbs, tab nav) well enough to
  read in context
- A `.mock-controls` role switcher covering every role in §Who/RBAC —
  switching hides controls that role doesn't have (absent, not disabled)
- Represent Loading/Empty/Error/Success as togglable controls, not separate
  files
- Represent destructive-confirm flows with the same information the real
  ConfirmDialog would show, without needing to be the real component
- No build step, no TypeScript, no production imports — this is throwaway

After the file(s): a short PM decision note — direction chosen, why (against
§Desired Outcome), and any open questions the prototype surfaced. If the
prototype changed scope, update the intent statement before moving to
`/idd-ui-component` (Skill 4).
