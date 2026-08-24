# IDD Archive

Archive a shipped Capella feature.

Feature slug: $ARGUMENTS

Steps:
1. Find docs/intents/active/intent-$ARGUMENTS-*.md
2. Find tasks-$ARGUMENTS.md in the codebase
3. Create folder: docs/intents/archive/[YYYY-MM-DD]-$ARGUMENTS/
4. Move intent statement and tasks.md into the archive folder
5. Check capella-core-url-map.md — prompt to add any new routes this feature introduced
6. List any open questions from the intent statement that were never resolved
7. Print archive summary

Commit message to suggest:
chore(idd): archive $ARGUMENTS intent
Intent archived: docs/intents/archive/[date]-$ARGUMENTS/
