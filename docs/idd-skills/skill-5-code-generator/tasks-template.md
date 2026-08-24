# tasks.md — [Feature Name]
**Intent:** docs/intents/[intent-filename].md
**Generated:** [date]
**Status:** In progress

---

## Implementation tasks

Generated from §Scope of the intent statement.
Engineer checks off each item as it is completed and verified.

### Hook / state management
- [ ] [hook name] — [what it manages]
- [ ] Lazy fetch: [entity] fetched on [trigger]
- [ ] Search/filter: [scope]
- [ ] Pagination: PAGE_SIZE=[n]

### Components
- [ ] [ComponentA] — [what it renders, which roles see it]
- [ ] [ComponentB] — [what it renders, which roles see it]
- [ ] [ComponentC] — [what it renders, which roles see it]
- [ ] DeleteConfirmModal — typed confirmation, never backdrop-dismiss
- [ ] [FormComponent] — [fields, immutable badges, dangerous defaults]

### RBAC wiring
- [ ] Confirm real role string values from auth context
- [ ] Verify all absent controls use conditional render — not disabled prop
- [ ] Test all 4 role views: Org Owner / Project Owner / Data Writer / Data Reader

### API integration
- [ ] [endpoint 1] — [component that calls it]
- [ ] [endpoint 2] — [component that calls it]
- [ ] Error states wired for all API calls
- [ ] Loading skeletons wired for all async fetches

### Routing
- [ ] Confirm [route] exists in router
- [ ] Add [new route] if flagged as gap
- [ ] Back navigation / breadcrumbs correct

### Gap resolution (from Skill 5 gap report)
- [ ] [Gap 1]: [action needed] — owner: [PM/Design/Engineering]
- [ ] [Gap 2]: [action needed] — owner: [PM/Design/Engineering]

### Design alignment
- [ ] All CSS tokens match real variable names from tokens file
- [ ] Spacing and density matches existing screen
- [ ] Component matches approved mock: [mock filename]

### Quality gate
- [ ] No `any` types
- [ ] No `disabled` prop used for access control
- [ ] All destructive actions: typed modal, never backdrop-dismiss
- [ ] Dangerous defaults correct per §Constraints
- [ ] Skeleton + empty + error + success states all present

---

## Archive checklist (run when feature ships)
- [ ] Move intent to docs/intents/archive/[date]-[feature-slug]/
- [ ] Update capella-core-url-map.md with any new routes
- [ ] Update capella-ds-index.json if new tokens were added
- [ ] PR description links to intent statement
- [ ] Close open questions that were resolved during build
