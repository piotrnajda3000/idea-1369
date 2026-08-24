# IDD UI Component — Skill 4

Run IDD Skill 4: UI Component for a Capella feature screen or component.

Read docs/idd-skills/skill-4-ui-component/SKILL.md for methodology: this
generates real, presentational React components and a co-located Storybook
story file — not a mock or approximation of any kind, this is the real
component that ships.

This assumes the feature already went through Skill 3 (Prototype) and the
screen's shape is settled — if it hasn't, run `/idd-prototype` first. This
skill builds the real thing directly from the intent statement; it does not
rebuild from the Skill 3 HTML prototype.

Arguments: $ARGUMENTS
Format: [intent-filename] [component-name]
Example: intent-backup-restore-v1.0.md cluster-backups

Find the intent statement at: docs/intents/active/[intent-filename]
Find the existing sibling component(s) this feature extends under
src/pages/** and match its directory/naming convention exactly.

Before writing any code, output the header:
Component path / Route / RBAC scope / Reused DS components / Destructive ops / Gaps

Then write:
1. The presentational component (real imports, typed props, no hooks, no
   data fetching — data and callbacks arrive as props only)
2. Its `*.stories.tsx` file, using this repo's real fixture utilities
   (mocks/permission, mocks/utils/wrap-with-rbac, mocks/responses/*) — one
   story per RBAC role with a distinct view, plus Empty/Loading/Error
   stories and a destructive-confirm-open story if applicable

Follow all constraints from §Constraints in the intent statement.
Destructive actions use the real ConfirmDialog (confirmationValue set,
enableOutsideClickClose unset), never a placeholder modal.

No filename versioning — edit these files in place on later rounds; git diff
is the change history, same discipline as the intent statement itself.

After the files: output the state transitions table (Action | Result | Condition).
