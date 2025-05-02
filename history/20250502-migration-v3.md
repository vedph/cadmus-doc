---
title: "2025-05-02 Migration V3"
layout: default
parent: History
nav_order: 1
---

# 2025-05-02: Migration V3

Migration V3 is a new version of Cadmus data migration components where export has been refactored to a higher abstraction level. This allows for a more generalized approach, covers both Cadmus and GVE for their export processes, and makes the export more powerful by introducing text trees rather than just a list of text blocks in the rendering pipeline.

New major versions are:

- backend (migration): 7.
- backend (API): 12.
- frontend (API and preview): 13.

Once upgraded backend to [migration V3](https://github.com/vedph/cadmus-migration-v3) (backend version 7), you need to apply these changes:

1. in your backend, update libraries and adjust the preview configuration [preview-profile.json](https://github.com/vedph/cadmus-api/blob/master/CadmusApi/wwwroot/preview-profile.json) according to the [breaking changes](https://github.com/vedph/cadmus-migration-v3#700):
   - rename section `RendererFilters` into `TextFilters`.
   - use `:` instead of `|` for type ID + role ID separator, e.g. `it.vedph.token-text-layer:fr.it.vedph.apparatus` rather than `it.vedph.token-text-layer|fr.it.vedph.apparatus`.
   - update component tags where required.

2. in your frontend, update libraries.
