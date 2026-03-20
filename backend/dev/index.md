---
title: "Developing Backend"
parent: "Cadmus Backend"
layout: default
nav_order: 5
---

# Developing Backend

When creating the backend for a Cadmus project, typically you will:

1. create a core solution to host your project-specific models (parts and fragments with their seeders and services), if required.
2. create an API solution for your project web API, which will be consumed by the frontend editor. This API solution will just be a wrapper for imported logic except for the configuration, which will be specific to your project.

This section just contains suggestions; you are free to pick your structure. In general, having a separate solution for core backend models is suggested when you are going to share it. If you are creating models specific to your own project and you do not plan to share them, you can also adopt a single solution with 2 projects in it, one for the models and another for the API.
