---
title: "Creating Help"
parent: "Developing Backend"
layout: default
nav_order: 9
---

# Creating Help

- [Creating Help](#creating-help)
  - [Layered Help](#layered-help)
  - [URL Templates](#url-templates)
  - [Search Order](#search-order)
  - [Scenarios](#scenarios)
  - [Writing Help Sites](#writing-help-sites)
  - [Development](#development)

Each part or fragment editor can show a help button in its header, opening the help page for the edited part or fragment type in a new tab. Help pages are just a web site: typically, a GitHub repository with a Markdown document for each part or fragment editor, published as HTML pages via Jekyll (GitHub Pages). You just tell your Cadmus editor how to build the URLs of those pages, and it does the rest. When no page is found for an editor, no help button is shown.

## Layered Help

As Cadmus is modular, most projects share many editors (e.g. those of the general parts). So, rather than requiring each project to document all its editors, help can be built from **layers**:

- a **general-purpose help site**, documenting the editors of the shared libraries. It is valid for all the Cadmus projects, so any project using these editors gets their help for free.
- an optional **project-specific help site**, documenting only the editors which are specific to the project, or the shared editors which need a different help in it. Its pages override those of the general-purpose site.
- optional **localized help sites**: a copy of a help site translated into another language. Translation can be partial: for the pages which are not translated, the default (usually English) help is used.

Within each site, help can also be differentiated by **role**: you can write a single page for a part type, and add role-specific pages only where needed.

So, for each editor the help system tries a list of candidate URLs, and uses the first page which exists. You only need to configure the URL templates and, optionally, the language.

## URL Templates

The URLs of help pages are built from one or more **templates**, which can include these **placeholders**. Their values are URL-encoded; an empty value is replaced by nothing:

| placeholder   | part editor        | fragment editor                               |
| ------------- | ------------------ | --------------------------------------------- |
| `{typeId}`    | part type ID       | fragment type ID (e.g. `fr.it.vedph.comment`) |
| `{roleId}`    | part role ID       | always empty                                  |
| `{frRoleId}`  | always empty       | fragment role ID                              |
| `{language}`  | help language code | help language code                            |
| `{separator}` | the separator      | the separator                                 |

**Optional groups** in square brackets are output (without brackets) only when all the placeholders inside them have a value; otherwise, the whole group is dropped. `{separator}` immediately followed by a placeholder is a shortcut for such a group, so `{separator}{roleId}` is the same as `[{separator}{roleId}]`. Groups cannot be nested.

For instance, with template `https://www.mysite.com/help/topics/{typeId}{separator}{roleId}.html` and the default separator `__`, the note part with role `history` gets `.../topics/it.vedph.note__history.html`, and without role `.../topics/it.vedph.note.html`.

⚠️ Always put `{language}` in an optional group together with its delimiter, so that the URL without language is well-formed: e.g. `help[-{language}]/` gives `help-it/` or `help/`; `help/[{language}/]` gives `help/it/` or `help/`. A bare `help-{language}/` would give `help-/` for the default help.

The **language** is either set explicitly (e.g. `it`), or taken from the browser (`auto`). Language codes (typically [BCP47](https://www.rfc-editor.org/info/bcp47)) are normalized to lowercase, with `_` replaced by `-`. When a code has more subtags, they are removed one by one from the end: e.g. a browser set to `it-IT` tries `it-it` and then `it`. So, name localized sites with the lowercase primary language code (e.g. `help-it`), unless you need regional variants.

Other template styles are possible:

- one page per type, with a **bookmark** per role: `https://www.mysite.com/help/{typeId}.html[#{roleId}][#{frRoleId}]`. Bookmarks do not affect the availability check, so the check is done on the type page only.
- **query string**: `https://www.mysite.com/help?type={typeId}[&role={roleId}][&frRole={frRoleId}][&lang={language}]`.
- language as a **path segment**: `https://www.mysite.com/help/[{language}/]{typeId}.html`.
- Jekyll with pretty **permalinks**: `https://www.mysite.com/help/{typeId}[-{roleId}][-{frRoleId}]/`.

## Search Order

The candidate URLs are tried in an order following this rule: **the most relevant content wins; language is preferred only among equally relevant pages**. So, candidates are built:

1. for each **template**, in their order. So, list the project-specific site first, and the general-purpose site last: a page found in the project site always wins.
2. for each **topic**, from the most specific to the least specific: first with all the IDs, then without `{frRoleId}`, then without `{roleId}`. So, a page for the type ID is used for all its roles, unless a role-specific page exists.
3. for each **language**, from the most specific to the least specific (e.g. `it-it`, then `it`), and finally with no language (the default help).

Duplicate URLs are removed: e.g. when a template has no `{language}`, or there is no role, fewer candidates are tried.

Two consequences of this rule are worth noting:

- once a project site documents a type, it **owns** all of its help: a project page for the note part overrides also the role-specific note pages of the general-purpose site.
- an English role-specific page wins over a translated generic page, because it is more relevant. So, when translating, translate role-specific pages together with their generic page.

When no page is found, the URLs tried are logged in the browser console (as info), so you can see which topics are missing. Pages are checked with anonymous requests (no credentials are sent), and results are cached for the session, so each URL is requested at most once.

> The help sites must allow cross-origin requests (CORS); GitHub Pages does. If your site does not, you can disable checks (`helpUrlCheck`): then the first candidate URL is always used, even if it does not exist. As fallbacks require checks, in this case use a single template with no `{language}`.

## Scenarios

In these examples the separator is the default (`__`).

(1) **single help site**, with template `https://myproject.github.io/help/topics/{typeId}{separator}{roleId}{separator}{frRoleId}.html`:

| editor                           | URLs tried, in order                                                              |
| -------------------------------- | --------------------------------------------------------------------------------- |
| note part, no role               | `.../topics/it.vedph.note.html`                                                   |
| note part, role `history`        | `.../topics/it.vedph.note__history.html`, `.../topics/it.vedph.note.html`         |
| comment fragment, fr. role `sch` | `.../topics/fr.it.vedph.comment__sch.html`, `.../topics/fr.it.vedph.comment.html` |

(2) **project site with fallback to the general-purpose site**, with templates `https://myproject.github.io/help/topics/{typeId}{separator}{roleId}.html` and `https://cadmus.github.io/help/topics/{typeId}{separator}{roleId}.html`. For the note part with role `history`:

1. `https://myproject.github.io/help/topics/it.vedph.note__history.html`
2. `https://myproject.github.io/help/topics/it.vedph.note.html`
3. `https://cadmus.github.io/help/topics/it.vedph.note__history.html`
4. `https://cadmus.github.io/help/topics/it.vedph.note.html`

(3) **localized help**, with template `https://cadmus.github.io/help[-{language}]/topics/{typeId}{separator}{roleId}.html` and language `it`. For the note part with role `history`:

1. `https://cadmus.github.io/help-it/topics/it.vedph.note__history.html`
2. `https://cadmus.github.io/help/topics/it.vedph.note__history.html`
3. `https://cadmus.github.io/help-it/topics/it.vedph.note.html`
4. `https://cadmus.github.io/help/topics/it.vedph.note.html`

Without a language, only (2) and (4) are tried.

(4) **both**, with the templates of the [development](#development) section below, language `auto` and a browser set to `it-IT`. For the note part with no role:

1. `https://myproject.github.io/help-it-it/topics/it.vedph.note.html`
2. `https://myproject.github.io/help-it/topics/it.vedph.note.html`
3. `https://myproject.github.io/help/topics/it.vedph.note.html`
4. `https://cadmus.github.io/help-it-it/topics/it.vedph.note.html`
5. `https://cadmus.github.io/help-it/topics/it.vedph.note.html`
6. `https://cadmus.github.io/help/topics/it.vedph.note.html`

## Writing Help Sites

With GitHub Pages and Jekyll, each Markdown file becomes an HTML page with the same name, and a repository named `help` in the GitHub account `cadmus` is published at `https://cadmus.github.io/help/`.

- **topics**: with the templates above, create a Markdown file for each part/fragment type, named after its type ID (e.g. `topics/it.vedph.note.md`, `topics/fr.it.vedph.comment.md`). For role-specific help, add a file named after type ID, separator and role ID (e.g. `topics/it.vedph.note__history.md`, `topics/fr.it.vedph.comment__sch.md`). Avoid characters in role IDs which are not valid in file names; when needed, pick a different separator or template.
- **general-purpose site**: document the editors of the shared libraries (e.g. general and philology parts).
- **project site**: document only the project-specific editors, and the shared ones which need a different help in your project.
- **localized site**: copy the files of a site into a new repository, named like the original one plus the language code (e.g. `help-it`), translate them (all or just some), and publish it.

## Development

To enable contextual help in your Cadmus editor:

▶️ (1) add these environment parameters to `env.js`:

```js
// URL template(s) for help pages, in their search order
// TODO: replace with your own URLs...
window.__env.helpUrlTemplate = [
  "https://myproject.github.io/help[-{language}]/topics/{typeId}{separator}{roleId}{separator}{frRoleId}.html",
  "https://cadmus.github.io/help[-{language}]/topics/{typeId}{separator}{roleId}{separator}{frRoleId}.html",
];
// value of {language} (optional): a code like "it", or "auto" for the browser language
window.__env.helpUrlLanguage = "auto";
// value of {separator} (optional, default: __)
window.__env.helpUrlSeparator = "__";
// false to skip checking page availability (optional, default: true)
window.__env.helpUrlCheck = true;
```

- `helpUrlTemplate` is required: without it, no help is available. It can be a single template, or an array of templates in their search order. A string with several templates separated by spaces or newlines is accepted too, which is handy when `env.js` is generated from environment variables (e.g. in Docker).
- `helpUrlLanguage` is optional: when not set, only the default help is used. Invalid codes are ignored.

>To get the help language from another source (e.g. a language setting of your app), derive a service from `EditorHelpService` (`@myrmidon/cadmus-ui`), override its `getLanguages()` method (which returns the language codes to try, from the most specific to the least specific), and provide it in place of `EditorHelpService`.

▶️ (2) import `HelpLinkComponent` into your part/fragment editor:

```ts
import {
  CloseSaveButtonsComponent,
  ModelEditorComponentBase,
  // ADD THIS:
  HelpLinkComponent,
} from '@myrmidon/cadmus-ui';
```

▶️ (3) ensure that your part/fragment editor template contains the help component:

```html
<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>picture_in_picture</mat-icon>
      </div>
      <mat-card-title>{{
        (modelName() | titlecase) || "Some Part"
      }}</mat-card-title>
      <!-- ADD THIS -->
      <cadmus-help-link [url]="helpUrl()" />
    </mat-card-header>
    ...
  </mat-card>
</form>
```
