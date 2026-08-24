# Reference Example — Bucket · Scope · Collection

This example shows the output of this skill applied to the
Bucket/Scope/Collection feature for Couchbase Capella.

## Where to find the full example output

The complete deliverables from this example session are in the outputs folder:

| File | Description |
|---|---|
| `intent-bucket-scope-collection-v2.md` | Complete intent statement (Skill 1 output) |
| `design_brief_prompts.html` | 6 design brief prompts with live inputs (Skill 2) |
| `capella_bsc_from_intent.html` | Full interactive mock built from the intent (Skill 3) |
| `capella_ux_iteration_ab.html` | Concept A vs B comparison (Skill 3 iteration) |
| `CLAUDE.md` | Repo instruction set used in the code generation session (Skill 4) |
| `claude_code_workflow.md` | Step-by-step Claude Code session transcript (Skill 4) |

## Key decisions this example illustrates

- Navigation model: 3 screens, Template C-flat + Template B
- RBAC: 4 roles with distinct experiences, absent not disabled
- Scale: 30 buckets / 20 scopes / 50 collections — search + pagination + lazy load
- Dangerous default: backup inverted from "Do Not Backup" to "Set Weekly Schedule"
- Immutable fields: Bucket Name, Bucket Type, Conflict Resolution
- Feature boundary: backup at creation → this feature. Post-creation → Backup & Restore intent.

Use this example to calibrate the depth and format of output
expected from the skill when applied to a new feature.
