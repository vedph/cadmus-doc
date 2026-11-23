---
title: "2026-11-23 History in Editor"
layout: default
parent: History
nav_order: 11
---

# History in Editor

Starting from these versions, items and parts history in the editor is available to Cadmus editors:

- backend: 18.1.x;
- frontend: 20.1.x (for affected libraries only).

To enable history in editor, in **backend** just update all Cadmus packages. In **frontend**:

1. **add packages**:

    ```sh
    pnpm i @myrmidon/cadmus-ui-history @myrmidon/cadmus-ui-diff @myrmidon/timeline-view jsondiffpatch
    ```

2. **update packages**.
3. in your app **routes**, add the item history route, and wrap the routes of each parts library (yours too) with `withHistoryRoutes`, loading the history library lazily with them (so that it is not included in the main bundle):

    ```ts
    export const routes: Routes = [
        // ...
        {
        path: 'items/:id/history',
        loadComponent: () =>
            import('@myrmidon/cadmus-ui-history').then(
            (module) => module.ItemHistoryPageComponent,
            ),
        canActivate: [jwtGuard],
        },
        // for each parts library:
        {
        path: 'items/:iid/general',
        loadChildren: () =>
            Promise.all([
            import('@myrmidon/cadmus-part-general-pg'),
            import('@myrmidon/cadmus-ui-history'),
            ]).then(([module, history]) =>
            history.withHistoryRoutes(module.CADMUS_PART_GENERAL_PG_ROUTES),
            ),
        canActivate: [jwtGuard],
        },
        // ...
    ];
    ```

4. **opt-in in history**: set `window.__env.historyEnabled = true`; in your `env.js`.
