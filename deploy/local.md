---
title: "Local Deployment"
layout: default
parent: "Deployment"
nav_order: 5
---

# Local Deployment

You might want to deploy Cadmus on your local machine, either because you plan to use it in this way, or because you just want to experiment with it.

To this end, every Cadmus project has its `docker-compose.yml` script which creates the stack of services required to run Cadmus locally with no setup (except having Docker installed). In most cases, this project also creates some mock data to play with (usually 100 items) so you can play with an editor with data already available.

## Starting Editor

To start the editor, follow this procedure:

▶️ (1) prerequisite: [install Docker](docker.md) if you do not already have it.

▶️ (2) locate the Cadmus repository project you want to play with. Usually such projects are named after the project abbreviation, e.g. `cadmus-ndp-app` is the name of the frontend editor for a project whose abbreviation is NDP. Once located, download the `docker-compose.yml` file you find in it, saving it in any directory in your computer. It is good practice to reserve a specific directory for it, as this makes things easier when you use multiple scripts.

> 💡 Pick a script's folder name which is short and contains just letters, preferably all lowercase, without diacritics or spaces. This will make it easier to refer to these folder names when using the terminal.

▶️ (3) open a terminal in the folder where you placed the downloaded `docker-compose.yml` and run this command (omit `sudo` if using Windows):

```sh
sudo docker compose up
```

This should fire up a number of services, each outputting diagnostic messages to the terminal window during its startup, and having a prefix with a different color according to the service name. Once messages stop running, and if no errors are displayed, you should see a final message telling something like "listening on port..." and a number.

> ⚠️ Note that in most cases the startup phase will also seed some mock data into the database. To this end, it will pause for a few seconds (defined in the script) before seeding to ensure that the database services have completed their startup. So, if you see a seeding message and messages pause for a while, just wait for the seed process to start. You will see one message for each item being seeded.
>
> ⚠️ Also note that the default script assumes you do not have MongoDB or PostgreSQL database services running on your system. If this is not the case, change the script to either change the database ports (so that they can run side by side to your existing services), or use them rather than creating database services containers.

▶️ (4) open your browser and goto <localhost:4200>. The Cadmus editor homepage should open. Click the login menu item and enter these fake default credentials:

- username: `zeus`
- password: `P4ss-W0rd!`

You should now see other menu items appear, and you will be able to browse the items which were seeded into the database. You can now play with the editor as you wish.

## Stopping Editor

To stop the editor, do not just close the terminal window: first, click on it and press CTRL+C to stop services. After a few seconds you should see messages which tell you that services are being stopped. Once they complete, you can safely close the terminal window.

## Restarting Editor

If you use Docker Desktop, once you have run the script the first time you will find a corresponding stack of services in it. The easiest way to restart the stack to use the editor again is just clicking the "play" button to start it from that UI.

If instead you prefer to use the terminal, just do as before: open it under the script folder, and run `docker-compose up` again.

## Resetting Editor

When you want to reset the editor, wipe out all data and restart fresh:

- on Docker Desktop, delete the whole stack.
- alternatively, in the terminal, under the script folder, run `docker compose down`.

You can then restart the editor as explained above to start fresh. This is useful when you are just playing with the editor and you freely mess with data without breaking anything.

## Updating Editor

If an update is published, you can update your editor by just changing the version numbers in the script, or simply downloading the script again _if you did not modify it_ (otherwise you would overwrite your changes).

Once you have updated the script, you can either reset the editor as explained above, or just recreate containers (which is safer if you are not going to throw data away) with the following commands (run from the script's folder; remove `sudo` if using Windows):

```sh
sudo docker compose up -d cadmus-YOURPROJECT-api
sudo docker compose up -d cadmus-YOURPROJECT-app
```

These command just pull the updated images and restart the containers for Cadmus API backend and frontend application, which usually have the names listed above. Should you change them, update the commands accordingly. Note that database containers are not touched in this case, so your data will be intact.

## Preserving Data

If you want to preserve data even when you destroy and recreate Docker containers, which is the recommended way to use the editor once you enter real data, you must add _volumes_ to the docker compose script (see [app deployment](app.md)). These ensure that even if you destroy the whole stack (`docker compose down`) your data will survive.

Of course, even when running locally, you should always implement a periodic [backup](backup.md) policy.
