---
title: "Deployment - Backup"
layout: default
parent: "Deployment"
nav_order: 4
---

# Data Backup

- [Data Backup](#data-backup)
  - [Backup](#backup)
  - [Restore](#restore)
  - [Cleanup](#cleanup)
  - [Transfer - LFTP](#transfer---lftp)
  - [Transfer - GDrive](#transfer---gdrive)
  - [Crontab](#crontab)
  - [Host Setup](#host-setup)
    - [MongoDB Client](#mongodb-client)
    - [PostgreSQL Client](#postgresql-client)
    - [LFTP Tool](#lftp-tool)

To backup your Cadmus data, you must:

- backup the **Mongo** databases (data), conventionally named as follows:
  - `cadmus-PRJ`
- backup 2 or 3 **PostgreSQL** database(s) (indexes, user accounts, and optionally graph). Conventionally they are named as follows:
  - `cadmus-PRJ` (the index anyway can be rebuilt from data if required)
  - `cadmus-PRJ-auth`
  - `cadmus-PRJ-graph` (present only when using the graph)

If you expose the database services from your Docker containers, as it is usually the case, you just have to use the corresponding database tools to dump databases. Just be sure to use the right port, as often ports are remapped when several services run in the same host machine.

>Note that usually in Docker omitting `ports` for database services altogether is the safest option. Anyway, exposing the database ports to the loopback address (127.0.0.1) on your host machine is better than exposing them to all interfaces (`- 27017:27017`). Since only the API containers need to access the databases, we could remove the `ports` section entirely from the database services. The API services will still be able to connect using the service name and internal port (e.g. `cadmus-PRJ-mongo:27017`) because they share the same Docker network. This is the most secure configuration as the databases are completely isolated from the host machine's external network interfaces. Anyway, as we need to access the DB locally for backup, it is acceptable to use the loopback address.

In Linux, which is the most typical host, you can just have a `.sh` batch file saving your data somewhere, and then launch it periodically with [crontab](https://crontab.guru).

## Backup

👉 In this script replace `PRJ` with your project name.

```sh
#!/bin/bash
# Backup script for Cadmus databases.
# You can launch this by editing cron with crontab -e, using a line like this (daily dump at 3 AM):
# 00 03 * * * /home/crontab-scripts/cadmus-dump.sh

# --- Configuration ---
# Set the base directory for backups
BASE_BACKUP_DIR="./backup"
# Get the current date in YYYYMMDD format for the directory name
DATE_DIR_NAME=`date +%Y%m%d`
# Define the full path for today's backup directory
TODAY_BACKUP_DIR="${BASE_BACKUP_DIR}/${DATE_DIR_NAME}"

# Ensure the base directory exists
mkdir -p "${BASE_BACKUP_DIR}"

# Create the date-stamped folder for today's backup
echo "Creating backup directory: ${TODAY_BACKUP_DIR}"
mkdir -p "${TODAY_BACKUP_DIR}"

# --- MongoDB Dump ---
echo "Dumping MongoDB databases..."

# Dump cadmus-PRJ (main data)
mongodump --port=27017 --db cadmus-PRJ --archive="${TODAY_BACKUP_DIR}/cadmus-PRJ-mongo.gz" --gzip

# Dump cadmus-PRJ-log (logs)
mongodump --port=27017 --db cadmus-PRJ-log --archive="${TODAY_BACKUP_DIR}/cadmus-PRJ-log-mongo.gz" --gzip

# --- PostgreSQL Dump ---
echo "Dumping PostgreSQL databases..."
# Note: Using '-h 127.0.0.1' requires your DB services to be configured with '127.0.0.1:PORT:PORT' in docker-compose.yml

# export the PostgreSQL password so child processes (pg_dump) can see it
export PGPASSWORD='postgres'

# use the -w flag to ensure it fails rather than hangs if something goes wrong
pg_dump -h 127.0.0.1 -U postgres -d cadmus-PRJ -w | gzip > "${TODAY_BACKUP_DIR}/cadmus-PRJ-pgsql.gz"
pg_dump -h 127.0.0.1 -U postgres -d cadmus-PRJ-auth -w | gzip > "${TODAY_BACKUP_DIR}/cadmus-PRJ-auth-pgsql.gz"

# security best practice: unset it after the dump is done
unset PGPASSWORD

echo "Backup completed successfully in ${TODAY_BACKUP_DIR}"
```

⚠️ Note that usually for PostgreSQL you should set the password in the home folder of your user (get it via `echo $HOME`) in a file named `.pgpass` with this content:

```txt
127.0.0.1:5432:*:postgres:YOURPASSWORDHERE
```

Be sure to set its permissions:

```sh
chmod 0600 /root/.pgpass
```

Anyway, this often does not work because when put it in a script the PGPASSWORD variable gets "lost" if not handled correctly. So, the approach to set the password explicitly is more effective.

💡 If you want to download files, just connect to your VM (using your account credentials) via SCP (use your VM IP and port 22). You can either use GUI like WinSCP or command line tools.

## Restore

👉 In this script replace `PRJ` with your project name.

```sh
#!/bin/bash
# Restore script for Cadmus databases.
# Usage: ./restore.sh YYYYMMDD

set -e

# --- Configuration & Validation ---
BASE_BACKUP_DIR="./backup"
TARGET_DATE=$1

if [ -z "$TARGET_DATE" ]; then
    echo "ERROR: Please provide a date folder name (YYYYMMDD) as an argument."
    echo "Usage: $0 YYYYMMDD"
    exit 1
fi

TODAY_BACKUP_DIR="${BASE_BACKUP_DIR}/${TARGET_DATE}"

if [ ! -d "$TODAY_BACKUP_DIR" ]; then
    echo "ERROR: Backup directory ${TODAY_BACKUP_DIR} does not exist."
    exit 1
fi

echo "===================================================="
echo " Starting Cadmus Database Restore from: ${TARGET_DATE}"
echo "===================================================="

# --- MongoDB Restore ---
echo "--> Restoring MongoDB databases via docker exec..."

# Added --drop to clear seeded collections before writing backup data
if [ -f "${TODAY_BACKUP_DIR}/cadmus-PRJ-mongo.gz" ]; then
    echo "Restoring cadmus-PRJ..."
    docker exec -i cadmus-PRJ-mongo mongorestore --drop --archive --gzip < "${TODAY_BACKUP_DIR}/cadmus-PRJ-mongo.gz"
else
    echo "WARNING: ${TODAY_BACKUP_DIR}/cadmus-PRJ-mongo.gz not found. Skipping."
fi

if [ -f "${TODAY_BACKUP_DIR}/cadmus-PRJ-log-mongo.gz" ]; then
    echo "Restoring cadmus-PRJ-log..."
    docker exec -i cadmus-PRJ-mongo mongorestore --drop --archive --gzip < "${TODAY_BACKUP_DIR}/cadmus-PRJ-log-mongo.gz"
else
    echo "WARNING: ${TODAY_BACKUP_DIR}/cadmus-PRJ-log-mongo.gz not found. Skipping."
fi
```

## Cleanup

This script can be periodically run to cleanup the backup folder from oldest dumps.

```sh
#!/bin/bash
# Script to delete old local backups

# --- Configuration ---
# Match this exactly with your backup script location (use absolute paths if run via cron)
BASE_BACKUP_DIR="./backup"
# Number of latest backup folders to KEEP (e.g., 7 days)
KEEP_DAYS=7

echo "Starting cleanup: retaining the last ${KEEP_DAYS} backup folders in ${BASE_BACKUP_DIR}"

# Ensure directory exists before searching
if [ ! -d "${BASE_BACKUP_DIR}" ]; then
    echo "Error: Directory ${BASE_BACKUP_DIR} does not exist."
    exit 1
fi

# Find all daily directories, sort chronologically reverse, skip the first X, and delete
find "${BASE_BACKUP_DIR}" -mindepth 1 -maxdepth 1 -type d | sort -r | tail -n +$((KEEP_DAYS + 1)) | while read dir_to_delete; do
    echo "Deleting old backup folder: ${dir_to_delete}"
    rm -rf "${dir_to_delete}"
done

echo "Cleanup completed."
```

## Transfer - LFTP

Of course, you can then **transfer** your files somewhere else, e.g. (this script is placed in `backup` folder). This script just uses some FTP account.

```sh
#!/bin/bash
# Transfer script to upload the latest backup archive using lftp

# --- Configuration ---
BASE_BACKUP_DIR="./backup"
DATE_DIR_NAME=`date +%Y%m%d`
ARCHIVE_NAME="cadmus-backup-${DATE_DIR_NAME}.tar.gz"
FULL_ARCHIVE_PATH="${BASE_BACKUP_DIR}/${ARCHIVE_NAME}"

FTP_HOST="ftp.myserver.net"
FTP_USER="user"
FTP_PASS="password"

# Verify source folder exists
if [ ! -d "${BASE_BACKUP_DIR}/${DATE_DIR_NAME}" ]; then
    echo "ERROR: Today's backup directory (${BASE_BACKUP_DIR}/${DATE_DIR_NAME}) not found!"
    exit 1
fi

# --- Archiving ---
echo "Creating single archive for upload: ${ARCHIVE_NAME}"
tar -czf "${FULL_ARCHIVE_PATH}" -C "${BASE_BACKUP_DIR}" "${DATE_DIR_NAME}"

# --- Upload ---
echo "Starting upload using lftp..."

# Fixed: Pass the explicit local path to the 'put' command
lftp -e "set cmd:prompt ''; put ${FULL_ARCHIVE_PATH}; bye" -u "${FTP_USER}","${FTP_PASS}" "${FTP_HOST}"

# Check the exit status of lftp
if [ $? -eq 0 ]; then
    echo "Upload completed successfully!"
    echo "Removing local archive file: ${FULL_ARCHIVE_PATH}"
    rm "${FULL_ARCHIVE_PATH}"
else
    echo "ERROR: Upload failed! Keeping archive for inspection at ${FULL_ARCHIVE_PATH}"
    exit 1
fi

echo "Transfer process finished."
```

## Transfer - GDrive

Another popular option is using **GDrive** as your backup target. To this end, you could use a script like this:

```sh
#!/bin/bash
# Backup and upload script using rclone to Google Drive

# --- Configuration ---
BASE_BACKUP_DIR="./backup"
DATE_DIR_NAME=$(date +%Y%m%d)
ARCHIVE_NAME="cadmus-backup-${DATE_DIR_NAME}.tar.gz"
FULL_ARCHIVE_PATH="${BASE_BACKUP_DIR}/${ARCHIVE_NAME}"

RCLONE_REMOTE="gdrive"           # Name of your rclone remote
RCLONE_DESTINATION="backups"     # Folder in Google Drive

# Verify source folder exists
if [ ! -d "${BASE_BACKUP_DIR}/${DATE_DIR_NAME}" ]; then
    echo "ERROR: Today's backup directory (${BASE_BACKUP_DIR}/${DATE_DIR_NAME}) not found!"
    exit 1
fi

# --- Archiving ---
echo "Creating archive: ${ARCHIVE_NAME}"
tar -czf "${FULL_ARCHIVE_PATH}" -C "${BASE_BACKUP_DIR}" "${DATE_DIR_NAME}"

# --- Upload ---
echo "Uploading to Google Drive via rclone..."
rclone copy "${FULL_ARCHIVE_PATH}" "${RCLONE_REMOTE}:${RCLONE_DESTINATION}" --progress

if [ $? -eq 0 ]; then
    echo "Upload successful. Cleaning up local archive..."
    rm "${FULL_ARCHIVE_PATH}"
else
    echo "ERROR: Upload failed. Archive retained for inspection at ${FULL_ARCHIVE_PATH}"
    exit 1
fi

echo "Backup process complete."
```

>💡 Use `rclone move` instead of `copy` if you want to delete local files after upload.

To **setup rclone** in the VM:

```sh
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.deb
sudo apt install ./rclone-current-linux-amd64.deb
```

To test the installation and see the GDrive files listed:

```sh
rclone ls gdrive:
```

If you need to create the target folder:

```sh
rclone mkdir gdrive:backups
```

To **configure rclone** with your GDrive account you need a headless setup via its `--copy-config` method. These are the instructions for the machine with access to your GDrive account:

1. install rclone from <https://rclone.org/downloads>.
2. run `rclone config` and create a remote named `gdrive`: choose Google Drive and follow the OAuth flow in browser, then save the configuration.
3. locate the configuration file you saved (e.g. in Windows it's usually under your user folder `.config\rclone\rclone.conf`) and copy it to the VM hosting Cadmus, e.g. via SCP:

```sh
scp ~/.config/rclone/rclone.conf user@your-vm:/home/user/.config/rclone/rclone.conf
```

## Crontab

You could launch all these scripts sequentially:

```txt
00 03 * * * /home/crontab-scripts/cadmus-dump.sh && \
            /home/crontab-scripts/cadmus-transfer.sh && \
            /home/crontab-scripts/cadmus-cleanup.sh
```

>⚠️ Ensure all scripts are executable: `chmod +x /home/crontab-scripts/*.sh`.

---

## Host Setup

The host would require the database client tools and FTP utility. Should you need to install them, you can follow the procedures outlined here.

### MongoDB Client

This procedure is for Ubuntu 24 ("noble"):

```sh
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor --yes

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

sudo apt-get update

# this installs the shell and the backup/restore tools only
sudo apt-get install -y mongodb-mongosh mongodb-database-tools
```

>💡 To restore a whole MongoDB database, use a command like: `mongorestore --drop --archive="cadmus-PRJ-mongo.gz" --gzip --db cadmus-PRJ`. To restore a single collection, e.g. the facets, in an easy way, export the `facets` collection via a tool like Studio3T: you will get a folder named after the database, including a couple of `.gz` files for the `facets` collection. Upload this folder to the VM (e.g. via SCP) and run a command like this from the PARENT folder of the folder containing the uploaded folder: `mongorestore --drop --nsInclude "cadmus-PRJ.facets" --gzip ./xfer`. So, if the folder was uploaded to `/opt/xfer`, you must run this command from `/opt`.

Old procedure (for versions before 24, change version numbers as required):

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-8.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt update
sudo apt install -y mongodb-database-tools

# for mongosh
curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-8.0.gpg
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -sc)/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt update
sudo apt install -y mongodb-mongosh
```

### PostgreSQL Client

(1) add the APT repository:

```sh
echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
```

(2) import its GPG key:

```sh
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
```

(3) Update the package list (if not recently done):

```sh
sudo apt update
```

(4) Install the client package (change client version accordingly):

```sh
sudo apt install -y postgresql-client-18
```

To open the client shell:

```sh
psql -h 127.0.0.1 -p 5432 -U postgres
```

### LFTP Tool

Install the `lftp` tool:

```sh
sudo apt install -y lftp
```
