---
title: "Developing Backend"
parent: "Cadmus Backend"
layout: default
nav_order: 1
---

# Developing Backend

When creating the backend for a Cadmus project, typically you will create a blank backend Visual Studio solution to host:

- optionally, your specific **models** (parts and fragments with their seeders).
- optionally, a **service** wrapping these models (typically when they represent a self-contained set of models, like e.g. codicological models). This is not required, but it provides a quicker way to include them in your project.
- the **API** for your project. The API project is only a wrapper for imported logic except for the configuration, which will be specific to your project.
- optionally a **CLI** tool if you need to import data or do other project-specific tasks.
