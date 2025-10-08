---
title: "Deployment - Backup"
layout: default
parent: "Deployment"
nav_order: 4
---

# Data Backup

- [Data Backup](#data-backup)
  - [Linux](#linux)
    - [Backup](#backup)
    - [Cleanup](#cleanup)
    - [Transfer](#transfer)
    - [Crontab](#crontab)
  - [Host Setup](#host-setup)
    - [MongoDB Client](#mongodb-client)
    - [PostgreSQL Client](#postgresql-client)
    - [LFTP Tool](#lftp-tool)

To backup your Cadmus data, you must:

- backup 1 or 2 **Mongo** databases (data and logs). You can leave out logs if you are not interested in keeping audit records. Conventionally they are named as follows:
  - `cadmus-PRJ`
  - `cadmus-PRJ-log`
- backup 2 or 3 **PostgreSQL** database(s) (indexes, user accounts, and optionally graph). Conventionally they are named as follows:
  - `cadmus-PRJ` (the index anyway can be rebuilt from data if required)
  - `cadmus-PRJ-auth`
  - `cadmus-PRJ-graph` (present only when using the graph)

If you expose the database services from your Docker containers, as it is usually the case, you just have to use the corresponding database tools to dump databases. Just be sure to use the right port, as often ports are remapped when several services run in the same host machine.

>Note that usually in Docker omitting `ports` for database services altogether is the safest option. Anyway, exposing the database ports to the loopback address (127.0.0.1) on your host machine is better than exposing them to all interfaces (`- 27017:27017`). Since only the API containers need to access the databases, we could remove the `ports` section entirely from the database services. The API services will still be able to connect using the service name and internal port (e.g. `cadmus-ndp-mongo:27017`) because they share the same Docker network. This is the most secure configuration as the databases are completely isolated from the host machine's external network interfaces. Anyway, as we need to access the DB locally for backup, it is acceptable to use the loopback address.

## Linux

In Linux, which is the most typical host, you can just have a `.sh` batch file saving your data somewhere, and then launch it periodically with [crontab](https://crontab.guru). Here is an example, which saves all the databases into separate, compressed files, with their date.

### Backup

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

# Dump cadmus-PRJ (main data)
pg_dump --username=postgres -h 127.0.0.1 cadmus-PRJ | gzip > "${TODAY_BACKUP_DIR}/cadmus-PRJ-pgsql.gz"

# Dump cadmus-PRJ-auth (accounts)
pg_dump --username=postgres -h 127.0.0.1 cadmus-PRJ-auth | gzip > "${TODAY_BACKUP_DIR}/cadmus-PRJ-auth-pgsql.gz"

echo "Backup completed successfully in ${TODAY_BACKUP_DIR}!"
```

⚠️ Note that for PostgreSQL you should set the password in the home folder of your user (get it via `echo $HOME`) in a file named `.pgpass` with this content:

```txt
127.0.0.1:5432:*:postgres:YOURPASSWORDHERE
```

Be sure to set its permissions:

```sh
chmod 0600 /root/.pgpass
```

💡 If you want to download files, just connect to your VM (using your account credentials) via SCP (use your VM IP and port 22). You can either use GUI like WinSCP or command line tools.

### Cleanup

This script can be periodically run to cleanup the backup folder from oldest dumps.

```sh
#!/bin/bash
# Script to delete old local backups

# --- Configuration ---
# Set the base directory where your backup folders are located
BASE_BACKUP_DIR="./backup"
# Number of latest backup folders to KEEP (e.g., 7 days)
KEEP_DAYS=7

echo "Starting cleanup: retaining the last ${KEEP_DAYS} backup folders in ${BASE_BACKUP_DIR}"

# Find all directories in BASE_BACKUP_DIR, sort them by name (which is the date),
# skip the latest 'KEEP_DAYS' directories, and delete the rest.
# The date format (YYYYMMDD) ensures alphabetical sorting is chronological.
find "${BASE_BACKUP_DIR}" -mindepth 1 -maxdepth 1 -type d | sort -r | tail -n +$((KEEP_DAYS + 1)) | while read dir_to_delete; do
    echo "Deleting old backup folder: ${dir_to_delete}"
    rm -rf "${dir_to_delete}"
done

echo "Cleanup completed."
```

### Transfer

Of course, you can then **transfer** your files somewhere else, e.g. (this script is placed in `backup` folder):

```sh
#!/bin/bash
# Transfer script to upload the latest backup archive using lftp

# --- Configuration ---
BASE_BACKUP_DIR="./backup"
DATE_DIR_NAME=`date +%Y%m%d`
ARCHIVE_NAME="cadmus-backup-${DATE_DIR_NAME}.tar.gz"

# NOTE: For security, use SFTP if your server supports it:
# Change 'ftp://' to 'sftp://'
FTP_HOST="ftp.myserver.net"
FTP_USER="user"
FTP_PASS="password" # Replace with a real password or read from a secure source

# --- Archiving ---
echo "Creating single archive for upload: ${ARCHIVE_NAME}"

# Create a tarball of the daily folder
tar -czf "${BASE_BACKUP_DIR}/${ARCHIVE_NAME}" -C "${BASE_BACKUP_DIR}" "${DATE_DIR_NAME}"

# --- Upload ---
echo "Starting upload using lftp..."

# Use lftp to connect, set prompt off, put the file, and quit
lftp -e "set cmd:prompt ''; put ${ARCHIVE_NAME}; bye" -u "${FTP_USER}","${FTP_PASS}" "${FTP_HOST}"

# Check the exit status of lftp
if [ $? -eq 0 ]; then
    echo "Upload completed successfully!"
    # Clean up the single archive file after upload (keep the original folder)
    echo "Removing local archive file: ${ARCHIVE_NAME}"
    rm "${BASE_BACKUP_DIR}/${ARCHIVE_NAME}"
else
    echo "ERROR: Upload failed! Keeping archive for inspection."
fi

echo "Transfer process finished."
```

This script just uses some FTP account. Another popular option is using **GDrive** as your backup target. To this end, you could use a script like this:

```sh
#!/bin/bash
# Backup and upload script using rclone to Google Drive

# --- Configuration ---
BASE_BACKUP_DIR="./backup"
DATE_DIR_NAME=$(date +%Y%m%d)
ARCHIVE_NAME="cadmus-backup-${DATE_DIR_NAME}.tar.gz"
RCLONE_REMOTE="gdrive"           # Name of your rclone remote
RCLONE_DESTINATION="backups"     # Folder in Google Drive

# --- Archiving ---
echo "Creating archive: ${ARCHIVE_NAME}"
tar -czf "${BASE_BACKUP_DIR}/${ARCHIVE_NAME}" -C "${BASE_BACKUP_DIR}" "${DATE_DIR_NAME}"

# --- Upload ---
echo "Uploading to Google Drive via rclone..."
rclone copy "${BASE_BACKUP_DIR}/${ARCHIVE_NAME}" "${RCLONE_REMOTE}:${RCLONE_DESTINATION}" --progress

if [ $? -eq 0 ]; then
    echo "Upload successful. Cleaning up local archive..."
    rm "${BASE_BACKUP_DIR}/${ARCHIVE_NAME}"
else
    echo "ERROR: Upload failed. Archive retained for inspection."
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

### Crontab

You could launch all these scripts sequentially:

```txt
00 03 * * * /home/crontab-scripts/cadmus-dump.sh && \
            /home/crontab-scripts/cadmus-transfer.sh && \
            /home/crontab-scripts/cadmus-cleanup.sh
```

>⚠️ Ensure all scripts are executable: `chmod +x /home/crontab-scripts/*.sh`.

## Host Setup

The host would require the database client tools and FTP utility. Should you need to install them, you can follow the procedures outlined here.

### MongoDB Client

(1) Import the MongoDB Public Key (⚠️ change the key and version as required for all these commands!):

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-8.0.asc | sudo apt-key add -
```

(2) Add MongoDB repository to the list:

```sh
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

(3) Update the package list:

```sh
sudo apt update
```

(4) Install the MongoDB Database Tools (which includes mongodump):

```sh
sudo apt install -y mongodb-database-tools
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
sudo apt install -y postgresql-client-17
```

### LFTP Tool

Install the `lftp` tool:

```sh
sudo apt install -y lftp
```
