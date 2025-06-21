---
title: "Creating Frontend Libraries"
parent: "Developing Frontend"
layout: default
nav_order: 8
---

- [Creating Frontend Libraries](#creating-frontend-libraries)
  - [Adding Libraries](#adding-libraries)
    - [Single-Component Libraries](#single-component-libraries)
    - [Legacy Approach](#legacy-approach)
    - [Setup](#setup)

# Creating Frontend Libraries

## Adding Libraries

When creating libraries parts and fragments, you can use different approaches according to the desired level of granularity:

- when you plan for **reuse**, creating a [single library](#single-component-libraries) for each component (part/fragment editor) is the best choice, unless your parts/fragments can be considered so related among themselves that they can be implemented in the same library.
- when implementing **project-specific** editors that will probably never be reused outside of your project, or **domain-specific** sets of editors, you can go with a multiple-components approach and implement all of them in a single library. In this case the library will typically have a module with the sub-routes for each targeted component in it, as you design a sub-route for each target component.

### Single-Component Libraries

In the single-component approach you create a single library for each editor, containing both the editor UI and its page wrapper.

Also, optionally a `-pg` library will be created for all the single-components libraries which go together in a project.

▶️ (1) Add the library to the app workspace:

```sh
ng g library @myrmidon/cadmus-part-__PRJ__-__NAME__ --prefix cadmus
```

>Example name: `cadmus-part-itinera-cod-loci`. Use `-fr-` instead of `-part-` for fragments.

In each single-component library you will have:

- the part/fragment model (e.g. a file `cod-loci-part`).
- the part/fragment editor UI (e.g. a folder with the `cod-loci-part` editor component).
- the part/fragment editor wrapper (e.g. a folder with the `cod-loci-part-feature` editor wrapper).

▶️ (2) Optionally, you might want to provide another library for all of your components in the project with sub-routes to the editor wrappers (e.g. `cadmus-part-itinera-pg`). This will include only a module, which imports all the single component libraries, and exports routes like e.g.:

```ts
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { RouterModule } from '@angular/router';

// cadmus
import { PendingChangesGuard } from '@myrmidon/cadmus-core';

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

// sub-routes
export const RouterModuleForChild = RouterModule.forChild([
  {
    path: `${COD_BINDINGS_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodBindingsPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  {
    path: `${COD_CONTENTS_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodContentsPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  {
    path: `${COD_DECORATIONS_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodDecorationsPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  // ... etc.
]);

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    // Cadmus
    RouterModuleForChild,
  ],
  exports: [],
})
export class CadmusPartCodicologyPgModule {}
```

### Legacy Approach

This approach for multiple-components libraries is more a legacy and is used in the core part/fragment [shell](https://github.com/vedph/cadmus-shell-v3/tree/master) libraries like general and philology.

▶️ (1) Create two libraries: one with the editors UI, and another for their page wrappers. Conventionally, these libraries are suffixed with `-ui` and `-pg` respectively, e.g.:

```sh
ng g library @myrmidon/cadmus-PRJ-part-ui --prefix cadmus
ng g library @myrmidon/cadmus-PRJ-part-pg --prefix cadmus
```

The UI library is just a standard Angular library with a bunch of components in it. Once you have added all of your imports, you just have to include your part/fragment editors there, and export their components.

The PG library is a standard Angular library with some added routes, one for each part included in the library. In its **module** add code like this:

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule, FormsModule } from '@angular/forms';
import { RouterModule } from '@angular/router';

import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatExpansionModule } from '@angular/material/expansion';
import { MatInputModule } from '@angular/material/input';
import { MatIconModule } from '@angular/material/icon';
import { MatProgressBarModule } from '@angular/material/progress-bar';
import { MatProgressSpinnerModule } from '@angular/material/progress-spinner';
import { MatSelectModule } from '@angular/material/select';
import { MatTooltipModule } from '@angular/material/tooltip';
import { MatToolbarModule } from '@angular/material/toolbar';

import { PendingChangesGuard } from '@myrmidon/cadmus-core';
import {
  BIBLIOGRAPHY_PART_TYPEID,
  CATEGORIES_PART_TYPEID,
  CHRONOTOPES_PART_TYPEID,
  // ...
} from '@myrmidon/cadmus-part-general-ui';

import { BibliographyPartFeatureComponent } from './bibliography-part-feature/bibliography-part-feature.component';
import { CategoriesPartFeatureComponent } from './categories-part-feature/categories-part-feature.component';
import { ChronologyFragmentFeatureComponent } from './chronology-fragment-feature/chronology-fragment-feature.component';
// ...

// https://github.com/ng-packagr/ng-packagr/issues/778
export const RouterModuleForChild = RouterModule.forChild([
  {
    path: `${BIBLIOGRAPHY_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: BibliographyPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  {
    path: `${CATEGORIES_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CategoriesPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  {
    path: `${COMMENT_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CommentPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  // ...
  // for fragments:
  // {
  //   path: `fragment/:pid/${COMMENT_FRAGMENT_TYPEID}/:loc`,
  //   pathMatch: 'full',
  //   component: CommentFragmentFeatureComponent,
  //   canDeactivate: [PendingChangesGuard],
  // },
]);

@NgModule({
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModuleForChild,
    // material
    MatButtonModule,
    MatCheckboxModule,
    MatCardModule,
    MatExpansionModule,
    MatIconModule,
    MatInputModule,
    MatProgressBarModule,
    MatProgressSpinnerModule,
    MatSelectModule,
    MatTooltipModule,
    MatToolbarModule,
    // cadmus
    BibliographyPartFeatureComponent,
    CategoriesPartFeatureComponent,
    ChronologyFragmentFeatureComponent,
    // ...
  ],
  exports: [
    BibliographyPartFeatureComponent,
    CategoriesPartFeatureComponent,
    ChronologyFragmentFeatureComponent,
    // ...
  ],
})
export class CadmusPartGeneralPgModule {}
```

### Setup

Once you create a library, whatever its type:

(1) remove the stub code files added by Angular CLI: the sample component and its service. Also, take the time for adding more metadata to its `package.json` file, e.g. (replace `__PRJ__` with your project's ID). In `peerDependencies` you should add at least the Cadmus peer dependencies, like in this example:

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
    "@angular/common": "^18.0.0",
    "@angular/core": "^18.0.0",
    "@myrmidon/ngx-tools": "^2.0.0",
    "@myrmidon/cadmus-core": "^12.0.0",
    "@myrmidon/cadmus-state": "^13.0.0",
    "@myrmidon/cadmus-ui": "^13.0.0"
  },
  "dependencies": {
    "tslib": "^2.3.0"
  }
}
```

(2) in your app's `package.json` file, to speed up builds, for each added library you can add the corresponding **build commands** to `package.json` scripts (to be run like `npm run <SCRIPTNAME>`), e.g.:

```json
{
  "build-ass": "ng build @myrmidon/cadmus-refs-assertion",
  "build-blk": "ng build @myrmidon/cadmus-text-block-view",
  "build-lib": "npm run-script build-ass && npm run-script build-blk"
}
```

The `build-lib` command is used to build all the libraries in the workspace. Be sure to enumerate them in their order of _dependencies_ when writing the command.
