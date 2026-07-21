---
title: "Models" 
layout: default
nav_order: 2
---

# Models

At the heart of Cadmus there are its data models, which are built by composition; and this is true also of its editor.

Cadmus is **object**-based: items (the main records) are composite objects, containing parts and fragments; these are objects, having any number and type of properties. Each property is either a scalar value (e.g. a string, a number, etc.) or yet another object, which in turn has its properties. So you can think of an object as a tree of properties.

You are free to create any model with its editing UI, and add them into an existing or new editor; but often you just reuse models and editors which have been designed to be shared across projects.

In fact, it may well happen that a real-world project is completely built by assembling these shared resources, without having to add anything new; while other projects can add their own specific models.

As Cadmus is geared towards reuse, it is often the case that project-specific models prove to be general enough to be promoted to project-independent, [shared models](shared), primarily designed for cross-project reuse. This approach fosters reuse and even indirect collaboration among scholars, while also providing a foundation for modeling specific knowledge domains by generalizing from specific real-world projects.

## Items

Items are the top-level objects in a Cadmus database. You can think of them as boxes, designed to contain any number and type of objects, named parts.

Items have a fixed set of **metadata** (the label attached to the box):

- id: a GUID uniquely and globally identifying the item.
- title: a human-friendly title for the item.
- description: a short description of the item.
- sort key: a string used to sort items in the editor. This is calculated according to item's metadata.
- facet: the item's facet ID. A facet tells which parts can be put inside an item, with all their metadata. So you can think of the facet as the "type" of an item; even if this is open and dynamic, because at any time we can add new part definitions to the facet to extend it.
- group ID: an arbitrary string used to virtually group all the items sharing that string value. For instance, you might have a set of epigrams belonging to a notebook called H5, and assign H5 as the group ID to each item representing an epigram.
- flags: editorial flags attached to the item. These are defined per-project up to a maximum of 32, and represent editorial states like draft, completed, revised, published, etc.

## Parts

Parts are objects of any type put into an item. They are identified by a GUID and have a type ID (which identifies the object type) and optionally a role ID.

The role ID is added whenever the same part is used multiple times with different roles. For instance, you might use a generic categories part twice, to assign tags from a vocabulary A and from a second vocabulary B. In this case, the item contains two parts of the same type (categories), which refer to two distinct vocabularies, and these parts are distinguished by their role ID.

## Fragments

Fragments are used for text annotation. Text can be encoded in Cadmus just like any other object: as an item with parts:

- a part represents the "base text" (the text being annotated).
- any number of parts represent layers of annotations on top of the base text (e.g. critical apparatus, orthography, paleography, chronology, links, etc.). These parts have all the same type, but a role which changes according to the layer type. In this case, the role is the identifier of the annotation type, which by convention always starts with `fr.` = fragment.

A fragment is an annotation belonging to a specific layer. For instance, a critical apparatus is a text layer part using critical apparatus entries as annotation types; each entry is a fragment.

>The name "fragments" represents the fact that these annotations are still part of an item, but differently from parts, which are independent and self-contained objects, fragments exist only within a single text layer part, as the annotations in it.

## Defining Models

You define each 'top-level' entity in your project as an item, which can be fully described by including all the parts you want for it: e.g. one for its date, another for its place, another for its tags, another for a free text description, etc.

Each item type is defined in its own facet. For instance, you might have a facet representing a person, another representing a manuscript, another representing an inscription, etc. There is no limit to facets and to their contents. This provides a dynamic and composable model.

When starting a project, you typically follow this procedure:

- decide which are your **entities**. Usually you start from top-level entities, represented by items.
- for each entity, decide the **parts** used to describe all what you want to say about it. You can reuse general parts or parts from other projects, and also create new parts of your own.
- for each part used, define its closed **vocabularies**. Most parts have such vocabularies to allow for a consistent and user-friendly model. Most of these vocabularies are implemented as [thesauri](../linking/thesauri.md), but they can also be [taxonomies](../linking/taxonomies.md).
- define the **relationships** among entities and use the proper models to implement them as [links](../linking/index.md). If you are linking an item to external or internal resources, you can just add a links part to it. If you are linking a specific object inside a part's (or fragment's) model, you can use the links "brick" in its definition.

>A brick is a self-contained micro-model used as the building block of bigger objects like parts or fragments. For instance, a structured date is a brick, and you can insert it into the model of any part or fragment requiring it. Each brick has its model and its UI widget. You can play with stock Cadmus bricks at <https://cadmus-bricks-v3.fusi-soft.com/>.
