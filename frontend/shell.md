---
title: "Cadmus Shell" 
layout: default 
---

# Shell

Cadmus shell is the main repository for Cadmus frontend. It contains the editor's core components, and the most used part libraries (general and philology). Other shells exist for more specific knowledge domains.

- 🌐 <https://github.com/vedph/cadmus-shell-v3>

```mermaid
graph LR;
  cadmus-core --> ngx-tools
  cadmus-api --> ngx-tools
  cadmus-api --> auth-jwt-login
  cadmus-api --> cadmus-core
  cadmus-flags-pg --> auth-jwt-login
  cadmus-flags-pg --> cadmus-flags-ui
  cadmus-flags-ui --> ngx-tools
  cadmus-flags-ui --> ngx-mat-tools
  cadmus-flags-ui --> cadmus-state
  cadmus-graph-pg --> ngx-tools
  cadmus-graph-pg --> ngx-mat-tools
  cadmus-graph-pg --> cadmus-api
  cadmus-graph-pg --> cadmus-graph-ui
  cadmus-graph-pg-ex --> ngx-tools
  cadmus-graph-pg-ex --> ngx-mat-tools
  cadmus-graph-pg-ex --> cadmus-refs-lookup
  cadmus-graph-pg-ex --> cadmus-api
  cadmus-graph-pg-ex --> cadmus-graph-ui
  cadmus-graph-pg-ex --> cadmus-graph-ui-ex
  cadmus-graph-ui --> ngx-tools
  cadmus-graph-ui --> ngx-mat-tools
  cadmus-graph-ui --> cadmus-api
  cadmus-graph-ui --> paged-data-browsers
  cadmus-graph-ui --> cadmus-refs-lookup
  cadmus-graph-ui-ex --> ngx-tools
  cadmus-graph-ui-ex --> ngx-mat-tools
  cadmus-graph-ui-ex --> cadmus-refs-lookup
  cadmus-graph-ui-ex --> cadmus-core
  cadmus-graph-ui-ex --> cadmus-api
  cadmus-graph-ui-ex --> cadmus-graph-ui
  cadmus-item-editor --> ngx-tools
  cadmus-item-editor --> ngx-mat-tools
  cadmus-item-editor --> paged-data-browsers
  cadmus-item-editor --> cadmus-core
  cadmus-item-editor --> cadmus-api
  cadmus-item-editor --> cadmus-ui
  cadmus-item-editor --> cadmus-state
  cadmus-item-list --> ngx-tools
  cadmus-item-list --> ngx-mat-tools
  cadmus-item-list --> paged-data-browsers
  cadmus-item-list --> cadmus-refs-lookup
  cadmus-item-list --> cadmus-core
  cadmus-item-list --> cadmus-api
  cadmus-item-list --> cadmus-state
  cadmus-item-list --> cadmus-ui
  cadmus-item-search --> ngx-tools
  cadmus-item-search --> ngx-mat-tools
  cadmus-item-search --> paged-data-browsers
  cadmus-item-search --> cadmus-api
  cadmus-item-search --> cadmus-state
  cadmus-item-search --> cadmus-ui
  cadmus-layer-demo --> ngx-tools
  cadmus-layer-demo --> cadmus-core
  cadmus-layer-demo --> cadmus-ui
  cadmus-part-general-pg --> ngx-tools
  cadmus-part-general-pg --> ngx-mat-tools
  cadmus-part-general-pg --> cadmus-core
  cadmus-part-general-pg --> cadmus-state
  cadmus-part-general-pg --> cadmus-ui
  cadmus-part-general-pg --> cadmus-ui-pg
  cadmus-part-general-pg --> cadmus-part-general-ui
  cadmus-part-general-ui --> ngx-tools
  cadmus-part-general-ui --> ngx-mat-tools
  cadmus-part-general-ui --> cadmus-refs-asserted-ids
  cadmus-part-general-ui --> cadmus-refs-doc-references
  cadmus-part-general-ui --> cadmus-refs-historical-date
  cadmus-part-general-ui --> cadmus-refs-asserted-chronotope
  cadmus-part-general-ui --> cadmus-refs-assertion
  cadmus-part-general-ui --> cadmus-refs-proper-name
  cadmus-part-general-ui --> cadmus-core
  cadmus-part-general-ui --> cadmus-ui
  cadmus-part-philology-pg --> ngx-tools
  cadmus-part-philology-pg --> ngx-mat-tools
  cadmus-part-philology-pg --> cadmus-core
  cadmus-part-philology-pg --> cadmus-state
  cadmus-part-philology-pg --> cadmus-ui
  cadmus-part-philology-pg --> cadmus-ui-pg
  cadmus-part-philology-pg --> cadmus-part-philology-ui
  cadmus-part-philology-ui --> ngx-tools
  cadmus-part-philology-ui --> ngx-mat-tools
  cadmus-part-philology-ui --> cadmus-core
  cadmus-part-philology-ui --> cadmus-ui
  cadmus-preview-pg --> ngx-tools
  cadmus-preview-pg --> ngx-mat-tools
  cadmus-preview-pg --> cadmus-api
  cadmus-preview-pg --> cadmus-preview-ui
  cadmus-preview-ui --> ngx-tools
  cadmus-preview-ui --> ngx-mat-tools
  cadmus-preview-ui --> cadmus-text-block-view
  cadmus-preview-ui --> cadmus-api
  cadmus-profile-core
  cadmus-state --> ngx-tools
  cadmus-state --> cadmus-core
  cadmus-state --> cadmus-api
  cadmus-thesaurus-editor --> ngx-tools
  cadmus-thesaurus-editor --> ngx-mat-tools
  cadmus-thesaurus-editor --> cadmus-api
  cadmus-thesaurus-editor --> cadmus-state
  cadmus-thesaurus-editor --> cadmus-ui
  cadmus-thesaurus-editor --> cadmus-thesaurus-ui
  cadmus-thesaurus-list --> ngx-tools
  cadmus-thesaurus-list --> ngx-mat-tools
  cadmus-thesaurus-list --> cadmus-api
  cadmus-thesaurus-list --> cadmus-ui
  cadmus-thesaurus-ui --> ngx-tools
  cadmus-thesaurus-ui --> paged-data-browsers
  cadmus-thesaurus-ui --> cadmus-ui
  cadmus-ui --> ngx-tools
  cadmus-ui --> cadmus-refs-lookup
  cadmus-ui --> cadmus-core
  cadmus-ui-pg --> cadmus-core
  cadmus-ui-pg --> cadmus-api
  cadmus-ui-pg --> cadmus-state
  cadmus-ui-pg --> cadmus-ui
  cadmus-ui-pg --> cadmus-item-editor
```
