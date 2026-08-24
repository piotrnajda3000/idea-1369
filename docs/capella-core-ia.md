# Capella Core IA — Layout & Chrome Patterns

Generated from real source in `cp-ui-v3`. Two distinct page templates exist. The
"chrome" (dark top nav + cluster context bar + tabs) is **not** a single reusable
component — it's assembled compositionally across three layers: global app shell,
per-route element, and a `Screen`-wrapping layout component.

---

## Template A — Cluster-context shell (dominant pattern)

Used by: `database.tsx`, `columnar.tsx`, `app-service.tsx`, `app-endpoint.tsx`,
`project.tsx`, most of `organization.tsx` and `product-hub.tsx`.

### Layer 1 — Global dark nav (rendered once for all authenticated routes)
File: `src/root-authorised.tsx`
```tsx
<BoundTrialBar />
<AppBar isResourceCenterEnabled />
<Outlet />
```
`AppBar` styling: `src/components/app-bar/app-bar-ui/app-bar-ui.tsx`
```tsx
<header data-auto-id="app-bar-wrapper"
  className="relative z-50 flex flex-col gap-x-4xl gap-y-md bg-on-background-app-bar
             fill-background px-3xl py-xl text-on-background-decoration ...">
```

### Layer 2 — Cluster/project/org context bar (per top-level route)
Added in each `src/routing/definitions/*.tsx` file, not inside a shared layout:
```tsx
element: (
  <>
    <BreadCrumbsNav />
    <LayoutDatabase>
      <Outlet />
    </LayoutDatabase>
  </>
),
```
Component: `src/components/navigation/breadcrumbs-nav/breadcrumbs-nav.tsx`
```
className="box-border flex min-h-[40px] flex-wrap items-center justify-between
           gap-md border-b border-on-background-decoration bg-panel-queue px-3xl py-xs"
```

### Layer 3 — Tab strip + content shell
Base component: `src/components/screen/screen/screen.tsx`
```tsx
export function Screen({ renderSlotGutter = true, fullWidth = false, children, tabBar, fullHeight = false }: ScreenProps) {
  return (
    <>
      {tabBar}
      <TransientNotificationBar />
      <div className={clsx('grow self-stretch', fullHeight && 'flex flex-col')}>
        <main id="main-content"
          className={clsx(!fullHeight && 'mx-auto', renderSlotGutter && 'px-gutter', !fullWidth && 'xl:w-3/4', fullHeight && 'flex grow flex-col')}>
          {children}
        </main>
      </div>
    </>
  );
}
```

Per-domain layout wrappers (`src/layouts/layout-*/`) plug a domain-specific tab
navigation component into `Screen`'s `tabBar` prop:

| Domain | Layout component | Tab nav component |
|---|---|---|
| Database | `LayoutDatabase` (`layouts/layout-database`) | `DatabaseTabNavigation` |
| Organization | `LayoutOrganization` (`layouts/layout-organization`) | `OrganizationTabNavigation` |
| Project | `LayoutProject` (`layouts/layout-project`) | `ProjectTabNavigation` |
| Columnar | `LayoutColumnar` | Columnar tab nav |
| App Service | `LayoutAppService` | App Service tab nav |
| App Endpoint | `LayoutAppEndpoint*` | App Endpoint tab nav |

Example (`src/layouts/layout-database/layout-database.tsx`):
```tsx
return (
  <Screen
    fullHeight={fullHeight}
    fullWidth={fullWidth}
    tabBar={
      <DatabaseTabNavigation
        quickStartPath={quickStartPath}
        dataToolsPath={createURL('/database/datatools', params)}
        appServicesPath={getAppServicePath()}
        connectPath={createURL('/database/connect', params)}
        bucketsPath={createURL('/database/buckets', params)}
        monitoringPath={createURL('/database/monitoring', params)}
        backupPath={canReadClusterBackup ? createURL('/database/backup', params) : ''}
        settingsPath={createURL('/database/settings', params)}
        isServerlessDatabase={serverlessDatabase}
        aiServicesPath={createURL('/database/ai-functions', params)}
        isDb800={isDb800}
      />
    }
  >
    <DatabaseJobsBar dataAutoId="progress" />
    {children}
  </Screen>
);
```
Tab bar rendering primitive: `TabBarMenu` (`src/components/tab-bar/tab-bar-menu`) —
`DatabaseTabNavigation` builds an `items` array (Home / Data Tools / App Services /
Connect / Buckets / Monitoring / Backup / AI Functions / Settings) and renders
`<TabBarMenu items={...} dataAutoId="main-tab-bar" />`.

Some sub-sections nest a **secondary left sidebar** inside the tab content, e.g.
`src/layouts/layout-database-monitoring/layout-database-monitoring.tsx` renders
`<MonitoringNavigation links={databaseMonitoringLinks} />` beside `{children}`.

### Full render tree example
Route: `/database/monitoring/overview` →
```
RootAuthorised (AppBar)
  → BreadCrumbsNav
    → LayoutDatabase (Screen + DatabaseTabNavigation + DatabaseJobsBar)
      → LayoutDatabaseMonitoring (MonitoringNavigation sidebar)
        → ClusterOverviewPage  (src/pages/database/monitoring/cluster-overview/cluster-overview.tsx)
```
The leaf page component (`ClusterOverviewPage`) only renders its own content
(charts/tiles) — it does **not** import `AppBar`, `BreadCrumbsNav`, or any tab nav.
All chrome is supplied by router + layout composition, wired in
`src/routing/definitions/database.tsx`.

**When generating a new Template-A screen:** create the leaf page component only
(content, no chrome imports), then wire it into the existing domain's
`src/routing/definitions/<domain>.tsx` route array under the existing `LayoutX`
element, and add a tab entry to the relevant `*TabNavigation` component if it needs
its own tab.

---

## Template B — Simple sidebar layout (no cluster context bar, no tab strip)

Used by: `user-settings.tsx` (`LayoutSettings`).

Example: `src/layouts/layout-settings/layout-settings.tsx`
```tsx
export function LayoutSettings({ children }: LayoutSettingsProps) {
  return (
    <div className="md:flex md:items-start md:justify-start">
      <div className="md:w-2/6 md:shrink-0 md:grow-0 md:basis-1/6">
        <div className="my-5xl">
          <SettingsNavigation
            generalHref={...}
            invitationsHref={...}
            organizationsHref={...}
            activityHref={...}
          />
        </div>
      </div>
      <div className="md:w-4/6 md:shrink-0 md:grow-0 md:basis-5/6">
        <PageContainer>{children}</PageContainer>
      </div>
    </div>
  );
}
```
Distinguishing features vs. Template A: left `SettingsNavigation` sidebar instead
of a top tab strip, `PageContainer` instead of `Screen`, no `BreadCrumbsNav` in this
route's chrome.

`setup.tsx` (`/setup-account/ex2`, the account-creation wizard) has **no** layout
wrapper and **no** `BreadCrumbsNav` at all — a genuinely chrome-less route, distinct
from both templates. Treat any full-screen wizard/onboarding flow the same way.

---

## Decision rule for new features

- Extending an existing cluster/project/org/columnar/app-service/app-endpoint
  screen with tabs and cluster context → **Template A**. Add the page under the
  existing domain's `LayoutX`, add a route in `src/routing/definitions/<domain>.tsx`.
- A user-level settings page (not cluster/project scoped) → **Template B**
  (`LayoutSettings` + `SettingsNavigation`).
- A standalone wizard/onboarding flow with no persistent nav → no layout wrapper,
  follow `setup.tsx`'s pattern.
