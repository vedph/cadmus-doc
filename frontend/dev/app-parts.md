---
title: "Creating Frontend Parts"
parent: "Developing Frontend"
layout: default
nav_order: 9
---

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

The part editor UI is a dumb component which essentially uses a form to represent the data of a part's model. These data are adapted to the form when loading them, and converted back to the part's model when saving.

▶️ (1) in `src/lib`, add a **part editor dumb component** named after the part (e.g. `ng g component note-part` for `NotePartComponent` after the model `NotePart`), and extending `ModelEditorComponentBase<T>` where `T` is the part's type. Here we usually have two cases:
    - a generic part
    - a part consisting only of a list of entities.

Two different templates are provided here.

### 2.1. Generic Part Editor Template

▶️ (1) write code and HTML template:

- 📁 part editor code:

```ts
// NAME-part.component.ts

import { Component, OnInit } from '@angular/core';
import {
  FormControl,
  FormBuilder,
  FormGroup,
  UntypedFormGroup,
  ReactiveFormsModule,
} from '@angular/forms';

import { CommonModule } from '@angular/common';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatExpansionModule } from '@angular/material/expansion';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatTooltipModule } from '@angular/material/tooltip';
// ... etc.

import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { ThesauriSet, ThesaurusEntry } from '@myrmidon/cadmus-core';
import { EditedObject, ModelEditorComponentBase } from '@myrmidon/cadmus-ui';

import { __NAME__Part, __NAME___PART_TYPEID } from '../__NAME__-part';

/**
 * __NAME__ part editor component.
 * Thesauri: ...TODO list of thesauri IDs...
 */
@Component({
  selector: 'cadmus-__NAME__-part',
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatCardModule,
    MatExpansionModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatSelectModule,
    MatTooltipModule,
    // ... etc.
    // cadmus
    CloseSaveButtonsComponent
  ],
  templateUrl: './__NAME__-part.component.html',
  styleUrls: ['./__NAME__-part.component.scss'],
})
export class __NAME__PartComponent
  extends ModelEditorComponentBase<__NAME__Part>
  implements OnInit
{
  // TODO: add your form controls here, e.g.:
  // public tag: FormControl<string | null>;
  // public text: FormControl<string | null>;

  // TODO: add your thesauri entries here, e.g.:
  // public tagEntries?: ThesaurusEntry[];

  constructor(authService: AuthJwtService, formBuilder: FormBuilder) {
    super(authService, formBuilder);
    // form
    // TODO: create your form controls (but NOT the form itself), e.g.:
    // this.tag = formBuilder.control(null, Validators.maxLength(100));
    // this.text = formBuilder.control('', Validators.required, { nonNullable: true });
  }

  public override ngOnInit(): void {
    super.ngOnInit();
  }

  protected buildForm(formBuilder: FormBuilder): FormGroup | UntypedFormGroup {
    return formBuilder.group({
      // TODO: assign your created form controls to the form returned here, e.g.:
      // tag: this.tag,
      // text: this.text,
    });
  }

  private updateThesauri(thesauri: ThesauriSet): void {
    // TODO: setup thesauri entries here, e.g.:
    // const key = 'note-tags';
    // if (this.hasThesaurus(key)) {
    //  this.tagEntries = thesauri[key].entries;
    // } else {
    //  this.tagEntries = undefined;
    // }
  }

  private updateForm(part?: __NAME__Part | null): void {
    if (!part) {
      this.form.reset();
      return;
    }
    // TODO: set values of your form controls, e.g.:
    // this.tag.setValue(part.tag || null);
    // this.text.setValue(part.text);
    this.form.markAsPristine();
  }

  protected override onDataSet(data?: EditedObject<__NAME__Part>): void {
    // thesauri
    if (data?.thesauri) {
      this.updateThesauri(data.thesauri);
    }

    // form
    this.updateForm(data?.value);
  }

  protected getValue(): __NAME__Part {
    let part = this.getEditedPart(__NAME___PART_TYPEID) as __NAME__Part;
    // TODO: assign values to your part properties from form controls, e.g.:
    // part.tag = this.tag.value || undefined;
    // part.text = this.text.value?.trim() || '';
    return part;
  }
}
```

- 📁 part editor HTML template:

```html
<!-- NAME-part.component.html -->

<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>picture_in_picture</mat-icon>
      </div>
      <mat-card-title>__NAME__ Part</mat-card-title>
    </mat-card-header>
    <mat-card-content> TODO: your template here... </mat-card-content>
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

▶️ (2) ensure the component has been added to the `public-api.ts` barrel file (and, if still using modules, to the library module's `declarations` and `exports`).

### 2.2. List Part Editor Template

▶️ (1) write code and HTML template:

- 📁 list part editor code:

```ts
// NAME-part.component.ts

import { Component, OnInit } from '@angular/core';
import {
  FormControl,
  FormBuilder,
  FormGroup,
  UntypedFormGroup,
  ReactiveFormsModule,
} from '@angular/forms';
import { take } from 'rxjs/operators';

import { CommonModule } from '@angular/common';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatExpansionModule } from '@angular/material/expansion';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatTooltipModule } from '@angular/material/tooltip';
// ... etc.

import { NgxToolsValidators } from '@myrmidon/ngx-tools';
import { DialogService } from '@myrmidon/ngx-mat-tools';
import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { EditedObject, ModelEditorComponentBase } from '@myrmidon/cadmus-ui';
import { ThesauriSet, ThesaurusEntry } from '@myrmidon/cadmus-core';

import {
  __NAME__,
  __NAME__sPart,
  __NAME__S_PART_TYPEID,
} from '../__NAME__s-part';

/**
 * __NAME__sPart editor component.
 * Thesauri: ...TODO list of thesauri IDs...
 */
@Component({
  selector: 'cadmus-__NAME__s-part',
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatCardModule,
    MatExpansionModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatSelectModule,
    MatTooltipModule,
    // ... etc.
    // cadmus
    CloseSaveButtonsComponent
  ],
  templateUrl: './__NAME__s-part.component.html',
  styleUrls: ['./__NAME__s-part.component.scss'],
})
export class __NAME__sPartComponent
  extends ModelEditorComponentBase<__NAME__sPart>
  implements OnInit
{
  public editedIndex: number;
  public edited: __NAME__ | undefined;

  // TODO: add your thesauri entries here, e.g.:
  // cod-binding-tags
  // public tagEntries: ThesaurusEntry[] | undefined;

  public entries: FormControl<__NAME__[]>;

  constructor(
    authService: AuthJwtService,
    formBuilder: FormBuilder,
    private _dialogService: DialogService
  ) {
    super(authService, formBuilder);
    this.editedIndex = -1;
    // form
    this.entries = formBuilder.control([], {
      // at least 1 entry
      validators: NgToolsValidators.strictMinLengthValidator(1),
      nonNullable: true,
    });
  }

  public override ngOnInit(): void {
    super.ngOnInit();
  }

  protected buildForm(formBuilder: FormBuilder): FormGroup | UntypedFormGroup {
    return formBuilder.group({
      entries: this.entries,
    });
  }

  private updateThesauri(thesauri: ThesauriSet): void {
    // TODO setup your thesauri entries here, e.g.:
    // let key = 'cod-binding-tags';
    // if (this.hasThesaurus(key)) {
    //   this.tagEntries = thesauri[key].entries;
    // } else {
    //   this.tagEntries = undefined;
    // }
  }

  private updateForm(part?: __NAME__sPart | null): void {
    if (!part) {
      this.form.reset();
      return;
    }
    this.entries.setValue(part.__NAME__s || []);
    this.form.markAsPristine();
  }

  protected override onDataSet(data?: EditedObject<__NAME__sPart>): void {
    // thesauri
    if (data?.thesauri) {
      this.updateThesauri(data.thesauri);
    }

    // form
    this.updateForm(data?.value);
  }

  protected getValue(): __NAME__sPart {
    let part = this.getEditedPart(__NAME__S_PART_TYPEID) as __NAME__sPart;
    part.__NAME__s = this.entries.value || [];
    return part;
  }

  public add__NAME__(): void {
    const entry: __NAME__ = {
      // TODO: set your entry default properties...
    };
    this.edit__NAME__(entry, -1);
  }

  public edit__NAME__(entry: __NAME__, index: number): void {
    this.editedIndex = index;
    this.edited = entry;
  }

  public close__NAME__(): void {
    this.editedIndex = -1;
    this.edited = undefined;
  }

  public save__NAME__(entry: __NAME__): void {
    const entries = [...this.entries.value];
    if (this.editedIndex === -1) {
      entries.push(entry);
    } else {
      entries.splice(this.editedIndex, 1, entry);
    }
    this.entries.setValue(entries);
    this.entries.markAsDirty();
    this.entries.updateValueAndValidity();
    this.close__NAME__();
  }

  public delete__NAME__(index: number): void {
    this._dialogService
      .confirm('Confirmation', 'Delete __NAME__?')
      .subscribe((yes: boolean | undefined) => {
        if (yes) {
          if (this.editedIndex === index) {
            this.close__NAME__();
          }
          const entries = [...this.entries.value];
          entries.splice(index, 1);
          this.entries.setValue(entries);
          this.entries.markAsDirty();
          this.entries.updateValueAndValidity();
        }
      });
  }

  public move__NAME__Up(index: number): void {
    if (index < 1) {
      return;
    }
    const entry = this.entries.value[index];
    const entries = [...this.entries.value];
    entries.splice(index, 1);
    entries.splice(index - 1, 0, entry);
    this.entries.setValue(entries);
    this.entries.markAsDirty();
    this.entries.updateValueAndValidity();
  }

  public move__NAME__Down(index: number): void {
    if (index + 1 >= this.entries.value.length) {
      return;
    }
    const entry = this.entries.value[index];
    const entries = [...this.entries.value];
    entries.splice(index, 1);
    entries.splice(index + 1, 0, entry);
    this.entries.setValue(entries);
    this.entries.markAsDirty();
    this.entries.updateValueAndValidity();
  }
}
```

- 📁 list part editor HTML template:

```html
<!-- NAME-part.component.html -->

<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>picture_in_picture</mat-icon>
      </div>
      <mat-card-title>__NAME__s Part</mat-card-title>
    </mat-card-header>
    <mat-card-content>
          <div>
            <button
              type="button"
              mat-flat-button
              color="primary"
              (click)="add__NAME__()"
            >
              <mat-icon>add_circle</mat-icon> __NAME__
            </button>
          </div>
          @if (entries.value.length) {
          <table>
            <thead>
              <tr>
                <th></th>
                TODO: add model properties
              </tr>
            </thead>
            <tbody>
              @for (entry of entries.value; track entry; let i = $index; let first =
          $first; let last = $last) {
              <tr [class.selected]="entry === edited">
                <td class="fit-width">
                  <button
                    type="button"
                    mat-icon-button
                    color="primary"
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
                    color="warn"
                    matTooltip="Delete this __NAME__"
                    (click)="delete__NAME__(i)"
                  >
                    <mat-icon class="mat-warn">remove_circle</mat-icon>
                  </button>
                </td>
                TODO: td's for properties
              </tr>
              }
            </tbody>
          </table>
          }

        @if (edited) {
        <fieldset>
          <mat-expansion-panel [expanded]="edited" [disabled]="!edited">
            <mat-expansion-panel-header>
              <mat-panel-title>__NAME__ #{{ editedIndex + 1 }}</mat-panel-title>
            </mat-expansion-panel-header>
            TODO: editor control with: [model]="edited"
            (modelChange)="save__NAME__($event)"
            (editorClose)="close__NAME__()"
          </mat-expansion-panel>
        </fieldset>
        }
    </mat-card-content>
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

- 📁 list part editor CSS styles:

```css
table {
  width: 100%;
  border-collapse: collapse;
}
tbody tr:nth-child(odd) {
  background-color: #e2e2e2;
}
th {
  text-align: left;
  font-weight: normal;
  color: silver;
}
td.fit-width {
  width: 1px;
  white-space: nowrap;
}
tr.selected {
  background-color: #d0d0d0 !important;
}
fieldset {
  border: 1px solid silver;
  border-radius: 6px;
  padding: 6px;
}
```

Typically you should edit each **single entry** in a component (generated with `ng g component <NAME>-editor` where NAME is the model's name, e.g. `cod-binding-editor` for the `cod-bindings-part` component - remember to export it both from the library's module and from its barrel `public-api.ts` file), similar to the following template (rename `model` as you prefer):

- 📁 entry editor code:

```ts
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
import {
  FormArray,
  FormBuilder,
  FormControl,
  FormGroup,
  Validators,
} from '@angular/forms';

import { MatButtonModule } from '@angular/material/button';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatTooltipModule } from '@angular/material/tooltip';
// ... etc.

import { ThesaurusEntry } from '@myrmidon/cadmus-core';

@Component({
  selector: 'cadmus-__PRJ__-__NAME__',
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatCheckboxModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatSelectModule,
    MatTooltipModule,
    // ... etc.
  ],
  templateUrl: './__NAME__.component.html',
  styleUrls: ['./__NAME__.component.scss'],
})
export class __NAME__Component implements OnInit {
  // TODO rename model into something more specific
  public readonly model = model<__TYPE__>();
  public readonly modelCancel = output();

  // TODO: controls
  public form: FormGroup;

  constructor(formBuilder: FormBuilder) {
    // form
    // TODO: create controls

    this.form = formBuilder.group({
      // TODO: add controls to form model
    });

    // when model changes, update form
    effect(() => {
      this.updateForm(this.model());
    });
  }

  private updateForm(model: __TYPE__ | undefined | null): void {
    if (!model) {
      this.form.reset();
      return;
    }

    // TODO set controls values

    this.form.markAsPristine();
  }

  private getModel(): __TYPE__ {
    return {
      // TODO get values from controls
    };
  }

  public cancel(): void {
    this.modelCancel.emit();
  }

  public save(): void {
    if (this.form.invalid) {
      return;
    }
    this.model.set(this.getModel());
  }
}
```

- 📁 HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  TODO
  <!-- buttons -->
  <div>
    <button
      type="button"
      mat-icon-button
      matTooltip="Discard changes"
      (click)="cancel()"
    >
      <mat-icon class="mat-warn">clear</mat-icon>
    </button>
    <button
      type="submit"
      mat-icon-button
      matTooltip="Accept changes"
      [disabled]="form.invalid || form.pristine"
    >
      <mat-icon class="mat-primary">check_circle</mat-icon>
    </button>
  </div>
</form>
```

▶️ (2) ensure the component has been added to the `public-api.ts` barrel file.

## 3. Add PG Editor Wrapper

▶️ (1) under your library's `src/lib` folder, add a **part editor feature component** named after the part (e.g. `ng g component note-part-feature` for `NotePartFeatureComponent` after `NotePart`).

- 📁 editor wrapper code:

```ts
import { Component, OnInit } from '@angular/core';
import { MatSnackBar } from '@angular/material/snack-bar';
import { Router, ActivatedRoute } from '@angular/router';

import { ItemService, ThesaurusService } from '@myrmidon/cadmus-api';
import { EditPartFeatureBase, PartEditorService } from '@myrmidon/cadmus-state';
import { CurrentItemBarComponent } from '@myrmidon/cadmus-ui-pg';

import { __NAME__PartComponent } from '@myrmidon/cadmus-lon-part-ui';

@Component({
  selector: 'cadmus-__NAME__-part-feature',
  imports: [CurrentItemBarComponent, __NAME__PartComponent],
  templateUrl: './__NAME__-part-feature.component.html',
  styleUrl: './__NAME__-part-feature.component.css',
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
    editorService: PartEditorService
  ) {
    super(
      router,
      route,
      snackbar,
      itemService,
      thesaurusService,
      editorService
    );
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

- 📁 editor wrapper HTML template:

```html
<cadmus-current-item-bar/>
<cadmus-__NAME__-part
  [identity]="identity"
  [data]="$any(data)"
  (dataChange)="save($event!.value!)"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
/>
```

>Note that since version 12 the `dataChange` handler requires `$event!.value!` as an argument rather than just `$event` as before. This is because version 12 moved input/output endpoints to signals, which also implied a better alignment between the type of `data` and of its corresponding event.

▶️ (2) ensure that this component is exported from the `public-api.ts` barrel file.

## 4. Add Sub-Route

This is optional, and is required only when you are providing a library with a module containing sub-routes to a set of related PG editor wrapper components.

▶️ (1) add the corresponding **route** in the PG library's module, e.g.:

```ts
export const RouterModuleForChild = RouterModule.forChild([
  // TODO your part route
  {
     path: `${__NAME___PART_TYPEID}/:pid`,
     pathMatch: 'full',
     component: __NAME__PartFeatureComponent,
     canDeactivate: [PendingChangesGuard]
  },
]);

@NgModule({
  declarations: [
    __NAME__PartFeatureComponent
  ],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModuleForChild,
  ],
  exports: [
    __NAME__PartFeatureComponent
  ],
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
