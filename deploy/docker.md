---
title: "Deployment - Docker Setup"
layout: default
parent: "Deployment"
nav_order: 1
---

# Docker Setup

This is a quick recap about installing Docker. For deployment, you will usually have a [Linux VM](#ubuntu). For development (or running locally, mostly used for testing or evaluating the app) you just have to pick the OS of your choice and follow the instructions here.

## Windows

For Windows, it is recommended to apply all the latest updates, so that you can take advantage of WSL 2 (Windows Subsystem for Linux). Once your OS is up to date, just follow the directions during Docker setup (you will need to install an additional component when requested).

- download and install Docker Desktop from <https://hub.docker.com/editions/community/docker-ce-desktop-windows> (see <https://docs.docker.com/docker-for-windows/install/>).

## MacOS

- download and install Docker for Desktop from <https://docs.docker.com/docker-for-mac/install/>. As for Windows, it already includes docker compose.

## Ubuntu

You must install both *Docker* and *Docker compose*, as these are separate packages.

### Ubuntu - Docker

To install Docker run these commands (see <https://docs.docker.com/install/linux/docker-ce/ubuntu/>):

```bash
sudo apt-get update

sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
 "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

You can verify that Docker is installed correctly by running the hello-world image:

```bash
sudo docker run hello-world
```

To automatically start Docker:

```bash
sudo systemctl start docker
```

and then:

```bash
sudo systemctl enable docker
```

Check for installation: `docker --version`.

### Ubuntu - Docker-Compose

Docker compose (V2) now comes as a plugin, which is automatically installed by the desktop versions of Docker for Windows/MacOS. As for Linux, you install it with the commands shown here (from <https://www.rockyourcode.com/how-to-install-docker-compose-v2-on-linux-2021/>):

(1) find the latest release with the v2 tag at <https://github.com/docker/compose/tree/v2> (e.g. 2.28.1).

(2) ensure that the Docker CLI plugins directory exists:

```bash
mkdir -p ~/.docker/cli-plugins
```

(3) download the compose CLI plugin (here replace version `2.32.1` with the latest one):

```bash
curl -sSL https://github.com/docker/compose/releases/download/v2.32.1/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
```

(4) make it executable:

```bash
chmod +x ~/.docker/cli-plugins/docker-compose
```

(5) check with: `docker compose version`.

>Remember that docker V2 being a plugin no longer uses dashes (like `docker-compose`) but rather spaces (like `docker compose`).
