---
title: "Creating Help"
parent: "Creating Backend Core"
layout: default
nav_order: 10
---

# Creating Help

You can create a web site with help for each part or fragment editor in your project, optionally differentiating pages according to roles. For instance, you can create a GitHub repository and add a Markdown document for each part or fragment editor you want to document. Then, generate the corresponding HTML pages via Jekyll and provide the base URL to the resulting site to your Cadmus editor.

## URL Template

The URL to topic pages is provided with a template.

The template can include these **placeholders**. Their values are URL-encoded; an empty value is replaced by nothing:

| placeholder   | part editor   | fragment editor                               |
| ------------- | ------------- | --------------------------------------------- |
| `{typeId}`    | part type ID  | fragment type ID (e.g. `fr.it.vedph.comment`) |
| `{roleId}`    | part role ID  | always empty                                  |
| `{frRoleId}`  | always empty  | fragment role ID                              |
| `{separator}` | the separator | the separator                                 |

- **Optional groups** in square brackets are output (without brackets) only when all the placeholders inside them have a value; otherwise, the whole group is dropped. `{separator}` immediately followed by a placeholder is a shortcut for such a group, so `{separator}{roleId}` is the same as `[{separator}{roleId}]`. Groups cannot be nested.
- **Fallback**: when `helpUrlCheck` is not false, candidate URLs are tried from the most specific to the least specific: first with all the values, then without `{frRoleId}`, then without `{roleId}`. The first available page is used. So you can write a single page for a type ID, and add role-specific pages only where needed. When no page is found, the URLs tried are logged in the browser console (as info), so you can see which topics are missing. Checks use `fetch` (so no credentials are sent) and their results are cached for the session.

> The help site must allow cross-origin requests (CORS); GitHub Pages does. If your site does not, set `helpUrlCheck` to false: the most specific URL is then always used, without any check.

**Examples** using template `https://www.mysite.com/help/topics/{typeId}{separator}{roleId}{separator}{frRoleId}.html`:

- note part with no role: `.../topics/it.vedph.note.html`.
- note part with role `history`: `.../topics/it.vedph.note__history.html`, falling back to `.../topics/it.vedph.note.html`.
- comment fragment with role `sch`: `.../topics/fr.it.vedph.comment__sch.html`, falling back to `.../topics/fr.it.vedph.comment.html`.

**Other template styles**:

- one page per type, with a **bookmark** per role: `https://www.mysite.com/help/{typeId}.html[#{roleId}][#{frRoleId}]`. Bookmarks do not affect the availability check, so the check is done on the type page only.
- **query string**: `https://www.mysite.com/help?type={typeId}[&role={roleId}][&frRole={frRoleId}]`.
- Jekyll with pretty **permalinks**: `https://www.mysite.com/help/{typeId}[-{roleId}][-{frRoleId}]/`.

**Topic naming with Jekyll** (e.g. GitHub Pages): with the first template above, create one Markdown file for each part/fragment type, named after its type ID (e.g. `topics/it.vedph.note.md`, `topics/fr.it.vedph.comment.md`).
For role-specific help, add a file named after type ID, separator and role ID (e.g. `topics/it.vedph.note__history.md`, `topics/fr.it.vedph.comment__sch.md`). Avoid characters in role IDs which are not valid in file names; when needed, pick a different separator or template.

## Development

To enable contextual help in your Cadmus editor:

1. add these environment parameters to `env.js`:

```js
// URL template for help pages
// TODO: replace with your own URL...
window.__env.helpUrlTemplate =
  "https://www.mysite.com/help/topics/{typeId}{separator}{roleId}{separator}{frRoleId}.html";
// value of {separator} (optional, default: __)
window.__env.helpUrlSeparator = "__";
// false to skip checking page availability (optional, default: true)
window.__env.helpUrlCheck = true;
```

2. ensure that your part/fragment editor template contains the help component:

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
