---
title: "Creating Frontend Fragments"
parent: "Developing Frontend"
layout: default
nav_order: 10
---

## Creating Frontend Fragments

Adding a fragment is very similar to [adding a part](app-parts). Cadmus component [libraries](app-lib) may include both parts and fragments, and are created in the same way.

## 1. Add Fragment Model

▶️ (1) add the fragment _model_ (derived from `Fragment`), its type ID constant, and its JSON schema constant to `<fragment>.ts` (e.g. `comment-fragment.ts`). You can use a template like this (replace `__NAME__` with your part's name, e.g. `Comment`, adjusting case where required):

```ts
import { Fragment } from "@myrmidon/cadmus-core";

/**
 * The __NAME__ layer fragment server model.
 */
export interface __NAME__Fragment extends Fragment {
  // TODO: add properties
}

export const __NAME___FRAGMENT_TYPEID = "fr.it.vedph.__PRJ__.__NAME__";

export const __NAME___FRAGMENT_SCHEMA = {
  definitions: {},
  $schema: "http://json-schema.org/draft-07/schema#",
  $id:
    "www.vedph.it/cadmus/fragments/<PRJ>/" +
    __NAME___FRAGMENT_TYPEID +
    ".json",
  type: "object",
  title: "__NAME__Fragment",
  // TODO: add which properties are required
  required: ["location"],
  properties: {
    location: {
      $id: "#/properties/location",
      type: "string",
    },
    baseText: {
      $id: "#/properties/baseText",
      type: "string",
    },
    // TODO: add properties
  },
};
```

💡 If you want to infer a schema in the [JSON schema tool](https://jsonschema.net/), which is usually the quickest way of writing the schema, you can use this JSON template adding your model's properties to it:

```json
{
  "location": "1.2",
  "baseText": "abc",
  "TODO": "add properties here"
}
```

▶️ (2) add the export for the new file to the library's "barrel" file `public-api.ts`, e.g. `export * from './lib/<NAME>';`.

>If your editor needs to be customized with specific settings, you can add them to the backend JSON profile and retrieve them in the editor's code. To this end, inject the `AppRepository` service and request the setting object for the editor of the part/fragment type ID and role via its `getSettingFor(typeId, roleId?)` method. This will return an object with any model, representing all the settings for that specific editor.

## 2. Add Fragment Editor

▶️ (1) add a _fragment editor dumb component_ named after the fragment (e.g. `ng g component comment-fragment` for `CommentFragmentComponent` after `CommentFragment`), and extending `ModelEditorComponentBase<T>` where `T` is the fragment's type.

- 📁 code template:

```ts
import { Component, OnInit } from '@angular/core';
import {
  CloseSaveButtonsComponent,
  EditedObject,
  ModelEditorComponentBase,
} from '@myrmidon/cadmus-ui';
import {
  FormControl,
  FormBuilder,
  Validators,
  UntypedFormGroup,
  FormGroup,
  FormsModule,
  ReactiveFormsModule,
} from '@angular/forms';

import {
  MatCard,
  MatCardHeader,
  MatCardAvatar,
  MatCardTitle,
  MatCardSubtitle,
  MatCardContent,
  MatCardActions,
} from '@angular/material/card';
import { MatIcon } from '@angular/material/icon';
import { MatFormField, MatLabel, MatError } from '@angular/material/form-field';
import { MatInput } from '@angular/material/input';
import { MatSelect } from '@angular/material/select';
import { MatOption } from '@angular/material/core';
// ... etc.

import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { ThesauriSet, ThesaurusEntry } from '@myrmidon/cadmus-core';

import { __NAME__Fragment } from "../__NAME__-fragment";

/**
 * __NAME__ fragment editor component.
 * Thesauri: TODO...
 */
@Component({
  selector: "cadmus-__NAME__-fragment",
  imports: [
    FormsModule,
    ReactiveFormsModule,
    MatCard,
    MatCardHeader,
    MatCardAvatar,
    MatIcon,
    MatCardTitle,
    MatCardSubtitle,
    MatCardContent,
    MatFormField,
    MatLabel,
    MatInput,
    MatError,
    MatSelect,
    MatOption,
    MatCardActions,
    // ... etc.
    CloseSaveButtonsComponent,
  ],
  templateUrl: "./__NAME__-fragment.component.html",
  styleUrls: ["./__NAME__-fragment.component.scss"],
})
export class __NAME__FragmentComponent
  extends ModelEditorComponentBase<__NAME__Fragment>
  implements OnInit {
  // TODO: add form controls

  // TODO: add tag entries if required, e.g.:
  // public tagEntries: ThesaurusEntry[] | undefined;

  constructor(authService: AuthJwtService, formBuilder: FormBuilder) {
    super(authService, formBuilder);
    // form
    // TODO: instantiate your form's controls
  }

  public override ngOnInit(): void {
    super.ngOnInit();
  }

  protected buildForm(formBuilder: FormBuilder): FormGroup | UntypedFormGroup {
    return formBuilder.group({
      // TODO: add controls instantiated in ctor, e.g.:
      // tag: this.tag,      
    });
  }

  private updateThesauri(thesauri: ThesauriSet): void {
    // TODO: set thesaurus entries
    // const key = 'comment-tags';
    // if (this.hasThesaurus(key)) {
    //   this.tagEntries = thesauri[key].entries;
    // } else {
    //   this.tagEntries = undefined;
    // }
  }

  private updateForm(fr?: __NAME__Fragment | null): void {
    if (!fr) {
      this.form.reset();  
      return;
    }
    // TODO: set form controls from model, e.g.:
    // this.tag.setValue(fr.tag);
    this.form.markAsPristine();
  }

  protected override onDataSet(data?: EditedObject<__NAME__Fragment>): void {
    // thesauri
    if (data?.thesauri) {
      this.updateThesauri(data.thesauri);
    }

    // form
    this.updateForm(data?.value);
  }

  protected getValue(): __NAME__Fragment {
    const fr = this.getEditedFragment() as __NAME__Fragment;
    // TODO: set fragment's properties from form controls, e.g.:
    // fr.standard = this.standard.value.trim();
    return fr;
  }
}
```

- 📁 HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>textsms</mat-icon>
      </div>
      <mat-card-title>__NAME__ Fragment {{ data?.value?.location }}</mat-card-title>
      <mat-card-subtitle> {{ data?.baseText }} </mat-card-subtitle>
    </mat-card-header>

    <mat-card-content> TODO: add controls </mat-card-content>

    <mat-card-actions>
      <cadmus-close-save-buttons
        [form]="form"
        [noSave]="userLevel < 2"
        (closeRequest)="close()"
      />
    </mat-card-actions>
  </mat-card>
</form>
```

▶️ (2) remember to add the component to the exports barrel file `public-api.ts`.

## 3. Add PG Editor Wrapper

▶️ (1) under your library's `src/lib` folder, add a **fragment editor feature component** named after the part (e.g. `ng g component comment-fragment-feature` for `CommentFragmentFeatureComponent` after `CommentFragment`).

- 📁 editor wrapper code:

```ts
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';

import { MatSnackBar } from '@angular/material/snack-bar';

import { LibraryRouteService } from '@myrmidon/cadmus-core';
import {
  EditFragmentFeatureBase,
  FragmentEditorService,
} from '@myrmidon/cadmus-state';
import { CurrentItemBarComponent } from '@myrmidon/cadmus-ui-pg';
import { DecoratedTokenTextComponent } from '@myrmidon/cadmus-ui';
import { ApparatusFragmentComponent } from '@myrmidon/cadmus-part-philology-ui';

@Component({
  selector: 'cadmus-__NAME__-part-feature',
  imports: [
    CurrentItemBarComponent,
    DecoratedTokenTextComponent,
    ApparatusFragmentComponent,
  ],
  templateUrl: './note-__NAME__-feature.component.html',
  styleUrls: ['./note-__NAME__-feature.component.scss'],
})
export class __NAME__PartFeatureComponent
  extends EditFragmentFeatureBase
  implements OnInit
{
  constructor(
    router: Router,
    route: ActivatedRoute,
    snackbar: MatSnackBar,
    editorService: FragmentEditorService,
    libraryRouteService: LibraryRouteService
  ) {
    super(router, route, snackbar, editorService, libraryRouteService);
  }

  protected override getReqThesauriIds(): string[] {
    // TODO: if role-dependent thesauri are required, add:
    // this.roleIdInThesauri = true;

    // TODO: return the IDs of all the thesauri required by the wrapped editor, e.g.:
    return ['note-tags'];
    // or just avoid overriding the function if no thesaurus required
  }
}
```

💡 If you need to display or use the portion of text selected for the fragment being edited:

- inject `TextLayerService` in the editor class constructor (e.g. `private _layerService: TextLayerService`);
- in `onDataSet`, get the text like in this example, where we are updating a `public frText?: string` property with it:

```ts
// get fragment's text into frText
if (data?.baseText && data.value) {
  this.frText = this._layerService.getTextFragment(
    data.baseText,
    TokenLocation.parse(data.value.location)!
  );
}
```

- 📁 editor wrapper HTML template:

```html
<cadmus-current-item-bar/>
<div class="base-text">
  <cadmus-decorated-token-text
    [baseText]="data?.baseText || ''"
    [locations]="frLoc ? [frLoc] : []"
  />
</div>
<cadmus-__NAME__-fragment
  [identity]="identity"
  [data]="$any(data)"
  (dataChange)="save($event)"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
/>
```

▶️ (2) ensure that this component is in the exports barrel `public-api.ts` file.

## 3. Add Sub-Route

This is optional, and is required only when you are providing a library with a module containing sub-routes to a set of related PG editor wrapper components.

▶️ (1) add the corresponding **route** in the PG library's module of the project's PG library, e.g.:

```ts
{
  path: `fragment/:pid/${__NAME___FRAGMENT_TYPEID}/:loc`,
  pathMatch: "full",
  component: __NAME__FragmentFeatureComponent,
  canDeactivate: [PendingChangesGuard],
},
```

## 4. Add Fragment Mapping to App

▶️ (1) in your app's project `part-editor-keys.ts`, add the mapping for the fragment just created under the layer part key, like e.g.:

```ts
// layer parts
[TOKEN_TEXT_LAYER_PART_TYPEID]: {
  part: GENERAL,
  fragments: {
    // each fragment type in the layer part is a property
    [COMMENT_FRAGMENT_TYPEID]: GENERAL,
    [APPARATUS_FRAGMENT_TYPEID]: PHILOLOGY,
    [LING_TAGS_FRAGMENT_TYPEID]: TGR_GR,
  },
},
```

Here, the type ID of the fragment (from its model in the "ui" library) is mapped to the route prefix constant `TGR_GR` = `tgr-gr`, which is the root route to the "pg" library module for the app.
