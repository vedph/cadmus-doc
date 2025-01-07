---
title: "Deployment - Backup"
layout: default
parent: "Deployment"
nav_order: 4
---

# Data Backup

To backup your Cadmus data, you must:

- backup its 3 **Mongo** databases (data, authentication, and log). You can leave out logs if you are not interested in keeping audit records.
- backup its **PostgreSQL** database(s) (indexes and optionally graph).

If you expose the database services from your Docker containers, as it is usually the case, you just have to use the corresponding database tools to dump databases. Just be sure to use the right port, as often ports are remapped when several services run in the same host machine.

## Linux

In Linux, which is the most typical host, you can just have a `.sh` batch file savinig your data somewhere, and then launch it periodically with [crontab](https://crontab.guru). Here is an example, which saves all the databases into separate, compressed files, with their date.

### Backup

```sh
#!/bin/bash
# you can launch this by editing cron with crontab -e, using a line like this (daily dump at 3 AM):
# 00 03 * * * /home/crontab-scripts/cadmus-dump.sh

vardate=`date +%Y%m%d`

echo mongo data...
mongodump --port=27017 --db cadmus-PRJ --archive=./backup/cadmus-PRJ-${vardate}.gz --gzip

echo mongo auth...
mongodump --port=27017 --db cadmus-PRJ-auth --archive=./backup/cadmus-PRJ-auth-${vardate}.gz --gzip

echo mongo log...
mongodump --port=27017 --db cadmus-PRJ-log --archive=./backup/cadmus-PRJ-log-${vardate}.gz --gzip

echo pgsql data...
pg_dump --username=postgres -h 127.0.0.1 cadmus-PRJ | gzip > ./backup/cadmus-PRJ-pgsql-${vardate}.gz

# if using MySql
# echo mysql...
#mysqldump --host 127.0.0.1 --port 3306 -uroot -pmysql cadmus-PRJ | gzip > ./backup/cadmus-PRJ-mysql-${vardate}.gz

echo completed!
```

### Transfer

Of course, you can then **transfer** your files somewhere else, e.g. (this script is placed in `backup` folder):

```bash
#!/bin/bash

vardate=`date +%Y%m%d`
echo upload
ftp-upload -h ftp.myserver.net -u user --password password *.gz
echo completed!
```
