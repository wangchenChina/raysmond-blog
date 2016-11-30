---
title: How to backup MySQL database daily automatically in Ubuntu/Linux
date: 2015-09-28 00:00:00
tags:
  - 數據庫
  - MySQL
  - Linux
---

Here we use a bash script to backup MySQL database. First of all, create a backup bash script /root/backup_db.sh

```
#!/bin/bash

DB="your_db_name"
NOW=$(date +"%Y%m%d%H%M%S")
FILE="$DB-$NOW.sql"

echo "Backing up database($DB) to /root/backup/$FILE.tar.gz, please wait..."
cd /root/backup
mysqldump --defaults-file=/root/.my.cnf $DB > $FILE

tar -zcvf $FILE.tar.gz $FILE
rm $FILE

echo "Done."
```

To avoid password prompt for MySQL user root, You need to create a setting file .my.cnf in the home directory /root.

```
[mysqldump]
user=root
password=root_password
```

If you run the script, you’ll backup your db to the targeted directory.

```
/root/backup_db.sh
ls /root/backup
your_db_name-20150912001201.sql.tar.gz
```

Then how to run the bash script daily automatically? You can add it Linux cron task in order to run the backup script every day at 5:00 am.

```
crontab -e

# Add the following line to the end of the file
0 5 * * * /root/backup_db.sh
```
