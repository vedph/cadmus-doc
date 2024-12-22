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
  cadmus-graph-ui-ex --> cadmus-core
  cadmus-graph-ui-ex --> cadmus-api
  cadmus-graph-ui-ex --> cadmus-graph-ui
  cadmus-graph-ui-ex --> cadmus-refs-lookup
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
  cadmus-item-list --> cadmus-core
  cadmus-item-list --> cadmus-api
  cadmus-item-list --> cadmus-state
  cadmus-item-list --> cadmus-ui
  cadmus-item-list --> cadmus-refs-lookup
  cadmus-item-search --> ngx-tools
  cadmus-item-search --> ngx-mat-tools
  cadmus-item-search --> paged-data-browsers
  cadmus-item-search --> cadmus-api
  cadmus-item-search --> cadmus-state
  cadmus-item-search --> cadmus-ui
```
