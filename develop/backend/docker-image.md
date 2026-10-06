---
title: "Creating Docker Image"
parent: "Developing Backend"
layout: default
nav_order: 10
---

# Creating a Docker Image

In these templates `__PRJ__` is the placeholder for your project tag.

## Setup

At the root of your solution:

- 📁 add this `.dockerignore` file if missing:

```txt
**/.classpath
**/.dockerignore
**/.env
**/.git
**/.gitignore
**/.project
**/.settings
**/.toolstarget
**/.vs
**/.vscode
**/*.*proj.user
**/*.dbmdl
**/*.jfm
**/azds.yaml
**/bin
**/charts
**/docker-compose*
**/Dockerfile*
**/node_modules
**/npm-debug.log
**/nuget.config
**/obj
**/secrets.dev.yaml
**/values.dev.yaml
LICENSE
README.md
```

- 📁 add the `Dockerfile`:

```dockerfile
# Stage 1: base (uses target platform architecture for ASP.NET runtime)
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 443

# Stage 2: build/publish (SDK runs natively on host platform for speed)
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:10.0 AS build
ARG TARGETARCH
ARG TARGETOS

WORKDIR /src

# Copy project files and source
COPY . .

# Use bash to map amd64 -> x64 and build for the target RID cleanly
RUN /bin/bash -c '\
    RID_ARCH="${TARGETARCH/amd64/x64}" && \
    dotnet publish "Cadmus.__PRJ__.Api/Cadmus.__PRJ__.Api.csproj" \
    -c Release \
    -r "${TARGETOS:-linux}-${RID_ARCH}" \
    --no-self-contained \
    -o /app/publish'

# Stage 3: final image
FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "Cadmus.__PRJ__.Api.dll"]
```

## Create Image

- ⚠️ before creating Docker images, ensure you have a buildx builder instance running that supports multi-arch:

```sh
docker buildx create --use --name multi-arch-builder || docker buildx use multi-arch-builder
docker buildx inspect --bootstrap
```

> To run natively on Linux VMs, macOS (both Intel and Apple Silicon), and Windows (via WSL2 or Docker Desktop)—`linux/amd64` and `linux/arm64` are the only two targets we need. Note that `docker buildx` automatically injects variables like `TARGETARCH` and `TARGETOS` into the scope of your build. In `Dockerfile` we pass these directly to the .NET CLI commands.

- 🐋 run this command to build for multiple platforms and push directly to Docker Hub (replace the tags as required for your account):

```sh
docker buildx build --platform linux/amd64,linux/arm64 -t vedph2020/cadmus-__PRJ__-api:0.0.1 -t vedph2020/cadmus-__PRJ__-api:latest --push .
```

> When there is no available ARM image, add `platform: linux/amd64` as a sibling of the `image` line in docker compose script to force Docker to pull and run the x64 image under emulation, e.g.:

- add `platform: linux/arm64` to MongoDB and PostgreSQL. Note that some images (especially for PostgreSQL) are not available for ARM, so you may need to add `platform: linux/amd64` to them too or downgrade to a version which supports ARM (e.g. `image: postgres:16`).
- add `platform: linux/amd64` to API and app.
