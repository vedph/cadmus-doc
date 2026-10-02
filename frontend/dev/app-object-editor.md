---
title: "Creating an Object Editor"
parent: "Developing Frontend"
layout: default
nav_order: 11
---

# Creating an Object Editor

This component does not derive from `ModelEditorComponentBase`: it is a plain component with a `data` model signal. It comes in two flavors; keep the one you need and delete the sections of the other:

- **manual save**: the user saves with a button (or Enter in a text input), or discards the changes. Use this in list parts, where saving closes the editor.
- **autosave**: the model is updated on every valid change, after a short delay. Use this for editors which are always visible inside another editor. The parent then handles `(dataChange)` with `setFieldFromChild` (see [2.1](app-parts.md#21-how-a-part-editor-works)).

Sections belonging to a single flavor are marked `── MANUAL SAVE ONLY ──` and `── AUTOSAVE ONLY ──`. Placeholders: `__PRJ__`, `__NAME__` (the component name), `__TYPE__` (the edited model type).

The template keeps the conversion between model and draft in two pure functions, `toDraft()` and `toData()`, which are used in three places: building the draft, telling when the draft has unsaved changes, and saving.

## Code

- 📁 entry editor code:

```ts
// __NAME__-editor.component.ts

import {
  ChangeDetectionStrategy,
  Component,
  effect,
  input,
  linkedSignal,
  model,
  output,
  untracked,
} from "@angular/core";
import { takeUntilDestroyed, toObservable } from "@angular/core/rxjs-interop";
import { FormField, form, maxLength, required } from "@angular/forms/signals";
import { debounceTime } from "rxjs";

import { MatButtonModule } from "@angular/material/button";
import { MatCheckboxModule } from "@angular/material/checkbox";
import { MatFormFieldModule } from "@angular/material/form-field";
import { MatIconModule } from "@angular/material/icon";
import { MatInputModule } from "@angular/material/input";
import { MatSelectModule } from "@angular/material/select";
import { MatTooltipModule } from "@angular/material/tooltip";
// ... etc.

import { ThesaurusEntry } from "@myrmidon/cadmus-core";
import { isImplicitSubmission } from "@myrmidon/cadmus-ui";

/**
 * The editable draft behind the form.
 *
 * Fields bound to a native <input> must be non-nullable: use '' as the empty
 * value for text, and number | null only for numeric inputs. A
 * string | null field fails template type-checking. Fields bound to
 * Material selects and checkboxes, or to child editors, may be null.
 */
interface __NAME__Controls {
  // TODO: replace with your fields
  type: string;
  name: string;
  note: string;
}

function makeDefaultDraft(): __NAME__Controls {
  return { type: "", name: "", note: "" };
}

/**
 * Model -> draft. Pure: it reads nothing but its argument, so it can be
 * called from inside the linkedSignal computation below. Copy arrays of
 * objects with copyFormValue (from @myrmidon/cadmus-ui).
 */
function toDraft(data: __TYPE__ | undefined): __NAME__Controls {
  return !data
    ? makeDefaultDraft()
    : {
        type: data.type || "",
        name: data.name || "",
        note: data.note || "",
      };
}

/**
 * Draft -> model. This is where normalization happens (trimming, mapping ''
 * to undefined), which is why the draft and the model are not
 * interchangeable: see the note on _draft below.
 */
function toData(v: __NAME__Controls): __TYPE__ {
  return {
    type: v.type,
    name: v.name.trim(),
    note: v.note.trim() || undefined,
  };
}

/**
 * __NAME__ editor.
 */
@Component({
  selector: "cadmus-__PRJ__-__NAME__-editor",
  imports: [
    FormField,
    MatButtonModule,
    MatCheckboxModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatSelectModule,
    MatTooltipModule,
    // ... etc.
  ],
  templateUrl: "./__NAME__-editor.component.html",
  styleUrl: "./__NAME__-editor.component.scss",
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class __NAME__EditorComponent {
  /** The model being edited. */
  public readonly data = model<__TYPE__ | undefined>();

  // thesauri, received from the parent editor (TODO: replace with yours)
  public readonly typeEntries = input<ThesaurusEntry[]>();

  // ── MANUAL SAVE ONLY ──────────────────────────────────────────────────
  /** Emitted when the user discards the edit. */
  public readonly cancelEdit = output<void>();
  // ──────────────────────────────────────────────────────────────────────

  /**
   * The editable draft: derived from `data`, but writable, as the form
   * binds its fields to it.
   *
   * `previous` tells an external change apart from the echo of our own
   * save. toData() normalizes, so the model written back differs from the
   * draft; without this check, the incoming echo would rebuild the draft and
   * overwrite what the user is still typing. E.g. type "abc " (with a
   * trailing space), let autosave fire, keep typing: with the check you get
   * "abc d", without it "abcd".
   *
   * The JSON comparison works because both sides come from the same mapping
   * functions, so the order of keys is the same; a false "not equal" just
   * rebuilds the draft. If __TYPE__ contains Dates, Maps or class instances,
   * replace it with an explicit equality function.
   */
  private readonly _draft = linkedSignal<
    __TYPE__ | undefined,
    __NAME__Controls
  >({
    source: () => this.data(),
    computation: (data, previous) =>
      previous &&
      JSON.stringify(data) === JSON.stringify(toData(previous.value))
        ? previous.value
        : toDraft(data),
  });

  public readonly form = form(this._draft, (p) => {
    // TODO: replace with your rules. Prefer declarative rules, e.g.
    // disabled(p.note, { when: () => !p.name }) or required(p.x, { when })
    required(p.name);
    maxLength(p.name, 100);
    maxLength(p.note, 500);
  });

  constructor() {
    // when the draft mirrors the bound model again, there are no unsaved
    // edits: clear the interaction state (touched, dirty).
    // This is keyed on the draft, but resets only when the draft is in sync
    // with the model: so it does not reset the user's edits. On the echo of
    // our own save the draft does not change, so this does not run, and
    // validation errors are not cleared under the user's eyes.
    effect(() => {
      const draft = this._draft();
      untracked(() => {
        if (this.isDraftInSync(draft)) {
          this.form().reset();
        }
      });
    });

    // ── AUTOSAVE ONLY ───────────────────────────────────────────────────
    toObservable(this._draft)
      .pipe(debounceTime(400), takeUntilDestroyed())
      .subscribe(() => {
        // skip while the draft still mirrors the bound model: otherwise
        // just receiving a model would immediately save back a normalized
        // copy of it, and the parent would see a change
        if (this.isDraftInSync(this._draft()) || this.form().invalid()) {
          return;
        }
        this.save(false);
      });
    // ────────────────────────────────────────────────────────────────────
  }

  /** True when the draft still mirrors the bound model. */
  private isDraftInSync(draft: __NAME__Controls): boolean {
    return JSON.stringify(draft) === JSON.stringify(toDraft(this.data()));
  }

  // ── MANUAL SAVE ONLY ──────────────────────────────────────────────────
  public cancel(): void {
    this.cancelEdit.emit();
  }

  /**
   * Handle Enter: in a text input, save as the save button would, when it
   * is enabled. This replaces the implicit submission of a <form>.
   * @param event The keydown event.
   */
  public onEnterKey(event: Event): void {
    if (
      !isImplicitSubmission(event) ||
      this.form().invalid() ||
      !this.form().dirty()
    ) {
      return;
    }
    event.preventDefault();
    this.save();
  }
  // ──────────────────────────────────────────────────────────────────────

  /**
   * Save the current draft into the `data` model signal.
   * Called by the save button, or by the autosave subscription.
   * @param pristine true (default) to clear the form's interaction state
   * after saving. Autosave passes false, so that the form does not change
   * its validation state while the user is typing.
   */
  public save(pristine = true): void {
    if (this.form().invalid()) {
      // show the validation errors (descendants included)
      this.form().markAsTouched();
      return;
    }

    this.data.set(toData(this._draft()));

    if (pristine) {
      // this resets only the interaction state, not the values
      this.form().reset();
    }
  }
}
```

## Template

- 📁 entry editor HTML template:

```html
<!-- __NAME__-editor.component.html -->
<!-- no <form> here: this component can be nested at any depth. In the
     manual save flavor, Enter is handled by onEnterKey. Remove the keydown
     handler for autosave. -->
<div (keydown.enter)="onEnterKey($event)">
  <!-- TODO: replace with your controls -->

  <!-- type (bound to thesaurus) -->
  @if (typeEntries()?.length) {
  <mat-form-field>
    <mat-label>type</mat-label>
    <mat-select [formField]="form.type">
      @for (e of typeEntries(); track e.id) {
      <mat-option [value]="e.id">{{ e.value }}</mat-option>
      }
    </mat-select>
  </mat-form-field>
  }
  <!-- type (free) -->
  @else {
  <mat-form-field>
    <mat-label>type</mat-label>
    <input matInput [formField]="form.type" />
  </mat-form-field>
  }

  <!-- name -->
  <mat-form-field>
    <mat-label>name</mat-label>
    <input matInput [formField]="form.name" />
    @if ( form.name().getError("required") && (form.name().dirty() ||
    form.name().touched()) ) {
    <mat-error>name required</mat-error>
    } @if ( form.name().getError("maxLength") && (form.name().dirty() ||
    form.name().touched()) ) {
    <mat-error>name too long</mat-error>
    }
  </mat-form-field>

  <!-- note -->
  <mat-form-field>
    <mat-label>note</mat-label>
    <textarea matInput [formField]="form.note"></textarea>
    @if ( form.note().getError("maxLength") && (form.note().dirty() ||
    form.note().touched()) ) {
    <mat-error>note too long</mat-error>
    }
  </mat-form-field>

  <!-- ── MANUAL SAVE ONLY: delete this block for autosave ────────────── -->
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
      type="button"
      mat-icon-button
      matTooltip="Accept changes"
      [disabled]="form().invalid() || !form().dirty()"
      (click)="save()"
    >
      <mat-icon class="mat-primary">check_circle</mat-icon>
    </button>
  </div>
  <!-- ─────────────────────────────────────────────────────────────────── -->
</div>
```

💡 The accept button is enabled only when the form is valid and dirty. If a new entry with default values should be acceptable as it is, use `[disabled]="form().invalid()"` instead, and remove `!this.form().dirty()` from `onEnterKey`.

💡 If the entry has **child editors**, handle their outputs with `setFieldFromChild(this.form.x, value)` (from `@myrmidon/cadmus-ui`), as in part editors.

## In-Place Edited Array

If the entry has an **array of rows** edited in place (the former `FormArray`), use an array in the draft with `applyEach` rules, and change it by setting a new array on the draft signal (never `push` into it). Map incoming rows into fresh objects (`copyFormValue`), as the form tags each object it adopts:

```ts
interface __NAME__Controls {
  rows: RowControls[];
}

public readonly form = form(this._draft, (p) => {
  applyEach(p.rows, (row) => {
    required(row.value);
    maxLength(row.value, 500);
  });
});

public addRow(): void {
  this._draft.update((v) => ({ ...v, rows: [...v.rows, makeEmptyRow()] }));
}

public removeRow(index: number): void {
  this._draft.update((v) => ({
    ...v,
    rows: v.rows.filter((_, i) => i !== index),
  }));
}
```

In the template, iterate the field tree: `@for (row of form.rows; track row) { <input matInput [formField]="row.value" /> }`.

- 📁 entry editor tests: these pin the behaviors which are easy to break without noticing. Keep the ones of your flavor:

```ts
// __NAME__-editor.component.spec.ts
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { __NAME__EditorComponent } from "./__NAME__-editor.component";

const wait = (ms: number) => new Promise((r) => setTimeout(r, ms));

describe("__NAME__EditorComponent", () => {
  let component: __NAME__EditorComponent;
  let fixture: ComponentFixture<__NAME__EditorComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [__NAME__EditorComponent],
    }).compileComponents();
    fixture = TestBed.createComponent(__NAME__EditorComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it("populates the draft from a bound model, pristine", async () => {
    fixture.componentRef.setInput("data", { type: "t", name: "a", note: "n" });
    fixture.detectChanges();
    await fixture.whenStable();

    expect(component.form.name().value()).toBe("a");
    expect(component.form().dirty()).toBe(false);
  });

  it("refuses to save an invalid draft and shows the errors", () => {
    component.form.name().value.set("");
    component.save();

    expect(component.data()).toBeUndefined();
    expect(component.form.name().touched()).toBe(true);
  });

  it("renders no <form> element", () => {
    expect(fixture.nativeElement.querySelector("form")).toBeNull();
  });

  // ── MANUAL SAVE ONLY ──
  it("saves the trimmed model on save", () => {
    component.form.name().value.set(" b ");
    component.form.name().markAsDirty();
    component.save();

    expect(component.data()?.name).toBe("b");
    expect(component.form().dirty()).toBe(false);
  });

  // ── AUTOSAVE ONLY ──
  it("does not autosave a normalized copy of a value it was just given", async () => {
    fixture.componentRef.setInput("data", { type: "t", name: "  untrimmed  " });
    fixture.detectChanges();
    await fixture.whenStable();

    await wait(600); // past the autosave delay
    fixture.detectChanges();

    expect(component.data()?.name).toBe("  untrimmed  ");
  });

  // ── AUTOSAVE ONLY ──
  it("keeps an edit in progress when its own save comes back normalized", async () => {
    component.form.name().value.set("abc ");
    await wait(600);
    fixture.detectChanges();

    // the model got the trimmed value, but the draft keeps what was typed
    expect(component.data()?.name).toBe("abc");
    expect(component.form.name().value()).toBe("abc ");

    // so that typing on yields "abc d", not "abcd"
    component.form.name().value.set(component.form.name().value() + "d");
    await wait(600);
    fixture.detectChanges();

    expect(component.form.name().value()).toBe("abc d");
    expect(component.data()?.name).toBe("abc d");
  });
});
```
