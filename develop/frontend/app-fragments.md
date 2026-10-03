---
title: "Creating Frontend Fragments"
parent: "Developing Frontend"
layout: default
nav_order: 4
---

- [Creating Frontend Fragments](#creating-frontend-fragments)
- [1. Add Fragment Model](#1-add-fragment-model)
- [2. Add Fragment Editor](#2-add-fragment-editor)
- [3. Add PG Editor Wrapper](#3-add-pg-editor-wrapper)
- [3. Add Sub-Route](#3-add-sub-route)
- [4. Add Fragment Mapping to App](#4-add-fragment-mapping-to-app)

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
    "www.vedph.it/cadmus/fragments/<PRJ>/" + __NAME___FRAGMENT_TYPEID + ".json",
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

> If your editor needs to be customized with specific settings, you can add them to the backend JSON profile and load them in the editor with `initSettings`: see the fragment editor template below.

## 2. Add Fragment Editor

A fragment editor works exactly like a part editor: it is a dumb component extending `ModelEditorComponentBase<T>` (from `@myrmidon/cadmus-ui`), and edits the fragment through an Angular **signal form** (`@angular/forms/signals`). Before writing it, read [How a Part Editor Works](app-parts#20-how-a-part-editor-works): its rules (no `<form>`, a value for every draft field, `setFieldFromChild` for child editors, `copyFormValue` for arrays of objects, etc.) apply to fragments too. The only differences are:

- `getValue()` starts from `this.getEditedFragment()`, which keeps the fragment's `location`, instead of `getEditedPart(typeId)`;
- the fragment's location and its portion of the base text are usually displayed in the card's header;
- the type ID is the fragment's one (`fr.` prefix) when loading settings.

▶️ (1) add a _fragment editor dumb component_ named after the fragment (e.g. `ng g component comment-fragment` for `CommentFragmentComponent` after `CommentFragment`), and extending `ModelEditorComponentBase<T>` where `T` is the fragment's type.

In the template, replace `__NAME__` with your fragment's name, in the casing required by each place (e.g. `Comment` in class names, `comment` in file names and selectors, `COMMENT` in constants).

- 📁 code template:

```ts
// __NAME__-fragment.component.ts

import {
  ChangeDetectionStrategy,
  Component,
  computed,
  inject,
  linkedSignal,
} from "@angular/core";
import { TitleCasePipe } from "@angular/common";
import { FormField, maxLength, required } from "@angular/forms/signals";

import {
  MatCard,
  MatCardActions,
  MatCardAvatar,
  MatCardContent,
  MatCardHeader,
  MatCardSubtitle,
  MatCardTitle,
} from "@angular/material/card";
import { MatOption } from "@angular/material/core";
import { MatError, MatFormField, MatLabel } from "@angular/material/form-field";
import { MatIcon } from "@angular/material/icon";
import { MatInput } from "@angular/material/input";
import { MatSelect } from "@angular/material/select";
// ... etc.

import {
  TextLayerService,
  ThesaurusEntry,
  TokenLocation,
} from "@myrmidon/cadmus-core";
import {
  CloseSaveButtonsComponent,
  HelpLinkComponent,
  ModelEditorComponentBase,
} from "@myrmidon/cadmus-ui";

import { __NAME__Fragment } from "../__NAME__-fragment";

/**
 * The editable draft behind the form. Use '' for empty text (native inputs
 * need strings), number | null for numbers, [] for arrays.
 */
interface __NAME__FragmentControls {
  // TODO: replace with your fields
  tag: string;
  text: string;
}

/**
 * Fragment -> draft. Must return a value for each field also when there is
 * no fragment. Copy arrays of objects with copyFormValue (from
 * @myrmidon/cadmus-ui).
 */
function toDraft(fr?: __NAME__Fragment | null): __NAME__FragmentControls {
  return {
    // TODO: replace with your fields
    tag: fr?.tag || "",
    text: fr?.text || "",
  };
}

/**
 * __NAME__ fragment editor component.
 * Thesauri: TODO list of thesauri IDs, e.g. __NAME__-tags (optional).
 */
@Component({
  selector: "cadmus-__NAME__-fragment",
  imports: [
    FormField,
    TitleCasePipe,
    MatCard,
    MatCardActions,
    MatCardAvatar,
    MatCardContent,
    MatCardHeader,
    MatCardSubtitle,
    MatCardTitle,
    MatError,
    MatFormField,
    MatIcon,
    MatInput,
    MatLabel,
    MatOption,
    MatSelect,
    // ... etc.
    // cadmus
    CloseSaveButtonsComponent,
    HelpLinkComponent,
  ],
  templateUrl: "./__NAME__-fragment.component.html",
  styleUrl: "./__NAME__-fragment.component.scss",
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class __NAME__FragmentComponent extends ModelEditorComponentBase<__NAME__Fragment> {
  private readonly _layerService = inject(TextLayerService);

  /**
   * The portion of the base text this fragment refers to.
   */
  public readonly frText = computed<string | undefined>(() => {
    const data = this.data();
    return data?.baseText && data.value
      ? this._layerService.getTextFragment(
          data.baseText,
          TokenLocation.parse(data.value.location)!,
        )
      : undefined;
  });

  // thesauri (TODO: replace with yours):
  // __NAME__-tags
  public readonly tagEntries = computed<ThesaurusEntry[] | undefined>(
    () => this.data()?.thesauri?.["__NAME__-tags"]?.entries,
  );

  // the draft is rebuilt from each new data
  private readonly _draft = linkedSignal(() => toDraft(this.data()?.value));
  public readonly form = this.createForm(this._draft, (p) => {
    // TODO: replace with your rules
    maxLength(p.tag, 100);
    required(p.text);
    maxLength(p.text, 1000);
  });

  // OPTIONAL: settings, loaded by the fragment's type ID, e.g.:
  // private readonly _settings = signal<__NAME__FragmentSettings | undefined>(undefined);
  // constructor() {
  //   super();
  //   this.initSettings<__NAME__FragmentSettings>(
  //     __NAME___FRAGMENT_TYPEID,
  //     (s) => this._settings.set(s),
  //   );
  // }

  // EXAMPLE: handler of a child editor's output. Always use setFieldFromChild
  // (from @myrmidon/cadmus-ui), so that opening the fragment does not make it
  // dirty:
  // public onDateChange(date: HistoricalDateModel): void {
  //   setFieldFromChild(this.form.date, date);
  // }

  // draft -> fragment
  protected getValue(): __NAME__Fragment {
    const fr = this.getEditedFragment() as __NAME__Fragment;
    const draft = this._draft();
    // TODO: replace with your fields
    fr.tag = draft.tag.trim() || undefined;
    fr.text = draft.text.trim();
    return fr;
  }
}
```

- 📁 HTML template:

```html
<!-- __NAME__-fragment.component.html -->
<!-- no <form> here: the fragment is saved by the save button -->
<mat-card appearance="outlined">
  <mat-card-header>
    <div mat-card-avatar>
      <mat-icon>textsms</mat-icon>
    </div>
    <mat-card-title>
      {{ (modelName() | titlecase) || "__NAME__ Fragment" }} {{
      data()?.value?.location }}
    </mat-card-title>
    <mat-card-subtitle>{{ frText() }}</mat-card-subtitle>
    <cadmus-help-link [url]="helpUrl()" />
  </mat-card-header>

  <mat-card-content>
    <!-- TODO: replace with your controls -->

    <!-- tag (bound to thesaurus) -->
    @if (tagEntries()?.length) {
    <mat-form-field>
      <mat-label>tag</mat-label>
      <mat-select [formField]="form.tag">
        <mat-option [value]="''">(none)</mat-option>
        @for (e of tagEntries(); track e.id) {
        <mat-option [value]="e.id">{{ e.value }}</mat-option>
        }
      </mat-select>
    </mat-form-field>
    }
    <!-- tag (free) -->
    @else {
    <mat-form-field>
      <mat-label>tag</mat-label>
      <input matInput [formField]="form.tag" />
      @if ( form.tag().getError("maxLength") && (form.tag().dirty() ||
      form.tag().touched()) ) {
      <mat-error>tag too long</mat-error>
      }
    </mat-form-field>
    }

    <!-- text -->
    <div>
      <mat-form-field class="long-text">
        <mat-label>text</mat-label>
        <textarea matInput [formField]="form.text"></textarea>
        @if ( form.text().getError("required") && (form.text().dirty() ||
        form.text().touched()) ) {
        <mat-error>text required</mat-error>
        } @if ( form.text().getError("maxLength") && (form.text().dirty() ||
        form.text().touched()) ) {
        <mat-error>text too long</mat-error>
        }
      </mat-form-field>
    </div>
  </mat-card-content>

  <mat-card-actions>
    <cadmus-close-save-buttons
      [form]="form"
      [noSave]="userLevel < 2"
      (closeRequest)="close()"
      (saveRequest)="save()"
    />
  </mat-card-actions>
</mat-card>
```

💡 If the fragment's model is a list of entries, follow the [list part editor template](app-parts#22-list-part-editor-template), using `getEditedFragment()` in `getValue()`, and the [entry editor template](app-parts#list-entry-editor-template) for its entries.

▶️ (2) remember to add the component to the exports barrel file `public-api.ts`.

## 3. Add PG Editor Wrapper

▶️ (1) under your library's `src/lib` folder, add a **fragment editor feature component** named after the part (e.g. `ng g component comment-fragment-feature` for `CommentFragmentFeatureComponent` after `CommentFragment`).

- 📁 editor wrapper code:

```ts
import { Component, OnInit } from "@angular/core";
import { Router, ActivatedRoute } from "@angular/router";

import { MatSnackBar } from "@angular/material/snack-bar";

import { LibraryRouteService } from "@myrmidon/cadmus-core";
import {
  EditFragmentFeatureBase,
  FragmentEditorService,
} from "@myrmidon/cadmus-state";
import { CurrentItemBarComponent } from "@myrmidon/cadmus-item-editor";
import { DecoratedTokenTextComponent } from "@myrmidon/cadmus-ui";
import { ApparatusFragmentComponent } from "@myrmidon/cadmus-part-philology-ui";

@Component({
  selector: "cadmus-__NAME__-part-feature",
  imports: [
    CurrentItemBarComponent,
    DecoratedTokenTextComponent,
    ApparatusFragmentComponent,
  ],
  templateUrl: "./note-__NAME__-feature.component.html",
  styleUrls: ["./note-__NAME__-feature.component.scss"],
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
    libraryRouteService: LibraryRouteService,
  ) {
    super(router, route, snackbar, editorService, libraryRouteService);
  }

  protected override getReqThesauriIds(): string[] {
    // TODO: if role-dependent thesauri are required, add:
    // this.roleIdInThesauri = true;

    // TODO: return the IDs of all the thesauri required by the wrapped editor, e.g.:
    return ["note-tags"];
    // or just avoid overriding the function if no thesaurus required
  }
}
```

💡 If you need to display or use the portion of text selected for the fragment being edited, do it in the fragment editor, with the `frText` computed signal shown in its template above.

- 📁 editor wrapper HTML template:

```html
<cadmus-current-item-bar />
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
