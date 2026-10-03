---
title: "Developing Frontend"
parent: "Developing"
layout: default
nav_order: 2
---

# Developing Frontend

A Cadmus frontend app can be as simple as a simple Angular app using some Cadmus libraries, or add its own parts and/or fragments, or any other component useful for its purposes.

The typical procedure to setup a Cadmus frontend app consists of:

1. [create the app](app-setup).
2. add to the app all the required NPM packages and code resources.
3. optionally, add specific [parts](app-parts) and/or [fragments](app-fragments) packed into [libraries](app-lib) in the same workspace. Their editors often use child editors for single objects (see [object editors](app-object-editor)), and the [Monaco editor](monaco) for long texts.

Part and fragment editors use Angular signal forms. If you are maintaining editors written with the former reactive forms, see the migration guide in the [shell's changelog](https://github.com/vedph/cadmus-shell-v3/blob/master/CHANGELOG.md#migrating-a-part-editor).
