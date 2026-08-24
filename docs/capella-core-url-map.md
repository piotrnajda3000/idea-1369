# Capella Core URL Map

Generated from real source: `src/routing/routes.tsx` (top-level, react-router-dom v6
data router) + `src/routing/definitions/*.tsx` (9 domain files, re-exported via
`src/routing/definitions/index.ts`). Every leaf route maps 1:1 to a named export in
`src/pages/**` via `lazy: () => import('pages/...').then((m) => ({ Component: ... }))`.

Root structure (`src/routing/routes.tsx`):
```tsx
export const routes: RouteObject[] = [
  {
    errorElement: (<RouterLinkProvider><TopLevelErrorBoundary /></RouterLinkProvider>),
    path: '/',
    children: [
      { index: true, loader: /* redirects "/" to /databases or /sign-in */ },
      { element: <RootUnauthorised />, children: [...AUTH_ROUTES] },
      { path: '/diagnostics', element: <UseUrlManagerProvider><Diagnostics /></UseUrlManagerProvider> },
      {
        loader: /* JWT check, resolves organizationId (oid), redirects */,
        lazy: () => import('root-authorised').then((m) => ({ Component: m.RootAuthorised })),
        children: [
          ...ORGANIZATION_ROUTES, ...DATABASE_ROUTES, ...APP_ENDPOINT_ROUTES,
          ...APP_SERVICE_ROUTES, ...COLUMNAR_ROUTES, ...PLAYGROUND_ROUTES,
          ...PROJECT_ROUTES, ...SETUP_ROUTES, ...USER_SETTINGS_ROUTES, ...PRODUCT_HUB,
          { path: '*', element: <NotFoundPage resourceType={NotFoundResourceType.PAGE} /> },
        ],
      },
    ],
  },
];
```

---

## Auth routes (unauthenticated — `auth.tsx`, rendered under `<RootUnauthorised />`)

| Path | Notes |
|---|---|
| `/challenge` | |
| `/lost-mfa` | |
| `/lost-device/:token` | |
| `/confirm-Email/:token` | |
| `/confirm-email` | |
| `/accept-invite/:token` | |
| `/enterprise-sso` | |
| `/sso-callback` | |
| `/social-sign-in-callback-google` | |
| `/social-sign-in-callback-github` | |
| `/sign-in` | |
| `/signed-out` | |
| `/sign-up` | |
| `/setup-account/ex2/social-login` | |
| `/setup-account/ex2/social-verify` | |
| `/setup-account/ex2/social-verify-mfa` | |
| `/forgot-password` | |
| `/password-recovery/:token` | |
| `/password-expired` | |

---

## Organization routes (`organization.tsx`)

| Path | Notes |
|---|---|
| `/dashboard` | |
| `/no-membership` | |
| `people` → `view`, `edit` | |
| `teams` → `create`, `settings/view`, `settings/members`, `settings/organization-roles`, `settings/project-roles` | |
| `billing` → `reporting`, `alerts/create`, `alerts/edit`, `saved-cards` | |
| `/projects` | |
| `/invite-people` | |
| `/databases` | |
| `/columnar` | |
| `/app-services` → `guided-setup` | |
| `/support` → `create`, `view` | |
| `/settings` → `sizing/new-estimate`, `api-keys/create`, `api-keys/view`, `sso/create/saml`, `sso/create/oidc`, `sso/view/saml`, `sso/view/oidc`, `sso/listusers`, `mfa`, `session`, `activity/view`, `upgrade` | |
| `/card` → `add`, `edit` | |
| `/community` | |

---

## Database routes (`database.tsx`) — Template A, `LayoutDatabase`

Base: `/database` (element wraps `BreadCrumbsNav` + `LayoutDatabase`)

| Path (relative to `/database`) | Notes |
|---|---|
| `connect` (+5 sub-tools) | |
| `buckets` (+ `create`, `view`) | |
| `ai-functions` (+ `examples`, `associate-model/create`) | |
| `quick-start` | |
| `monitoring` → `overview`, `metrics-explorer`, `health-advisor`, `data-overview`, `index-overview`, `query-overview`, `node-overview` | wraps `LayoutDatabaseMonitoring` (adds `MonitoringNavigation` sidebar); `overview` = `pages/database/monitoring/cluster-overview/cluster-overview.tsx` (`ClusterOverviewPage`) |
| `backup` → `restores`, `buckets`; also `bucket/details`, `bucket/completed`, `bucket/restore`, `bucket/downloadable` | route array has a duplicate top-level `backup` entry — flag if consolidating |
| `datatools` → `analytics`, `import`, `query`, `documents`, `indexes`, `vector-indexes` (+`create`/`edit`), `eventing` (+`edit`/`create`/`function-editor`/`logs`), `search` (+`create`/`edit`/`create-old`/`edit-old`/`aliases/create`/`aliases/edit`) | |
| `settings` → `plan`, `services`, `express-scaling`, `nodes`, `replications` (+`setup`/`view`/`selfmanaged create+edit`), `maintenance` (+`completed`/`view`/`support`), `schedule` (+`edit`), `internet` (+`create`), `vpc` (+`create`/`view`), `private-endpoint` (+`create`), `access-control` (+`accounts`/`view`/`create`/`password`, `roles`), `security-certificates`, `activity` (+`view`), `health-advisor` | |

---

## Project routes (`project.tsx`) — Template A, `LayoutProject`

Base: `/project`

| Path | Notes |
|---|---|
| `databases` | |
| `columnar` | |
| `app-services` | |
| `collaborators` (+`add`/`view`/`edit`) | RBAC-heavy — see `view-collaborators.tsx`, `edit-collaborators.tsx` |
| `backup/cluster-backup` | |
| `alerts` (+`add`/`edit`/`list`) | |
| `create-database`, `create-free-database`, `create-default-database`, `create-columnar` | |
| `upgrade` | |
| `card/add` | |
| `create-app-service`, `create-app-service-guided-setup` | |
| `settings/activity` (+`view`) | |
| `settings/api-keys` (+`create`/`view`) | |

---

## Columnar routes (`columnar.tsx`) — Template A, `LayoutColumnar`

Base: `/columnar`

| Path | Notes |
|---|---|
| `workbench` | |
| `settings/plan` | |
| `settings/access-control/accounts` (+`view`/`create`), `settings/access-control/roles` (+`view`/`create`) | |
| `settings/maintenance` (+`completed`/`view`/`support`) | |
| `settings/schedule` (+`edit`) | |
| `settings/internet` (+`create`) | |
| `settings/activity` (+`view`) | |
| `settings/vpc` (+`create`/`view`) | |
| `settings/private-endpoint` (+`create`) | |
| `settings/connection` | |
| `settings/security-certificates` | |
| `backup` (+`restores`) | |
| `monitoring` | |

---

## App Service routes (`app-service.tsx`) — Template A, `LayoutAppService`

Base: `app-service`

| Path | Notes |
|---|---|
| `home` | |
| `app-endpoints` | |
| `create-app-endpoint` | |
| `monitoring` | |
| `settings/configuration` | |
| `settings/maintenance` (+`completed`/`view`/`support`) | |
| `settings/log-streaming` | |
| `settings/admin-credentials` (+`create`/`edit`) | |
| `settings/allowed-ip-addresses` (+`add`) | |
| `settings/private-endpoints` (+`add`) | |

---

## App Endpoint routes (`app-endpoint.tsx`) — Template A, `LayoutAppEndpoint*`

Base: `/app-endpoint`

| Path | Notes |
|---|---|
| `home` | |
| `security/app-users` (+`create`/`edit`) | |
| `security/app-roles` (+`create`/`edit`) | free-form custom roles — separate from `OrganizationRole`/`ProjectRole` |
| `security/auth-providers` (+`add-provider`) | |
| `monitoring` | |
| `connect/admin-rest`, `connect/public-rest` | |
| `settings/resources` | |
| `settings/delta-sync` | |
| `settings/import-filter` (+`config`) | |
| `settings/configuration` | |
| `settings/log-streaming` | |

---

## Playground routes (`playground.tsx`)

| Path |
|---|
| `/playground` |
| `/playground/:path/:exampleId` |

---

## Product Hub routes (`product-hub.tsx`)

Base: `/product-hub`

| Path | Notes |
|---|---|
| `workflows` (+`status`, `workflows/manage`) | |
| `integrations` → `openai`, `amazon-s3`, `bedrock`, `microsoft-foundry` | |
| `ai-functions` (+`examples`/`associate-model create`) | |
| `agent-tracer` | |
| `ai-library` → `tools`, `prompts` (+view each) | |
| `models` (+`deploy`/`edit`) | |
| `access-control` | |
| `offerings` → `rag-application` (vectorize/guided-flow/search/create(+v2/v2-unstructured)), `agentic-app`, `auto-embedding` (workflow/create(+v2)), `ai-agent`, `spark-app`, `semantic-search` (+guided-flow/search) | |
| `private-endpoints` (+`region/:region`) | |
| `monitoring` | |
| `*` | catch-all within product-hub |

---

## User Settings routes (`user-settings.tsx`) — Template B, `LayoutSettings`

Base: `/user-settings`

| Path | Notes |
|---|---|
| `/user-settings` | + `region`, `notifications`, `change-password`, `mfa` all redirect back to `/user-settings` |
| `resources/invitations` | |
| `resources/organizations` | |
| `activity` (+`view`) | |

---

## Setup routes (`setup.tsx`) — chrome-less

| Path | Notes |
|---|---|
| `/setup-account/ex2` (index only) | → `pages/setup-account/create-account`; no layout wrapper, no `BreadCrumbsNav` |

---

## Notes / gaps

- The `database.tsx` route array has what looks like a **duplicate top-level `backup`
  entry** — worth confirming with the owning team before treating both as canonical.
- Component file locations follow `pages/<matching-route-segment>/...` — when adding
  a new leaf route, put the page under the matching `src/pages/<domain>/...` path and
  register it with a `lazy()` import in the relevant `src/routing/definitions/<domain>.tsx`.
