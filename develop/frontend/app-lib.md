---
title: "Creating Frontend Libraries"
parent: "Developing Frontend"
layout: default
nav_order: 2
---

- [Creating Frontend Libraries](#creating-frontend-libraries)
  - [Adding Libraries](#adding-libraries)
    - [Single-Component Libraries](#single-component-libraries)
    - [Multiple-Components Approach](#multiple-components-approach)
    - [Routes in the App](#routes-in-the-app)
    - [Setup](#setup)

# Creating Frontend Libraries

## Adding Libraries

When creating libraries for parts and fragments, you can use different approaches according to the desired level of granularity:

- when you plan for **reuse**, creating a [single library](#single-component-libraries) for each component (part/fragment editor) is the best choice, unless your parts/fragments can be considered so related among themselves that they can be implemented in the same library.
- when implementing **project-specific** editors that will probably never be reused outside of your project, or **domain-specific** sets of editors, you can go with a [multiple-components approach](#multiple-components-approach) and implement all of them in a single library.

In both cases, editors are standalone components, and each page wrapper (the component hosting an editor in its own page, see [creating parts](app-parts)) gets its own route. Routes are grouped in a `Routes` array exported by a library, which the app loads lazily under a route prefix (e.g. `general` for the general parts library).

### Single-Component Libraries

In the single-component approach you create a single library for each editor, containing both the editor UI and its page wrapper.

Also, optionally a `-pg` library will be created for all the single-components libraries which go together in a project.

▶️ (1) Add the library to the app workspace:

```sh
ng g library @myrmidon/cadmus-part-__PRJ__-__NAME__ --prefix cadmus
```

> Example name: `cadmus-part-itinera-cod-loci`. Use `-fr-` instead of `-part-` for fragments.

In each single-component library you will have:

- the part/fragment model (e.g. a file `cod-loci-part.ts`).
- the part/fragment editor UI (e.g. a folder with the `cod-loci-part` editor component).
- the part/fragment editor wrapper (e.g. a folder with the `cod-loci-part-feature` editor wrapper).

All of them are exported from the library's `public-api.ts` barrel file.

▶️ (2) Optionally, you might want to provide another library for all of your components in the project with the routes to the editor wrappers (e.g. `cadmus-part-itinera-pg`). This library just imports all the single-component libraries, and exports their routes from a file like `cadmus-part-__PRJ__-pg.routes.ts`:

```ts
import { Routes } from '@angular/router';

import { pendingChangesGuard } from '@myrmidon/cadmus-core';

// parts from your libraries
import {
  CodBindingsPartFeatureComponent,
  COD_BINDINGS_PART_TYPEID,
} from '@myrmidon/cadmus-part-codicology-bindings';
import {
  CodContentsPartFeatureComponent,
  COD_CONTENTS_PART_TYPEID,
} from '@myrmidon/cadmus-part-codicology-contents';
import {
  CodDecorationsPartFeatureComponent,
  COD_DECORATIONS_PART_TYPEID,
} from '@myrmidon/cadmus-part-codicology-decorations';
// ...etc.

export const CADMUS_PART_CODICOLOGY_PG_ROUTES: Routes = [
  {
    path: `${COD_BINDINGS_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodBindingsPartFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
  {
    path: `${COD_CONTENTS_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodContentsPartFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
  {
    path: `${COD_DECORATIONS_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodDecorationsPartFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
  // ... etc.
];
```

Then export the routes from the library's `public-api.ts`:

```ts
export * from './lib/cadmus-part-codicology-pg.routes';
```

### Multiple-Components Approach

This approach for multiple-components libraries is used in the core part/fragment [shell](https://github.com/vedph/cadmus-shell-v3/tree/master) libraries like general and philology.

▶️ (1) Create two libraries: one with the editors UI, and another for their page wrappers. Conventionally, these libraries are suffixed with `-ui` and `-pg` respectively, e.g.:

```sh
ng g library @myrmidon/cadmus-__PRJ__-part-ui --prefix cadmus
ng g library @myrmidon/cadmus-__PRJ__-part-pg --prefix cadmus
```

The **UI** library is a standard Angular library with a set of standalone components: the part/fragment editors and their child editors, and the part/fragment models. Export all of them from its `public-api.ts`.

The **PG** library contains the page wrappers, one for each part/fragment editor, and a file with their routes (e.g. `cadmus-__PRJ__-part-pg.routes.ts`), exported from its `public-api.ts` as above. This is the routes file of the general parts library (abridged):

```ts
import { Routes } from '@angular/router';

import { pendingChangesGuard } from '@myrmidon/cadmus-core';
import {
  BIBLIOGRAPHY_PART_TYPEID,
  CATEGORIES_PART_TYPEID,
  COMMENT_PART_TYPEID,
  COMMENT_FRAGMENT_TYPEID,
  // ...
} from '@myrmidon/cadmus-part-general-ui';

import { BibliographyPartFeatureComponent } from './bibliography-part-feature/bibliography-part-feature.component';
import { CategoriesPartFeatureComponent } from './categories-part-feature/categories-part-feature.component';
import { CommentPartFeatureComponent } from './comment-part-feature/comment-part-feature.component';
import { CommentFragmentFeatureComponent } from './comment-fragment-feature/comment-fragment-feature.component';
// ...

export const CADMUS_PART_GENERAL_PG_ROUTES: Routes = [
  // parts: <part type ID>/<part ID>
  {
    path: `${BIBLIOGRAPHY_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: BibliographyPartFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
  {
    path: `${CATEGORIES_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CategoriesPartFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
  {
    path: `${COMMENT_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CommentPartFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
  // ...
  // fragments: fragment/<layer part ID>/<fragment type ID>/<location>
  {
    path: `fragment/:pid/${COMMENT_FRAGMENT_TYPEID}/:loc`,
    pathMatch: 'full',
    component: CommentFragmentFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
];
```

Each route uses `pendingChangesGuard` (from `@myrmidon/cadmus-core`), which asks the user to confirm before leaving an editor with unsaved changes. The class-based `PendingChangesGuard` still exists for legacy code, but new code should use the function.

### Routes in the App

▶️ In the app's `app.routes.ts`, load the routes of each PG library lazily, under the route prefix of its editors:

```ts
import { jwtGuard } from '@myrmidon/auth-jwt-login';

export const routes: Routes = [
  // ...
  // cadmus - parts
  {
    path: 'items/:iid/general',
    loadChildren: () =>
      import('@myrmidon/cadmus-part-general-pg').then(
        (module) => module.CADMUS_PART_GENERAL_PG_ROUTES,
      ),
    canActivate: [jwtGuard],
  },
  {
    path: 'items/:iid/__PRJ__',
    loadChildren: () =>
      import('@myrmidon/cadmus-__PRJ__-part-pg').then(
        (module) => module.CADMUS___PRJ___PART_PG_ROUTES,
      ),
    canActivate: [jwtGuard],
  },
  // ...
];
```

The last segment of the path (here `general` or `__PRJ__`) is the editor key used in the app's `part-editor-keys.ts` to map each part/fragment type ID to the library which edits it (see [creating parts](app-parts)).

### Setup

Once you create a library, whatever its type:

▶️ (1) remove the stub code files added by Angular CLI (the sample component and its spec), and their exports from `public-api.ts`.

▶️ (2) take the time for adding more metadata to its `package.json` file, e.g. (replace `__PRJ__` with your project's ID). In `peerDependencies` add all the packages your library code imports, at least the Cadmus ones, like in this example:

```json
{
  "name": "@myrmidon/cadmus-__PRJ__-__NAME__",
  "version": "0.0.1",
  "description": "Cadmus - __PRJ__ ...",
  "keywords": [
    "Cadmus",
    "__PRJ__"
  ],
  "homepage": "https://github.com/vedph/cadmus-__PRJ__-app",
  "repository": {
    "type": "git",
    "url": "https://github.com/vedph/cadmus-__PRJ__-app"
  },
  "author": {
    "name": "Daniele Fusi"
  },
  "peerDependencies": {
    "@angular/common": "^22.2.1",
    "@angular/core": "^22.2.1",
    "@angular/material": "^22.2.1",
    "@myrmidon/ngx-tools": "^3.0.2",
    "@myrmidon/ngx-mat-tools": "^3.0.1",
    "@myrmidon/cadmus-core": "^20.0.0",
    "@myrmidon/cadmus-state": "^20.0.0",
    "@myrmidon/cadmus-ui": "^20.0.0"
  },
  "dependencies": {
    "tslib": "^2.3.0"
  }
}
```

Use the versions installed in your workspace (see its root `package.json`). A PG library also needs `@myrmidon/cadmus-api`, `@myrmidon/cadmus-item-editor`, and its UI library.

💡 Packages listed in `dependencies` (rather than `peerDependencies`) are bundled with your library, and ng-packagr refuses to build unless you allow them in the library's `ng-package.json`. For instance, a library rendering Markdown with `marked` (see [Monaco](monaco)) has:

```json
{
  "allowedNonPeerDependencies": ["marked"]
}
```

▶️ (3) in your app's `package.json` file, for each added library you can add the corresponding **build commands** to `scripts` (to be run like `npm run <SCRIPTNAME>`), e.g.:

```json
{
  "build-ass": "ng build @myrmidon/cadmus-refs-assertion",
  "build-blk": "ng build @myrmidon/cadmus-text-block-view",
  "build-lib": "npm run build-ass && npm run build-blk"
}
```

The `build-lib` command is used to build all the libraries in the workspace. Be sure to enumerate them in their order of _dependencies_ when writing the command.

⚠️ Remember that in the workspace both the app and the libraries use each library from its build output (the workspace's `tsconfig.json` maps each library name to its folder under `dist`). So, a change to a library reaches the app and the other libraries only after you rebuild that library and every library depending on it, in dependency order.

💡 The [shell workspace](https://github.com/vedph/cadmus-shell-v3/tree/master) has a script (`scripts/build-libs.mjs`, run with `pnpm build:libs [lib...]`) which finds the dependencies among the libraries, and builds the specified libraries and all those depending on them (or all of them) in the right order. You can copy it into your workspace.
