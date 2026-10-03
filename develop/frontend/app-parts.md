---
title: "Creating Frontend Parts"
parent: "Developing Frontend"
layout: default
nav_order: 3
---

- [Creating Frontend Parts](#creating-frontend-parts)
  - [1. Add Part Model](#1-add-part-model)
  - [2. Add Part Editor](#2-add-part-editor)
    - [2.1. How a Part Editor Works](#21-how-a-part-editor-works)
    - [2.2. Generic Part Editor Template](#22-generic-part-editor-template)
    - [2.3. List Part Editor Template](#23-list-part-editor-template)
  - [3. Add PG Editor Wrapper](#3-add-pg-editor-wrapper)
  - [4. Add Sub-Route](#4-add-sub-route)
  - [5. Add Part Mapping to App](#5-add-part-mapping-to-app)

# Creating Frontend Parts

## 1. Add Part Model

▶️ (1) in your library, under `src/lib`, add the **part model** file, named `<PARTNAME>.ts` (e.g. `cod-bindings-part.ts`), like this:

```ts
import { Part } from "@myrmidon/cadmus-core";

/**
 * The __NAME__ part model.
 */
export interface __NAME__Part extends Part {
  // TODO: add properties
}

/**
 * The type ID used to identify the __NAME__Part type.
 */
export const __NAME___PART_TYPEID = "it.vedph.__PRJ__.__NAME__";

/**
 * JSON schema for the __NAME__ part.
 * You can use the JSON schema tool at https://jsonschema.net/.
 */
export const __NAME___PART_SCHEMA = {
  $schema: "http://json-schema.org/draft-07/schema#",
  $id:
    "www.vedph.it/cadmus/parts/__PRJ__/__LIB__/" +
    __NAME___PART_TYPEID +
    ".json",
  type: "object",
  title: "__NAME__Part",
  required: [
    "id",
    "itemId",
    "typeId",
    "timeCreated",
    "creatorId",
    "timeModified",
    "userId",
    // TODO: add other required properties here...
  ],
  properties: {
    timeCreated: {
      type: "string",
      pattern: "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$",
    },
    creatorId: {
      type: "string",
    },
    timeModified: {
      type: "string",
      pattern: "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$",
    },
    userId: {
      type: "string",
    },
    id: {
      type: "string",
      pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$",
    },
    itemId: {
      type: "string",
      pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$",
    },
    typeId: {
      type: "string",
      pattern: "^[a-z][-0-9a-z._]*$",
    },
    roleId: {
      type: ["string", "null"],
      pattern: "^([a-z][-0-9a-z._]*)?$",
    },

    // TODO: add properties and fill the "required" array as needed
  },
};
```

💡 If you want to infer a schema in the [JSON schema tool](https://jsonschema.net/), which is usually the quickest way of writing the schema, you can use this JSON template, adding your model's properties to it:

```json
{
  "id": "009dcbd9-b1f1-4dc2-845d-1d9c88c83269",
  "itemId": "2c2eadb7-1972-4415-9a43-b8036b6fa685",
  "typeId": "it.vedph.thetype",
  "roleId": null,
  "timeCreated": "2019-11-29T16:48:49.694Z",
  "creatorId": "zeus",
  "timeModified": "2019-11-29T16:48:49.694Z",
  "userId": "zeus",
  "TODO": "add properties here"
}
```

▶️ (2) add the new file to the exports of the "barrel" `public-api.ts` file in the module, like `export * from './lib/<NAME>-part';`.

## 2. Add Part Editor

The part editor is a dumb component which edits the part's model through an Angular **signal form** (`@angular/forms/signals`). Its data is converted into an editable _draft_ when loading, and the draft is converted back into the part's model when saving.

▶️ (1) in `src/lib`, add a **part editor dumb component** named after the part (e.g. `ng g component note-part` for `NotePartComponent` after the model `NotePart`), extending `ModelEditorComponentBase<T>` (from `@myrmidon/cadmus-ui`), where `T` is the part's type. There are usually two cases, each with its own template below:

- a generic part (2.1);
- a part whose model is just a list of entries (2.2).

In the templates, replace `__NAME__` with your model's name, in the casing required by each place (e.g. `CodBinding` in class names, `cod-binding` in file names and selectors, `COD_BINDING` in constants), and `__PRJ__` with your project's prefix.

### 2.1. How a Part Editor Works

Every part editor has the same three pieces:

1. a **draft** interface (`...Controls`), the editable shape behind the form, with a pure `toDraft(part)` function mapping the part into it;
2. the **draft signal and the form**: `_draft = linkedSignal(() => toDraft(this.data()?.value))` and `form = this.createForm(this._draft, schema)`. The draft is rebuilt whenever new data is bound;
3. `getValue()`, which builds the part from the draft when saving.

`ModelEditorComponentBase` does everything else, so do not reimplement it:

| member                                          | what it does                                                                                          |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `data` (model), `identity`, `disabled` (inputs) | the edited part with its thesauri, the part's identity, and a flag disabling the whole form.          |
| `editorClose`, `dirtyChange` (outputs)          | emitted on close, and when the dirty state changes.                                                   |
| `isDirty`, `modelName`, `helpUrl`               | signals: dirty state, human-friendly part name, URL of the help page if any.                          |
| `userLevel`                                     | the current user's level (0-4). Use `[noSave]="userLevel < 2"` on the save button.                    |
| `createForm(draft, schema?)`                    | creates the form. The whole form is disabled while `disabled` is true.                                |
| `save()`, `close()`                             | save (only when the form is valid, else it marks it as touched to show errors), and request to close. |
| `getEditedPart(typeId)`                         | a copy of the edited part, or a new part when creating it. Start `getValue()` from it.                |
| `initSettings(typeId, callback)`                | loads the editor's settings from the profile (call it in the constructor).                            |
| `onDataSet(data)`                               | override only if you must react to new data beyond the draft. Usually you don't need it.              |

Whenever new data is bound or saved, the base class resets the form's interaction state (dirty, touched). So never call `form().reset()` when data arrives, nor set field values from an `effect`.

**Rules to follow** (each of them prevents a bug found in real editors):

- ⚠️ **no `<form>` element** in the template. Bind each control with `[formField]="form.x"`, and save through `(saveRequest)="save()"` on `<cadmus-close-save-buttons>`. A `<form>` would make Enter in any input of any child widget save the whole part.
- ⚠️ **the draft must have a value for every field**, also when there is no part (new part). Use `''` for empty text (native inputs need strings, so a `string | null` field fails template type-checking), `number | null` for numeric inputs, `[]` for arrays. Fields bound only to Material selects and checkboxes, or to child editors, may be `null`. In `getValue()`, trim strings and save empty optional values as `undefined`.
- ⚠️ **child editor outputs go through `setFieldFromChild`**. Many child editors (e.g. doc references, proper names, asserted IDs, historical dates) emit a normalized copy of their value right after receiving it (e.g. `undefined` for a `null`). If your handler sets the field and calls `markAsDirty()`, the editor becomes dirty as soon as it opens, and the user gets a "pending changes" prompt when closing it without having changed anything. So write every handler of a child's `(xxxChange)` output as `setFieldFromChild(this.form.x, value)`. Instead, handlers of the user's own actions (add, delete, move an entry, pick an item) set the value and call `markAsDirty()` directly.
- ⚠️ **copy arrays of objects with `copyFormValue()`** when they enter the draft (`toDraft`), when they leave it (`getValue`), and when they come from a child editor. The form tags every object in its arrays with a hidden identity `Symbol`, and object spread (`{ ...x }`) copies it. Never copy objects from the form with spread. Class instances whose methods you need are an exception: keep them as they are.
- **thesauri** are `computed()` signals over `this.data()?.thesauri?.['thesaurus-id']?.entries`. When a thesaurus is optional, offer a free text input as a fallback for the select.
- **settings** come from `initSettings()`, and may arrive after the data. If the draft depends on them, keep them in a signal and read it in the `linkedSignal`.
- **validation** is declared in the form's schema function: `required`, `maxLength`, `minLength`, `min`, `max`, `pattern`, `validate` (custom), `applyEach` (array items), `disabled` (conditional disabling), all from `@angular/forms/signals`. Error kinds are camel case (`getError('maxLength')`, not `maxlength`). `required()` does not flag an empty array: use `NgxToolsSignalValidators.strictMinLength(p.entries, 1)` from `@myrmidon/ngx-tools`. Do not put validation attributes like `required`, `min` or `max` on `[formField]` elements: Angular rejects them (`NG8022`), so use schema rules, with `{ when: () => ... }` for conditional ones.
- **Monaco editor** (`ngx-monaco-editor`): it reports text set by code as a user change. Do not bind it with `[formField]`. Use `[value]="form.text().value()"`, `(valueChange)="setFieldFromEditor(form.text, $event)"` and `(blur)="form.text().markAsTouched()"`, exposing the helper on the component with `public readonly setFieldFromEditor = setFieldFromEditor;`.
- use `ChangeDetectionStrategy.OnPush`, and in templates read signals (`form.x().value()`, `edited()`), never plain properties which change later.
- if you override `ngOnInit`, call `super.ngOnInit()`.

The helpers `copyFormValue`, `setFieldFromChild`, `sameFormValue`, `setFieldFromEditor` and `isImplicitSubmission` are all exported by `@myrmidon/cadmus-ui`.

### 2.2. Generic Part Editor Template

This template shows the most common cases: a text field, a field bound to an optional thesaurus, and a child editor (here doc references, from `@myrmidon/cadmus-refs-doc-references`). Replace them with your own fields.

▶️ (1) write code and HTML template:

- 📁 part editor code:

```ts
// __NAME__-part.component.ts

import {
  ChangeDetectionStrategy,
  Component,
  computed,
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
  MatCardTitle,
} from "@angular/material/card";
import { MatOption } from "@angular/material/core";
import { MatError, MatFormField, MatLabel } from "@angular/material/form-field";
import { MatIcon } from "@angular/material/icon";
import { MatInput } from "@angular/material/input";
import { MatSelect } from "@angular/material/select";
// ... etc.

import { ThesaurusEntry } from "@myrmidon/cadmus-core";
import {
  CloseSaveButtonsComponent,
  HelpLinkComponent,
  ModelEditorComponentBase,
  copyFormValue,
  setFieldFromChild,
} from "@myrmidon/cadmus-ui";
// EXAMPLE: a child editor; remove if not used
import {
  DocReference,
  DocReferencesComponent,
} from "@myrmidon/cadmus-refs-doc-references";

import { __NAME__Part, __NAME___PART_TYPEID } from "../__NAME__-part";

/**
 * The editable draft behind the form. Use '' for empty text (native inputs
 * need strings), number | null for numbers, [] for arrays.
 */
interface __NAME__PartControls {
  // TODO: replace with your fields
  tag: string;
  text: string;
  references: DocReference[];
}

/**
 * Part -> draft. Must return a value for each field also when there is no
 * part. Copy arrays of objects with copyFormValue.
 */
function toDraft(part?: __NAME__Part | null): __NAME__PartControls {
  return {
    // TODO: replace with your fields
    tag: part?.tag || "",
    text: part?.text || "",
    references: copyFormValue(part?.references || []),
  };
}

// OPTIONAL: the settings for this editor, from the profile
// export interface __NAME__PartSettings {
//   // TODO: settings properties
// }

/**
 * __NAME__ part editor component.
 * Thesauri: TODO list of thesauri IDs, e.g. __NAME__-tags (optional),
 * doc-reference-types, doc-reference-tags (optional).
 */
@Component({
  selector: "cadmus-__NAME__-part",
  imports: [
    FormField,
    TitleCasePipe,
    MatCard,
    MatCardActions,
    MatCardAvatar,
    MatCardContent,
    MatCardHeader,
    MatCardTitle,
    MatError,
    MatFormField,
    MatIcon,
    MatInput,
    MatLabel,
    MatOption,
    MatSelect,
    // ... etc.
    DocReferencesComponent,
    // cadmus
    CloseSaveButtonsComponent,
    HelpLinkComponent,
  ],
  templateUrl: "./__NAME__-part.component.html",
  styleUrl: "./__NAME__-part.component.scss",
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class __NAME__PartComponent extends ModelEditorComponentBase<__NAME__Part> {
  // thesauri (TODO: replace with yours):
  // __NAME__-tags
  public readonly tagEntries = computed<ThesaurusEntry[] | undefined>(
    () => this.data()?.thesauri?.["__NAME__-tags"]?.entries,
  );
  // doc-reference-types
  public readonly refTypeEntries = computed<ThesaurusEntry[] | undefined>(
    () => this.data()?.thesauri?.["doc-reference-types"]?.entries,
  );
  // doc-reference-tags
  public readonly refTagEntries = computed<ThesaurusEntry[] | undefined>(
    () => this.data()?.thesauri?.["doc-reference-tags"]?.entries,
  );

  // OPTIONAL: settings (see the constructor; import signal from @angular/core)
  // private readonly _settings = signal<__NAME__PartSettings | undefined>(undefined);

  // the draft is rebuilt from each new data. If it depends on settings,
  // read them here too: toDraft(this.data()?.value, this._settings())
  private readonly _draft = linkedSignal(() => toDraft(this.data()?.value));
  public readonly form = this.createForm(this._draft, (p) => {
    // TODO: replace with your rules
    maxLength(p.tag, 100);
    required(p.text);
    maxLength(p.text, 5000);
  });

  // OPTIONAL: a constructor is needed only to load settings
  // constructor() {
  //   super();
  //   this.initSettings<__NAME__PartSettings>(__NAME___PART_TYPEID, (s) =>
  //     this._settings.set(s),
  //   );
  // }

  // EXAMPLE: handler of a child editor's output. Always use setFieldFromChild
  // here: it ignores the copy which the child emits of the value it received,
  // so that opening the part does not make it dirty.
  public onReferencesChange(references: DocReference[]): void {
    setFieldFromChild(this.form.references, copyFormValue(references || []));
  }

  // draft -> part
  protected getValue(): __NAME__Part {
    const part = this.getEditedPart(__NAME___PART_TYPEID) as __NAME__Part;
    const draft = this._draft();
    // TODO: replace with your fields
    part.tag = draft.tag.trim() || undefined;
    part.text = draft.text.trim();
    part.references = draft.references.length
      ? copyFormValue(draft.references)
      : undefined;
    return part;
  }
}
```

- 📁 part editor HTML template:

```html
<!-- __NAME__-part.component.html -->
<!-- no <form> here: the part is saved by the save button -->
<mat-card appearance="outlined">
  <mat-card-header>
    <div mat-card-avatar>
      <mat-icon>picture_in_picture</mat-icon>
    </div>
    <mat-card-title
      >{{ (modelName() | titlecase) || "__NAME__ Part" }}</mat-card-title
    >
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

    <!-- EXAMPLE: child editor: input from the field's value, output through
         a handler using setFieldFromChild -->
    <cadmus-refs-doc-references
      [references]="form.references().value()"
      [typeEntries]="refTypeEntries()"
      [tagEntries]="refTagEntries()"
      (referencesChange)="onReferencesChange($event)"
    />
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

> Note that the `modelName()` human-friendly part name is dynamically defined according to the `model-types` thesaurus for both pure parts and parts with a specific role. That's why the title is `(modelName() | titlecase) || "__NAME__ Part"`. For instance, if you are going to use a categories part with role "eras", you should add to that thesaurus an entry with ID `it.vedph.categories:eras` whose value will be used as the human-friendly name for that part type with that specific role.

▶️ (2) ensure the component has been added to the `public-api.ts` barrel file.

💡 Some practical tips:

- **child editors which reset on any new object**: pass a child editor's input from the field's value (`form.x().value()`), as above. A few child editors reset their whole state whenever they get a different object (e.g. `NoteSetComponent`): for them, pass instead a `computed()` built only from `data()` (and settings), so that their own changes do not make them reset.
- **disabled state**: `disabled` disables all the form's fields, but not child editors bound with plain inputs. If a child editor has a `disabled` input, bind it to `disabled()`.
- **sub-forms**: a form which is not part of the edited model (e.g. a "new keyword" input with its add button) is a separate `form(signal(...))`, so that it does not affect the part's dirty and valid state. Handle Enter-to-add on its input with `(keydown.enter)`.
- **rows** (lists of editable objects edited in place, like metadata name=value pairs): use an array in the draft, `applyEach(p.rows, (row) => { ... })` for their rules, and iterate the field tree in the template: `@for (row of form.rows; track row) { <input matInput [formField]="row.name" /> }`. Change the array by setting a new one (e.g. `this.form.rows().value.set([...rows, newRow])` followed by `markAsDirty()`); never `push` into it.
- **settings**: if your editor needs to be customized with settings, add them to the backend JSON profile, and load them with `this.initSettings<YourSettings>(TYPEID, (s) => this._settings.set(s))` in the constructor. The settings are looked up by the part's type ID and role.

### 2.3. List Part Editor Template

This template is for a part whose model is just a list of entries, each edited by a separate entry editor component (see [the entry editor template](#list-entry-editor-template) below). The list is a single field of the form (`entries`), changed as a whole by the user's actions (add, edit, delete, move).

> Typically you edit each single entry in its own component, generated with `ng g component <NAME>-editor`, where NAME is the model's name (e.g. `cod-binding-editor` for the entries of the `cod-bindings-part`). Remember to export it from the library's `public-api.ts` barrel file. The same template is used for any child editor of a single object.

▶️ (1) write code and HTML template:

- 📁 list part editor code:

```ts
// __NAME__s-part.component.ts

import {
  ChangeDetectionStrategy,
  Component,
  computed,
  inject,
  linkedSignal,
  signal,
} from "@angular/core";
import { TitleCasePipe } from "@angular/common";

import { MatButton, MatIconButton } from "@angular/material/button";
import {
  MatCard,
  MatCardActions,
  MatCardAvatar,
  MatCardContent,
  MatCardHeader,
  MatCardTitle,
} from "@angular/material/card";
import { MatExpansionModule } from "@angular/material/expansion";
import { MatIcon } from "@angular/material/icon";
import { MatTooltip } from "@angular/material/tooltip";
// ... etc.

import { NgxToolsSignalValidators, FlatLookupPipe } from "@myrmidon/ngx-tools";
import { DialogService } from "@myrmidon/ngx-mat-tools";
import { ThesaurusEntry } from "@myrmidon/cadmus-core";
import {
  CloseSaveButtonsComponent,
  HelpLinkComponent,
  ModelEditorComponentBase,
  copyFormValue,
} from "@myrmidon/cadmus-ui";

import {
  __NAME__,
  __NAME__sPart,
  __NAME__S_PART_TYPEID,
} from "../__NAME__s-part";
import { __NAME__EditorComponent } from "../__NAME__-editor/__NAME__-editor.component";

interface __NAME__sPartControls {
  entries: __NAME__[];
}

function toDraft(part?: __NAME__sPart | null): __NAME__sPartControls {
  // copy: the form tags the objects in its arrays
  return { entries: copyFormValue(part?.__NAME__s || []) };
}

/**
 * __NAME__sPart editor component.
 * Thesauri: TODO list of thesauri IDs, e.g. __NAME__-types (optional).
 */
@Component({
  selector: "cadmus-__NAME__s-part",
  imports: [
    TitleCasePipe,
    MatButton,
    MatCard,
    MatCardActions,
    MatCardAvatar,
    MatCardContent,
    MatCardHeader,
    MatCardTitle,
    MatExpansionModule,
    MatIcon,
    MatIconButton,
    MatTooltip,
    FlatLookupPipe,
    // ... etc.
    __NAME__EditorComponent,
    // cadmus
    CloseSaveButtonsComponent,
    HelpLinkComponent,
  ],
  templateUrl: "./__NAME__s-part.component.html",
  styleUrl: "./__NAME__s-part.component.scss",
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class __NAME__sPartComponent extends ModelEditorComponentBase<__NAME__sPart> {
  private readonly _dialogService = inject(DialogService);

  // the entry being edited (a copy), and its index (-1 for a new entry)
  public readonly editedIndex = signal<number>(-1);
  public readonly edited = signal<__NAME__ | undefined>(undefined);

  // thesauri (TODO: replace with yours):
  // __NAME__-types
  public readonly typeEntries = computed<ThesaurusEntry[] | undefined>(
    () => this.data()?.thesauri?.["__NAME__-types"]?.entries,
  );

  private readonly _draft = linkedSignal(() => toDraft(this.data()?.value));
  public readonly form = this.createForm(this._draft, (p) => {
    // at least 1 entry (required() does not flag an empty array)
    NgxToolsSignalValidators.strictMinLength(p.entries, 1);
  });

  protected getValue(): __NAME__sPart {
    const part = this.getEditedPart(__NAME__S_PART_TYPEID) as __NAME__sPart;
    part.__NAME__s = copyFormValue(this._draft().entries);
    return part;
  }

  public add__NAME__(): void {
    const entry: __NAME__ = {
      // TODO: set your entry default properties, e.g. the first type:
      // type: this.typeEntries()?.length ? this.typeEntries()![0].id : '',
    } as __NAME__;
    this.edit__NAME__(entry, -1);
  }

  public edit__NAME__(entry: __NAME__, index: number): void {
    this.editedIndex.set(index);
    // structuredClone also drops the form's Symbol tag
    this.edited.set(structuredClone(entry));
  }

  public close__NAME__(): void {
    this.editedIndex.set(-1);
    this.edited.set(undefined);
  }

  // these are user actions: they set the list and mark it as dirty
  public save__NAME__(entry: __NAME__): void {
    const entries = [...this.form.entries().value()];
    if (this.editedIndex() === -1) {
      entries.push(entry);
    } else {
      entries.splice(this.editedIndex(), 1, entry);
    }
    this.form.entries().value.set(entries);
    this.form.entries().markAsDirty();
    this.close__NAME__();
  }

  public delete__NAME__(index: number): void {
    this._dialogService
      .confirm("Confirmation", "Delete __NAME__?")
      .subscribe((yes: boolean | undefined) => {
        if (yes) {
          if (this.editedIndex() === index) {
            this.close__NAME__();
          }
          const entries = [...this.form.entries().value()];
          entries.splice(index, 1);
          this.form.entries().value.set(entries);
          this.form.entries().markAsDirty();
        }
      });
  }

  public move__NAME__Up(index: number): void {
    if (index < 1) {
      return;
    }
    const entries = [...this.form.entries().value()];
    const entry = entries[index];
    entries.splice(index, 1);
    entries.splice(index - 1, 0, entry);
    this.form.entries().value.set(entries);
    this.form.entries().markAsDirty();
  }

  public move__NAME__Down(index: number): void {
    if (index + 1 >= this.form.entries().value().length) {
      return;
    }
    const entries = [...this.form.entries().value()];
    const entry = entries[index];
    entries.splice(index, 1);
    entries.splice(index + 1, 0, entry);
    this.form.entries().value.set(entries);
    this.form.entries().markAsDirty();
  }
}
```

- 📁 list part editor HTML template:

```html
<!-- __NAME__s-part.component.html -->
<mat-card appearance="outlined">
  <mat-card-header>
    <div mat-card-avatar>
      <mat-icon>picture_in_picture</mat-icon>
    </div>
    <mat-card-title
      >{{ (modelName() | titlecase) || "__NAME__s Part" }}</mat-card-title
    >
    <cadmus-help-link [url]="helpUrl()" />
  </mat-card-header>

  <mat-card-content>
    <div>
      <button
        type="button"
        mat-flat-button
        class="mat-primary"
        (click)="add__NAME__()"
      >
        <mat-icon>add_circle</mat-icon> __NAME__
      </button>
    </div>

    @if (form.entries().value().length) {
    <table>
      <thead>
        <tr>
          <th></th>
          <!-- TODO: a th for each displayed property -->
          <th>type</th>
        </tr>
      </thead>
      <tbody>
        @for ( entry of form.entries().value(); track entry; let i = $index; let
        first = $first; let last = $last ) {
        <tr [class.selected]="i === editedIndex()">
          <td class="fit-width">
            <span class="nr">{{ i + 1 }}.</span>
            <button
              type="button"
              mat-icon-button
              matTooltip="Edit this __NAME__"
              (click)="edit__NAME__(entry, i)"
            >
              <mat-icon class="mat-primary">edit</mat-icon>
            </button>
            <button
              type="button"
              mat-icon-button
              matTooltip="Move this __NAME__ up"
              [disabled]="first"
              (click)="move__NAME__Up(i)"
            >
              <mat-icon>arrow_upward</mat-icon>
            </button>
            <button
              type="button"
              mat-icon-button
              matTooltip="Move this __NAME__ down"
              [disabled]="last"
              (click)="move__NAME__Down(i)"
            >
              <mat-icon>arrow_downward</mat-icon>
            </button>
            <button
              type="button"
              mat-icon-button
              matTooltip="Delete this __NAME__"
              (click)="delete__NAME__(i)"
            >
              <mat-icon class="mat-warn">remove_circle</mat-icon>
            </button>
          </td>
          <!-- TODO: a td for each displayed property, e.g.: -->
          <td>{{ entry.type | flatLookup: typeEntries() : "id" : "value" }}</td>
        </tr>
        }
      </tbody>
    </table>
    }

    <!-- the entry editor (manual save: it emits only on its save button) -->
    @if (edited()) {
    <fieldset>
      <mat-expansion-panel [expanded]="true">
        <mat-expansion-panel-header>
          <mat-panel-title>
            __NAME__ #{{ editedIndex() > -1 ? editedIndex() + 1 : "new" }}
          </mat-panel-title>
        </mat-expansion-panel-header>
        <cadmus-__PRJ__-__NAME__-editor
          [data]="edited()"
          [typeEntries]="typeEntries()"
          (dataChange)="save__NAME__($event!)"
          (cancelEdit)="close__NAME__()"
        />
      </mat-expansion-panel>
    </fieldset>
    }
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

⚠️ The entry editor must use **manual save** here: `save__NAME__()` closes it, so an autosaving editor would close at the first change.

- 📁 list part editor styles:

```css
table {
  width: 100%;
  border-collapse: collapse;
}
tbody tr:nth-child(odd) {
  background-color: var(--mat-sys-surface-container);
}
th {
  text-align: left;
  font-weight: normal;
  color: var(--mat-sys-on-surface-variant);
}
tbody tr:hover {
  background-color: var(--mat-sys-surface-container-high);
}
td.fit-width {
  width: 1px;
  white-space: nowrap;
}
tbody tr.selected {
  background-color: var(--mat-sys-secondary-container);
  color: var(--mat-sys-on-secondary-container);
}
fieldset {
  border: 1px solid var(--mat-sys-on-surface-variant);
  border-radius: 6px;
  padding: 6px;
}
.nr {
  color: var(--mat-sys-on-surface-variant);
  background-color: var(--mat-sys-surface-container);
  border: 1px solid var(--mat-sys-on-surface-variant);
  border-radius: 4px;
  padding: 0 2px;
}
```

▶️ (2) ensure the component has been added to the `public-api.ts` barrel file.

## 3. Add PG Editor Wrapper

▶️ (1) under your library's `src/lib` folder, add a **part editor feature component** named after the part (e.g. `ng g component note-part-feature` for `NotePartFeatureComponent` after `NotePart`).

- 📁 editor wrapper code:

```ts
import { Component, OnInit } from "@angular/core";
import { MatSnackBar } from "@angular/material/snack-bar";
import { Router, ActivatedRoute } from "@angular/router";

import { ItemService, ThesaurusService } from "@myrmidon/cadmus-api";
import { EditPartFeatureBase, PartEditorService } from "@myrmidon/cadmus-state";
import { CurrentItemBarComponent } from "@myrmidon/cadmus-item-editor";

import { __NAME__PartComponent } from "@myrmidon/cadmus-lon-part-ui";

@Component({
  selector: "cadmus-__NAME__-part-feature",
  imports: [CurrentItemBarComponent, __NAME__PartComponent],
  templateUrl: "./__NAME__-part-feature.component.html",
  styleUrl: "./__NAME__-part-feature.component.css",
})
export class __NAME__PartFeatureComponent
  extends EditPartFeatureBase
  implements OnInit
{
  constructor(
    router: Router,
    route: ActivatedRoute,
    snackbar: MatSnackBar,
    itemService: ItemService,
    thesaurusService: ThesaurusService,
    editorService: PartEditorService,
  ) {
    super(
      router,
      route,
      snackbar,
      itemService,
      thesaurusService,
      editorService,
    );
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

- 📁 editor wrapper HTML template:

```html
<cadmus-current-item-bar />
<cadmus-__NAME__-part
  [identity]="identity()"
  [data]="$any(data())"
  (dataChange)="save($event!.value!)"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
/>
```

> Note that since version 12 the `dataChange` handler requires `$event!.value!` as an argument rather than just `$event` as before. This is because version 12 moved input/output endpoints to signals, which also implied a better alignment between the type of `data` and of its corresponding event.

▶️ (2) ensure that this component is exported from the `public-api.ts` barrel file.

## 4. Add Sub-Route

This is optional, and is required only when you are providing a library sub-routes to a set of related PG editor wrapper components. In legacy Cadmus applications, a module was used to export these sub-routes. In modern, module-less Angular instead, you can just export an array of routes.

▶️ (1) add the corresponding **route** in the PG library's module, e.g.:

Modern approach:

```ts
import { Routes } from "@angular/router";

// cadmus
import { pendingChangesGuard } from "@myrmidon/cadmus-core";

import {
  __NAME___PART_TYPEID,
  __NAME__PartFeatureComponent,
} from "@myrmidon/cadmus-part-TODO";

export const CADMUS_PART___PRJ___PG_ROUTES: Routes = [
  {
    path: `${__NAME___PART_TYPEID}/:pid`,
    pathMatch: "full",
    component: __NAME__PartFeatureComponent,
    canDeactivate: [pendingChangesGuard],
  },
  // ... etc.
];
```

> 💡 This is imported in `app.routes.ts` like e.g.:

```ts
export const routes: Routes = [
  // ...
  // ndp-books part
  {
    path: "items/:iid/ndp-books",
    loadChildren: () =>
      import("@myrmidon/cadmus-part-ndpbooks-pg").then(
        (module) => module.CADMUS_PART_NDPBOOKS_PG_ROUTES,
      ),
    canActivate: [jwtGuard],
  },
  // ...
];
```

Legacy approach using modules:

```ts
export const RouterModuleForChild = RouterModule.forChild([
  // TODO your part route
  {
    path: `${__NAME___PART_TYPEID}/:pid`,
    pathMatch: "full",
    component: __NAME__PartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
]);

@NgModule({
  declarations: [__NAME__PartFeatureComponent],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModuleForChild,
  ],
  exports: [__NAME__PartFeatureComponent],
})
export class CadmusPart__PRJ__PgModule {}
```

## 5. Add Part Mapping to App

▶️ (1) In your app's project `part-editor-keys.ts`, add the mapping for the part just created, like e.g.:

```ts
// this constant refers to the project-dependent portion of the route path
// (items/:iid/__PRJ__) in routes definitions
const ITINERA_LT = 'itinera_lt';

// itinera parts example
[PERSON_PART_TYPEID]: {
  part: ITINERA_LT
},
```

Here, the type ID of the part (from its model in the "ui" library) is mapped to the route prefix constant `ITINERA_LT` = `itinera-lt`, which is the root route to the "pg" library module for the app.

▶️ (2) Ensure that your app routes (usually `app.routes.ts`) include the PG component from its lazily loaded library, e.g.:

```ts
// cadmus - lon parts
{
  path: 'items/:iid/__PRJ__',
  loadComponent: () =>
    import('@myrmidon/cadmus-__PRJ__-part').then(
      (module) => module.Cadmus__PRJ____NAME__PartComponent
    ),
  canActivate: [AuthJwtGuardService],
},
```
