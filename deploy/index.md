---
title: "Cadmus Deployment"
layout: default
nav_order: 5
---

# Cadmus Deployment

This section contains general information about deploying a Cadmus app. Cadmus is distributed in Docker images, so in the end all what you need is a host with Docker installed, and a Docker compose script to orchestrate all the required services.

Typically, you start with a Linux VM and install Docker in it. Then, you create a Docker compose script by just [customizing](app) a provided template. This allows the server to run a full Cadmus stack including its databases, so that everything is contained in a single VM; or use external database servers for storage if you prefer.

Once your services are up and running, typically you will have to somehow connect them to the outer world. A typical in-house solution when using a single VM is just adding another Docker stack to provide a reverse proxy, which will connect external endpoints (one for the API, and another for the frontend app) to the local VM services hosted in Docker containers.
