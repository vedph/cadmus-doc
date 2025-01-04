---
title: "Models" 
layout: default
nav_order: 2
---

# Models

At the heart of Cadmus there are its data models, which are built by composition; and this is true also of its editor.

You are free to create any model with its editing UI, and add them into an existing or new editor; but often you can just reuse models and editors which have been designed to be shared across projects. In fact, it may well happen that a real-world project is completely built by assembling these shared resources, without having to add anything new; while other projects can add their own specific models.

Anyway, as Cadmus is geared towards reuse, it is often the case that project-specific models prove to be general enough to be promoted to project-independent, [shared models](shared), primarily designed for cross-project reuse. This approach fosters reuse and even indirect collaboration among scholars, while also providing a foundation for modeling specific knowledge domains by generalizing from specific real-world projects.

Additionally, models are complemented by the usage of [taxonomies](thesauri), named _thesauri_. These can be simple, "flat" lists, or complex hierarchies with classes and sub-classes. In both cases, they always provide a machine-targeted identifier string and a human-readable label for it.

>By convention, usually identifiers are English, while labels are freely defined in the language of choice.

Any UI component can leverage these taxonomies to allow users pick values from a preset (yet editable) list, rather than freely typing them. Next to this "static" data lookup Cadmus also provides a [dynamic lookup](lookup) to link items and parts entered in the editor.
