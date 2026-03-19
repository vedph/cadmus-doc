---
title: "2026-03-19 Cadmus UI PG"
layout: default
parent: History
nav_order: 4
---

# Remove Cadmus UI PG Library

`CurrentItemBarComponent` and `CurrentLayerPartBarComponent` were moved into `@myrmidon/cadmus-item-editor` from library `@myrmidon/cadmus-ui-pg` which had no other code and has thus been removed. This is a breaking change which avoids potential circular references and removes the need for this tiny library.

## Upgrade

To upgrade your app:

1. update `@myrmidon/cadmus-item-editor` (16.1.3 or higher).
2. remove package `@myrmidon/cadmus-ui-pg` altogether (`pnpm uninstall @myrmidon/cadmus-ui-pg`), then change the imports in your components accordingly (just compile and wherever you get an import error replace `@myrmidon/cadmus-ui-pg` with `@myrmidon/cadmus-item-editor`).
