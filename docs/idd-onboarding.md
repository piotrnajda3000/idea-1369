# Getting Started with IDD — Your First Intent

This is a practical walkthrough, not the pitch. If you haven't read the one-pager
yet ([docs/idd-one-pager_claudecode.md](idd-one-pager_claudecode.md)), skim it
first for the "why." This doc is the "how," for the first time you actually run it.

Reference example throughout: the bucket/scope/collection feature, at
`docs/intents/active/intent-bucket-scope-collection-v2.3.md` plus its design
brief and three mocks in `docs/mocks/`. Open that intent file side by side with
this doc — it's a real, messy, corrected-in-place example of what "good" looks
like, including the mistakes that got caught and fixed.

---

## 0. Nothing to install — but read this before your first session

If you can open this repo in Claude Code, you already have everything:

- `CLAUDE.md` at the repo root — read automatically at session start, tells
  Claude the RBAC model, design system rules, and the checklist it must run
  before writing any code.
- `docs/capella-ds-index.json`, `docs/capella-core-ia.md`,
  `docs/capella-core-url-map.md` — the grounding truth (design tokens, layout
  patterns, route map). These aren't specific to buckets — they're the
  foundation for any Capella UI feature.
- `.claude/commands/idd-*.md` and/or `.claude/skills/idd-*/SKILL.md` — the
  slash commands (`/idd-intent`, `/idd-brief`, `/idd-ui-component`, `/idd-code`,
  `/idd-archive`, `/idd-status` — **hyphens, not colons**; if you see `/idd:intent`
  written anywhere, including earlier in this repo's own docs, that's stale —
  the real command name matches the filename).
- `docs/idd-skills/` — the actual instructions each slash command runs.

You don't build any of this — but *where* you launch `claude` from, and whether
you accept a one-time prompt, both matter more than the content of any of these
files. Nearly every setup problem we hit doing a from-scratch run was
environmental, not a content bug. Concretely:

**Launch from the monorepo root**, not from inside `cmd/cp-ui-v3`. Confirmed by
directly testing both: the command definitions that actually work assume you're
sitting at the repo root when you launch. If you `cd` into `cmd/cp-ui-v3` first
and launch from there, some commands may still appear to work but others will
silently fail to find files, because the two command mechanisms in this repo
(`.claude/commands/` and `.claude/skills/`) don't always agree on which
directory paths are relative to.

**Bring the whole checkout, not a hand-picked folder.** If you're setting
someone up fresh (a new clone, or copying files for a colleague), don't cherry-pick
just the `docs/` files — the slash commands live in `.claude/` and depend on
`docs/idd-skills/` and `docs/capella-*` all being present *together*, at the
same relative paths. A partial copy reliably produces "Unknown command" with no
other symptom.

**Watch for a one-time trust prompt.** The first time Claude Code opens a
directory it hasn't seen before, it asks you to confirm you trust the folder.
Until you accept that, it won't load `.claude/commands` or `.claude/skills` at
all — and there's no error message pointing at this; commands just don't exist.
If your commands aren't registering and the directory is otherwise correct,
this is the most likely cause. Fully quit and relaunch `claude` fresh in that
exact directory and watch closely for the prompt.

**Sanity-check with a non-interactive call before you invest time.** Before
running a real session, confirm the setup is live:

```bash
claude --print "/idd-status"
```

This is read-only, fast, and — because `--print` mode skips the interactive
trust prompt — it tells you definitively whether the command *files* are
correct, separately from whether your interactive session has been trusted
yet. If this works but your interactive session still shows no commands, the
problem is the trust prompt, not the setup.

---

## 1. Start with a real problem, not a solution

`/idd-intent` wants a *problem statement*, not a feature spec. Bad input: "add a
dropdown to pick a scope." Good input (the real one we used): "users can't
create scopes and collections from the Buckets tab — they have to go to the
limited Data Tools view instead."

Run it:

```bash
claude
```

Then in the session:

```
/idd-intent "<your problem, in one or two sentences>"
```

Claude will read the three grounding docs and draft an intent statement into
`docs/intents/active/intent-<your-feature>-v1.0.md`. Expect it to ask you
clarifying questions if your problem statement is ambiguous about scope or
RBAC — answer those, don't let it guess.

**This is Gate 1, and it's yours to own.** Read the whole file before saying yes.
Check specifically:
- §Who/RBAC — does it match who should actually see this?
- §Scope — does it match what you actually meant, not a bigger or smaller thing?
- Any "open question" it flagged — resolve it in the chat, don't leave it open.

If something in the intent needs to change, ask Claude to revise it in place. It
should bump the version (`v1.0` → `v1.1`) and edit the same file — never
regenerate a new one from scratch. That version history is the point.

---

## 2. Design brief

```
/idd-brief intent-<your-feature>-v1.1.md
```

This produces a 6-section brief in chat (flows, RBAC matrix, layout template
choice, edge cases) for you to hand to design, or use yourself if design's
already looped in informally. It's not a new file by default — copy it out if
you want it saved.

## 3. Mock

```
/idd-ui-component intent-<your-feature>-v1.1.md <screen-name>
```

Produces the real presentational component plus a Storybook story — open it in
Storybook, no throwaway HTML step. **Actually click through it.** Don't just
read the code. In our run, two real bugs only showed up by clicking (an
empty-state that never hid the table, an error-simulation flag that got
clobbered by a loading-state transition) — reading the file wouldn't have
caught either.

This is the loop that iterates fastest — expect several rounds here before
anyone touches real code. **This is Gate 2**, owned jointly by you and design.
Don't let it advance to `/idd-code` until you'd actually ship this screen.

## 4. Code generation

```
/idd-code intent-<your-feature>-v1.1.md
```

This is where it gets slower and more literal — Claude has to find and reuse
real hooks, real components, real types, not invent plausible-looking ones.
Expect a gap report at the end listing anything it couldn't ground (a missing
API endpoint, an ambiguous RBAC case) — those are for you and engineering to
resolve, not for Claude to guess at.

**This is Gate 3, and it's engineering's, not yours** — someone needs to
actually run the app, click through the real feature (not the mock), and check
the gap report before this merges. In our run this is where the most serious
bugs surfaced (a wrong ID field, an unhandled promise rejection) — things no
amount of code reading would have caught, only running it did.

---

## What "good" looks like — read the reference example

Open `docs/intents/active/intent-bucket-scope-collection-v2.3.md`. Notice:

- It's one file, versioned in place (`v2.0` → `v2.3`), not five different files.
- Corrections are written back into it as they're found — e.g. the note about
  the grid needing `TanStackDataGrid`, not AG-Grid; the `bucket.id` vs.
  `bucket.name` finding. That's the paper trail — don't delete history when you
  fix something, append it.
- §Who/RBAC lists exact role strings from `src/constants/roles.ts`, not
  paraphrased roles.

If your first intent doesn't look like this yet, that's expected — it gets
there through the same corrections-in-place process, not by getting it right
on the first pass.

---

## Common pitfalls

- **Don't skip clicking the mock.** It's the cheapest place to catch a bug —
  free of a build step, free of engineering time.
- **Don't let Claude invent an API endpoint, variant name, or role string.**
  If it flags something as a gap, that's correct behavior — resolve it, don't
  ask it to guess.
- **Don't treat `/idd-code` output as done.** It's grounded, typed, and
  RBAC-aware, but "compiles" isn't "verified." Someone has to run the app.
- **One file per feature, not a new file per revision.** If you're about to
  create `intent-v2.md` next to `intent-v1.md`, stop — you should be editing
  the same file and bumping its header version instead.
- **A fresh `/idd-code` run doesn't inherit corrections from a previous one.**
  We regenerated this same feature twice in this repo. The second pass
  reintroduced a bug the first pass had already found and fixed (using the
  bucket's Capella ID instead of its cluster-native name for proxy-service
  calls) — because that fix lived in code comments, not in the intent
  statement itself. If a feature has been built before, check whether its
  intent file already documents the hard-won gotchas (search for `[CRITICAL`
  or `[CORRECTED` tags); if it doesn't yet, add them before regenerating.
- **Check the generated code actually landed where the intent said it would.**
  Same regeneration also put the scope/collection UI on the individual
  bucket's edit page instead of the Buckets *list* page the intent's own
  Navigation model specified. It compiled, it worked, and it was still wrong —
  Gate 3 needs to check placement against §Navigation model, not just
  "does it run."

---

## If you get stuck

Ping whoever ran the bucket/scope/collection build (this session) — that intent
file plus its mocks and `tasks.md` are the reference to compare against.

If slash commands themselves are the problem (not the feature content), see
§0 above first — in practice every command-registration issue we hit turned
out to be the launch directory or the trust prompt, not a broken command file.
