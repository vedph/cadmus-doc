---
title: "Using Monaco Editor"
parent: "Developing Frontend"
layout: default
nav_order: 11
---

- [Using Monaco Editor](#using-monaco-editor)
  - [Setup](#setup)
  - [Usage](#usage)

# Using Monaco Editor

To make it easier to integrate Monaco in your editor, you can use [this thin wrapper](https://cisstech.github.io/nge/docs/nge-monaco/getting-started).

## Setup

1. `npm i @cisstech/nge`.
2. add `importProvidersFrom(NgeMonacoModule.forRoot({})),` in the providers of `appConfig`.

If using [CTE plugins](https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-text-ed/README.md), add to your `app.config` the required plugins, e.g.:

```ts
import {
  CADMUS_TEXT_ED_SERVICE_OPTIONS_TOKEN,
  CADMUS_TEXT_ED_BINDINGS_TOKEN,
} from '@myrmidon/cadmus-text-ed';
import {
  MdBoldCtePlugin,
  MdItalicCtePlugin,
  MdLinkCtePlugin,
} from '@myrmidon/cadmus-text-ed-md';
import { TxtEmojiCtePlugin } from '@myrmidon/cadmus-text-ed-txt';

export const appConfig: ApplicationConfig = {
  providers: [
    //...
    // text editor plugins
    // https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-text-ed/README.md
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
        mdLinkCtePlugin: MdLinkCtePlugin
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
  ]
};
```

## Usage

1. in your component add these fields:

    ```ts
    import { NgeMonacoModule } from '@cisstech/nge/monaco';
    // ...

    @Component({
    selector: '...',
    imports: [
    NgeMonacoModule
    // ...
    ]
    })
    export class MyComponent implements OnDestroy {
    private _editorModel?: monaco.editor.ITextModel;
    private _editor?: monaco.editor.IStandaloneCodeEditor;
    // ...
    }
    ```

2. if using CTE plugins, inject in the component constructor:

    ```ts
    constructor(
        private _editService: CadmusTextEdService,
        @Inject(CADMUS_TEXT_ED_BINDINGS_TOKEN)
        @Optional()
        private _editorBindings?: CadmusTextEdBindings
    )
    ```

3. add to your component's form a `FormControl<string|null>` control to hold the text to edit, just like any other form element. This will be kept in synch with the Monaco editor. For instance:

    ```ts
    this.description = formBuilder.control(null, {
    validators: Validators.maxLength(10000),
    });
    this.form = formBuilder.group({
    description: this.description,
    // ...
    });
    ```

4. add code for initializing the editor:

    ```ts
    private updateEditorContent(description: string | null) {
        if (this._editorModel) {
        this._editorModel.setValue(description || '');
        }
    }

    public onEditorInit(editor: monaco.editor.IEditor) {
        editor.updateOptions({
        minimap: {
            side: 'right',
        },
        wordWrap: 'on',
        automaticLayout: true,
        });
        this._editorModel =
        this._editorModel || monaco.editor.createModel('', 'markdown');
        editor.setModel(this._editorModel);
        this._editor = editor as monaco.editor.IStandaloneCodeEditor;

        this._disposables.push(
        this._editorModel.onDidChangeContent((e) => {
            this.description.setValue(this._editorModel!.getValue());
            this.description.markAsDirty();
            this.description.updateValueAndValidity();
        })
        );

        // plugins
        if (this._editorBindings) {
        Object.keys(this._editorBindings).forEach((key) => {
            const n = parseInt(key, 10);
            console.log(
            'Binding ' + n + ' to ' + this._editorBindings![key as any]
            );
            this._editor!.addCommand(n, () => {
            this.applyEdit(this._editorBindings![key as any]);
            });
        });
        }

        // update the editor content if the description is already available
        this.updateEditorContent(this.description.value);
    }
    ```

5. in the form update code, called when data is bound to your component, update both the control and Monaco editor:

    ```ts
    private updateForm(sign: EpiSign | undefined | null): void {
        if (!sign) {
        this.form.reset();
        return;
        }

        // ...
        this.description.setValue(sign.description || null);
        this.form.markAsPristine();

        this.updateEditorContent(sign.description || null);
    }
    ```

6. when getting your text back, just read it from your control's value (here `this.description.value`).

    👉 Note that we are using `updateEditorContent` both in `updateForm` and in editor init. This is because typically the editor's text is set when data is bound to an input endpoint of the component, which typically is a signal got via `model` or `input`, so that `updateForm` is called in an effect like:

    ```ts
    effect(() => {
    this.updateForm(this.sign());
    });
    ```

    Thus, it might happen that when `updateForm` gets called in the effect, the Monaco editor is not yet initialized. Setting the content in both these places ensures that we set it later if Monaco wasn't ready yet.

7. in your template, add the editor and bind its `ready` event:

    ```hml
    <div id="editor">
        <nge-monaco-editor
        style="--editor-height: 100%"
        (ready)="onEditorInit($event)"
        />
    </div>
    ```

8. remember to destroy disposables:

```ts
public ngOnDestroy() {
  this._disposables.forEach((d) => d.dispose());
}
```
