---
title: "Using Monaco Editor"
parent: "Developing Frontend"
layout: default
nav_order: 12
---

- [Using Monaco Editor](#using-monaco-editor)
  - [Overview](#overview)
  - [1. Setup the Workspace](#1-setup-the-workspace)
    - [1.1. Install Packages](#11-install-packages)
    - [1.2. Add Monaco Assets](#12-add-monaco-assets)
    - [1.3. Provide the Monaco Loader](#13-provide-the-monaco-loader)
    - [1.4. Check TypeScript Configuration](#14-check-typescript-configuration)
    - [1.5. Declare Dependencies in Libraries](#15-declare-dependencies-in-libraries)
  - [2. Add Monaco to an Editor](#2-add-monaco-to-an-editor)
    - [2.1. Code](#21-code)
    - [2.2. Template](#22-template)
    - [2.3. Styles](#23-styles)
    - [2.4. Other Binding Options](#24-other-binding-options)
  - [3. Markdown Preview](#3-markdown-preview)
  - [4. Editing Shortcuts with Text Editing Plugins](#4-editing-shortcuts-with-text-editing-plugins)
    - [4.1. Configure Plugins and Key Bindings](#41-configure-plugins-and-key-bindings)
    - [4.2. Wire Shortcuts in the Editor](#42-wire-shortcuts-in-the-editor)
    - [4.3. Writing Your Own Plugins](#43-writing-your-own-plugins)
  - [Migrating from @cisstech/nge](#migrating-from-cisstechnge)
  - [Troubleshooting](#troubleshooting)

# Using Monaco Editor

## Overview

[Monaco](https://microsoft.github.io/monaco-editor/) is the code editor of VS Code. Cadmus editors use it for long texts, typically Markdown (e.g. the note part, the comment part/fragment, the token text part, and the witnesses fragment).

The Cadmus frontend integrates Monaco via:

- [@jean-merelis/ngx-monaco-editor](https://github.com/jean-merelis/ngx-monaco-editor), a thin Angular wrapper providing the `<ngx-monaco-editor>` component. Its text is a model signal (`[value]`/`(valueChange)`), and it gives you the underlying Monaco editor instance when it is ready.
- optionally, [marked](https://marked.js.org) to render a live Markdown preview of the text.
- optionally, the [Cadmus text editing libraries](https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-text-ed/README.md) (`@myrmidon/cadmus-text-ed`, `-md`, `-txt`), which add editing shortcuts like Ctrl+B for bold, Ctrl+I for italic, Ctrl+E for emoji, Ctrl+L for links to Cadmus entities.

⚠️ This replaces the former `@cisstech/nge` wrapper (`NgeMonacoModule`, `<nge-monaco-editor>`) and its Markdown renderer (`NgeMarkdownModule`). If your workspace still uses them, see [Migrating from @cisstech/nge](#migrating-from-cisstechnge).

All the code below comes from the note part editor (`NotePartComponent` in `@myrmidon/cadmus-part-general-ui`), which you can use as a complete reference.

## 1. Setup the Workspace

You do these steps once per app workspace.

### 1.1. Install Packages

▶️ (1) install Monaco and its wrapper:

```sh
npm i monaco-editor@~0.47.0 @jean-merelis/ngx-monaco-editor
```

⚠️ `monaco-editor` must satisfy the wrapper's peer dependency (`ngx-monaco-editor` 21.x requires `monaco-editor` `^0.47.0`). Do not let npm install the latest Monaco: for `0.x` versions, a caret range allows only patch updates, so `^0.47.0` means `>=0.47.0 <0.48.0`. After installing, check that there is no unmet peer dependency warning for `monaco-editor`.

▶️ (2) optionally, install the Markdown renderer (for a [preview](#3-markdown-preview)):

```sh
npm i marked
```

▶️ (3) optionally, install the text editing libraries (for [shortcuts](#4-editing-shortcuts-with-text-editing-plugins)):

```sh
npm i @myrmidon/cadmus-text-ed @myrmidon/cadmus-text-ed-md @myrmidon/cadmus-text-ed-txt
```

`@myrmidon/cadmus-text-ed` contains the editing service; `-md` contains the Markdown plugins (bold, italic, link), and `-txt` the plain text plugins (emoji). The link plugin also requires `@myrmidon/cadmus-refs-asserted-ids` and Angular Material dialogs.

### 1.2. Add Monaco Assets

The wrapper loads Monaco at runtime from the `vs` folder of the app. So, Monaco's files must be copied there when building.

▶️ in `angular.json`, add this entry to the `assets` array of your **app**'s `build` target (`projects.<app>.architect.build.options.assets`), next to the existing entries:

```json
{
  "glob": "**/*",
  "input": "node_modules/monaco-editor/min/vs",
  "output": "vs"
}
```

### 1.3. Provide the Monaco Loader

▶️ in your app configuration (`app.config.ts`), add the loader provider:

```ts
import {
  DefaultMonacoLoader,
  NGX_MONACO_LOADER_PROVIDER,
} from '@jean-merelis/ngx-monaco-editor';

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    {
      provide: NGX_MONACO_LOADER_PROVIDER,
      useFactory: () => new DefaultMonacoLoader(),
    },
  ],
};
```

`DefaultMonacoLoader` loads Monaco from the `vs` path set up in [1.2](#12-add-monaco-assets). If you serve the assets from another path, pass it to the loader: `new DefaultMonacoLoader({ paths: { vs: 'path/to/vs' } })`.

### 1.4. Check TypeScript Configuration

The wrapper does not use the global `monaco` namespace: it exports the types you need (`StandaloneCodeEditor`, `StandaloneEditorConstructionOptions`, `EditorInitializedEvent`).

▶️ if `tsconfig.app.json` (or any library's `tsconfig.lib.json`) has `"monaco-editor"` in its `types` array, remove it. In your code, use the wrapper's types instead of `monaco.editor.*`.

### 1.5. Declare Dependencies in Libraries

If your Monaco-based editors live in a library (as usual for Cadmus part and fragment editors), declare what the library uses:

▶️ (1) in the library's `package.json`, add the wrapper (and the text editing libraries, if used) to `peerDependencies`, and `marked` (if used) to `dependencies`:

```json
{
  "peerDependencies": {
    "@jean-merelis/ngx-monaco-editor": "^21.0.0",
    "@myrmidon/cadmus-text-ed": "^11.0.2"
  },
  "dependencies": {
    "marked": "^18.0.14",
    "tslib": "^2.3.0"
  }
}
```

▶️ (2) if you added `marked` to `dependencies`, allow it in the library's `ng-package.json`, else the library build fails:

```json
{
  "allowedNonPeerDependencies": ["marked"]
}
```

## 2. Add Monaco to an Editor

These steps show a Cadmus part editor with a Markdown text edited in Monaco. A part editor uses a signal form: see [creating parts](app-parts) for the rest of its code. The same applies to fragment editors and to any other component.

### 2.1. Code

▶️ (1) import the wrapper's component and types, and add `NgxMonacoEditorComponent` to the component's `imports`:

```ts
import {
  EditorInitializedEvent,
  NgxMonacoEditorComponent,
  StandaloneCodeEditor,
  StandaloneEditorConstructionOptions,
} from '@jean-merelis/ngx-monaco-editor';
import { setFieldFromEditor } from '@myrmidon/cadmus-ui';

@Component({
  // ...
  imports: [
    // ...
    NgxMonacoEditorComponent,
  ],
})
```

▶️ (2) add a draft field for the text, as for any other control (e.g. `text: string` in your draft interface, with `text: part?.text || ''` in `toDraft()` and its rules in the form's schema).

▶️ (3) add the editor options, the helper used to bind the text, and (if you need the editor instance, e.g. for [shortcuts](#4-editing-shortcuts-with-text-editing-plugins)) a field for the editor:

```ts
public readonly editorOptions: StandaloneEditorConstructionOptions = {
  minimap: { side: 'right' },
  wordWrap: 'on',
  automaticLayout: true,
};

/**
 * Set a text field from its editor. See setFieldFromEditor.
 */
public readonly setFieldFromEditor = setFieldFromEditor;

private _editor?: StandaloneCodeEditor;

public onEditorInit(event: EditorInitializedEvent): void {
  this._editor = event.editor;
  this._editor.focus();
}
```

`editorOptions` accepts any [Monaco editor option](https://microsoft.github.io/monaco-editor/docs.html#interfaces/editor.IStandaloneEditorConstructionOptions.html), except `value`, `language` and `theme`, which are separate inputs of the component. Keep `automaticLayout: true`, so that the editor follows the size of its container.

You no longer need to create a Monaco text model, to set its value when data arrives, or to dispose anything: the wrapper does all this.

When reading the text back (e.g. in `getValue()`), just read the draft field, like any other control.

### 2.2. Template

▶️ add the editor in a sized container:

```html
<div id="editor">
  <ngx-monaco-editor
    [value]="form.text().value()"
    (valueChange)="setFieldFromEditor(form.text, $event)"
    (blur)="form.text().markAsTouched()"
    [language]="'markdown'"
    [options]="editorOptions"
    (editorInitialized)="onEditorInit($event)"
  />
</div>
@if (
  form.text().getError("required") &&
  (form.text().touched() || form.text().dirty())
) {
  <mat-error>please enter some text</mat-error>
}
```

⚠️ Do **not** bind Monaco with `[formField]="form.text"`. Monaco reports any text it receives as a user change, also when it is set by code (e.g. when the part is loaded, or saved with trimmed text). With `[formField]`, this would make the editor dirty with no user change, and the user would be asked to save pending changes when closing it. Instead:

- `[value]` passes the field's text to Monaco;
- `(valueChange)` calls `setFieldFromEditor` (from `@myrmidon/cadmus-ui`), which ignores a text equal to the field's, and else sets the field and marks it as dirty;
- `(blur)` marks the field as touched, so that its errors are displayed.

Other inputs and outputs of `<ngx-monaco-editor>`:

- `language`: the Monaco language ID (e.g. `markdown`, `plaintext`, `json`, `xml`).
- `theme`: the Monaco theme (e.g. `vs`, `vs-dark`).
- `(focus)`, `(blur)`: emitted when the editor gets or loses focus.
- `(editorInitialized)`: emitted once with an `EditorInitializedEvent`, whose `editor` is the Monaco editor instance.

### 2.3. Styles

⚠️ Do not skip this: the wrapper's host element is a block with relative position, and the Monaco container inside it has absolute position. So the host gets no height from its content, and without explicit sizing the editor collapses to zero height, even inside a sized container.

▶️ give the container a height, and let the editor fill it:

```css
div#editor {
  height: 600px; /* or any sizing, e.g. 100% of a flex child */
}
div#editor ngx-monaco-editor {
  display: block;
  width: 100%;
  height: 100%;
}
```

### 2.4. Other Binding Options

The text is a model signal, so outside signal forms you can bind it as you prefer:

- to a plain writable signal, with two-way binding: `<ngx-monaco-editor [(value)]="text" />`.
- to a reactive form control (legacy code): the component is also a form control, so `<ngx-monaco-editor [formControl]="text" />` works. Note that the same issue about text set by code applies here: you might get a dirty control after loading data.

## 3. Markdown Preview

To show a live preview of the Markdown text, render it with `marked` into a sanitized HTML signal.

▶️ (1) in the component code:

```ts
import { inject, signal } from '@angular/core';
import { takeUntilDestroyed, toObservable } from '@angular/core/rxjs-interop';
import { DomSanitizer, SafeHtml } from '@angular/platform-browser';
import { debounceTime } from 'rxjs/operators';
import { marked } from 'marked';

// in the component class:
private readonly _sanitizer = inject(DomSanitizer);
public readonly previewHtml = signal<SafeHtml>('');

constructor() {
  super();
  // update the preview whenever the text changes, both by user edits and
  // by loading data
  toObservable(this.form.text().value)
    .pipe(debounceTime(50), takeUntilDestroyed())
    .subscribe((text) => this.updatePreview(text));
}

private updatePreview(text: string): void {
  const html = marked.parse(text || '', { async: false }) as string;
  this.previewHtml.set(this._sanitizer.bypassSecurityTrustHtml(html));
}
```

▶️ (2) in the template:

```html
<div class="preview" [innerHTML]="previewHtml()"></div>
```

💡 The note part places the editor and the preview side by side, and stacks them on narrow screens. See its template and styles for a ready-made layout.

## 4. Editing Shortcuts with Text Editing Plugins

The [Cadmus text editing service](https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-text-ed/README.md) (`CadmusTextEdService`) hosts a set of plugins. Each plugin receives a text (here, the text selected in Monaco) and returns it edited, e.g.:

| plugin             | package                       | ID          | does                                                                  |
| ------------------ | ----------------------------- | ----------- | --------------------------------------------------------------------- |
| `MdBoldCtePlugin`   | `@myrmidon/cadmus-text-ed-md`  | `md.bold`   | toggle Markdown bold.                                                  |
| `MdItalicCtePlugin` | `@myrmidon/cadmus-text-ed-md`  | `md.italic` | toggle Markdown italic.                                                |
| `MdLinkCtePlugin`   | `@myrmidon/cadmus-text-ed-md`  | `md.link`   | insert or edit a link to a Cadmus entity, picked in a dialog.          |
| `TxtEmojiCtePlugin` | `@myrmidon/cadmus-text-ed-txt` | `txt.emoji` | insert a Unicode emoji, picked in a dialog.                            |

You bind each plugin to a key combination in Monaco: when the user presses it, your component sends the selected text to the service, and replaces the selection with the result.

### 4.1. Configure Plugins and Key Bindings

You usually configure plugins and their key bindings once for the whole app.

▶️ in `app.config.ts`, provide the plugins, the service options (`CADMUS_TEXT_ED_SERVICE_OPTIONS_TOKEN`), and the key bindings (`CADMUS_TEXT_ED_BINDINGS_TOKEN`):

```ts
import {
  CADMUS_TEXT_ED_BINDINGS_TOKEN,
  CADMUS_TEXT_ED_SERVICE_OPTIONS_TOKEN,
} from '@myrmidon/cadmus-text-ed';
import {
  MdBoldCtePlugin,
  MdItalicCtePlugin,
  MdLinkCtePlugin,
} from '@myrmidon/cadmus-text-ed-md';
import { TxtEmojiCtePlugin } from '@myrmidon/cadmus-text-ed-txt';

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    // text editor plugins
    MdBoldCtePlugin,
    MdItalicCtePlugin,
    TxtEmojiCtePlugin,
    MdLinkCtePlugin,
    {
      provide: CADMUS_TEXT_ED_SERVICE_OPTIONS_TOKEN,
      useFactory: (
        mdBoldCtePlugin: MdBoldCtePlugin,
        mdItalicCtePlugin: MdItalicCtePlugin,
        txtEmojiCtePlugin: TxtEmojiCtePlugin,
        mdLinkCtePlugin: MdLinkCtePlugin,
      ) => {
        return {
          plugins: [
            mdBoldCtePlugin,
            mdItalicCtePlugin,
            txtEmojiCtePlugin,
            mdLinkCtePlugin,
          ],
        };
      },
      deps: [
        MdBoldCtePlugin,
        MdItalicCtePlugin,
        TxtEmojiCtePlugin,
        MdLinkCtePlugin,
      ],
    },
    // monaco bindings for plugins
    // 2080 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyB;
    // 2087 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyI;
    // 2083 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyE;
    // 2090 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyL;
    {
      provide: CADMUS_TEXT_ED_BINDINGS_TOKEN,
      useValue: {
        2080: 'md.bold', // Ctrl+B
        2087: 'md.italic', // Ctrl+I
        2083: 'txt.emoji', // Ctrl+E
        2090: 'md.link', // Ctrl+L
      },
    },
  ],
};
```

The bindings map Monaco key codes to plugin IDs. Key codes are numbers, combining Monaco's `KeyMod` and `KeyCode` values: `KeyMod.CtrlCmd` is 2048, and `KeyCode.KeyA`...`KeyZ` are 31...56, so e.g. Ctrl+B is 2048 + 32 = 2080. Use numbers directly: the wrapper does not expose Monaco's `KeyMod` and `KeyCode` enums.

💡 Instead of a global configuration, you can configure a single component's service by calling its `configure` method, creating each plugin with `inject()` (not `new`) so that its dependencies are injected: `this._editService.configure({ plugins: [inject(MdBoldCtePlugin)] })`.

### 4.2. Wire Shortcuts in the Editor

▶️ (1) provide the service in your component, and inject it with the key bindings. The service is **not** a singleton: each component gets its own instance, configured with the global options.

```ts
import {
  CadmusTextEdService,
  CADMUS_TEXT_ED_BINDINGS_TOKEN,
  CadmusTextEdBindings,
} from '@myrmidon/cadmus-text-ed';

@Component({
  // ...
  providers: [CadmusTextEdService],
})
export class MyPartComponent extends ModelEditorComponentBase<MyPart> {
  private readonly _editService = inject(CadmusTextEdService);
  // the bindings, if provided in the app configuration
  private readonly _editorBindings = inject(CADMUS_TEXT_ED_BINDINGS_TOKEN, {
    optional: true,
  }) as CadmusTextEdBindings | null;
  // ...
}
```

(The cast is needed because the token is declared with a different type parameter.)

▶️ (2) when the editor is ready, add a command for each binding:

```ts
public onEditorInit(event: EditorInitializedEvent): void {
  this._editor = event.editor;

  // bind each key code to its plugin
  if (this._editorBindings) {
    Object.keys(this._editorBindings).forEach((key) => {
      const keyCode = parseInt(key, 10);
      this._editor!.addCommand(keyCode, () => {
        this.applyEdit(this._editorBindings![keyCode]);
      });
    });
  }

  this._editor.focus();
}
```

▶️ (3) add the function applying a plugin to the selected text:

```ts
private async applyEdit(selector: string): Promise<void> {
  const editor = this._editor;
  if (!editor) {
    return;
  }
  const selection = editor.getSelection();
  const text = selection ? editor.getModel()!.getValueInRange(selection) : '';

  const result = await this._editService.edit({ selector, text });
  if (result.error) {
    console.warn(`Text edit "${selector}" failed: ${result.error}`);
    return;
  }

  editor.executeEdits('cadmus-text-ed', [
    {
      range: selection!,
      text: result.text,
      forceMoveMarkers: true,
    },
  ]);
}
```

The edit changes Monaco's text, so Monaco emits `valueChange`, and the form field is updated (and marked as dirty) as for any typing.

💡 `@myrmidon/cadmus-part-general-ui` exports `MonacoEditorHelper`, a small class which stores the editor instance (`initEditor(event)`) and adds the bindings (`addBindings(bindings, applyEdit)`). Use it if your library already depends on the general parts library; else, the code above is all you need.

### 4.3. Writing Your Own Plugins

A plugin is an injectable class implementing `CadmusTextEdPlugin`: it has an `id` (used as the selector in bindings), some metadata, an `enabled` flag, and two functions: `matches(query)`, and `edit(query)`, which returns a promise with the edited text. Provide it like the stock plugins, add it to the options factory, and bind it to a key code. See the [library's documentation](https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-text-ed/README.md) and the stock plugins' code for examples.

## Migrating from @cisstech/nge

If your workspace still uses `@cisstech/nge`:

1. find all its usages: search for `@cisstech/nge`, `NgeMonacoModule`, `NgeMarkdownModule`, `nge-monaco-editor`, `nge-markdown`, and `monaco.editor.`.
2. packages: uninstall `@cisstech/nge`, and follow [1.1](#11-install-packages) (pinning `monaco-editor` to the wrapper's peer version).
3. configuration: in `app.config.ts`, remove `importProvidersFrom(NgeMonacoModule.forRoot({}))` and add the loader provider ([1.3](#13-provide-the-monaco-loader)); add the assets to `angular.json` ([1.2](#12-add-monaco-assets)); remove `monaco-editor` from `types` in tsconfig files ([1.4](#14-check-typescript-configuration)).
4. in each component:
   - replace `NgeMonacoModule` with `NgxMonacoEditorComponent` in `imports`;
   - replace `monaco.editor.IStandaloneCodeEditor`/`IEditor` with `StandaloneCodeEditor`;
   - remove the text model field, `createModel`/`setModel` calls, the `onDidChangeContent` handler, the code setting the model's value when data arrives, and the disposables: binding the text replaces all of them ([2.2](#22-template));
   - replace `<nge-monaco-editor (ready)="onEditorInit($event)">` with `<ngx-monaco-editor>` as in [2.2](#22-template), and change `onEditorInit` to receive an `EditorInitializedEvent` (the editor is `event.editor`);
   - keep the plugin code (`applyEdit` works the same on the new editor instance);
   - add the CSS of [2.3](#23-styles);
   - replace `<nge-markdown [data]="...">` with a `marked` preview ([3](#3-markdown-preview)).
5. libraries: update their `package.json` and `ng-package.json` ([1.5](#15-declare-dependencies-in-libraries)).
6. verify: search again for `@cisstech/nge` (it must not appear anywhere, including the lock file), build the libraries and the app, and check each editor in the browser (see below).

## Troubleshooting

- **The editor is not visible, or has zero height**: add the CSS of [2.3](#23-styles).
- **The editor never appears, and the browser console shows a 404 for `vs/loader.js`**: the Monaco assets are not copied: check `angular.json` ([1.2](#12-add-monaco-assets)), and restart `ng serve` after changing it.
- **Peer dependency warnings for `monaco-editor`**: install the Monaco version required by the wrapper ([1.1](#11-install-packages)).
- **`TS2503: Cannot find namespace 'monaco'`**: use the wrapper's types instead of the global `monaco` namespace ([1.4](#14-check-typescript-configuration)).
- **The editor is dirty right after loading data, or after saving**: Monaco is bound with `[formField]` (or a reactive form control); bind it as in [2.2](#22-template).
- **Shortcuts do nothing**: check that the component provides `CadmusTextEdService`, that the bindings are provided in the app configuration, and that each binding's plugin ID matches a plugin in the options.
