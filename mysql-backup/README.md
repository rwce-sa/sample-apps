### mysql-backup

This is a sample bash application that runs two scripts, as accepted by Apcera Platform's built-in bash stager:

1. `bash_start.sh` creates a crontab file `/etc/cron.d/backup`, starts a monitoring loop, and starts cron.
2. `mysql_backup.sh` is executed once an hour by cron. It backs up the MySQL server that's bound to the app and saves the backup onto the NFS service that's bound to the app.

To create the app, perform the following:

```
cd mysql-backup
apc app create mysql-backup-app --allow-egress
```

Bind the database that you want to back up to the app:

```
apc service bind mysql-service --job mysql-backup-app
```

Create and bind the NFS service to the app, mounting the NFS file system at `/backups/mysql-service`:

```
apc service create mysql-backup-storage --provider /apcera/providers::apcfs --description "Storage for mysql backups"
apc service bind mysql-backup-storage --job mysql-backup-app --batch -- --mountpath /backups/mysql-service
```

Next, start the app as follows:

```
apc app start mysql-backup-app
```

Navigate to the URL provided from the app staging process to view the output page. You should see:

* A 503 response code and the message "First backup has not completed" if no backups have yet completed.
* A 200 response code and the message "Last backup completed [number of seconds] seconds ago" once hourly backups are running.
* A 500 response code and the message "ERROR: No new backups in over [number of seconds] seconds" if a backup hasn't happened in over 12 hours.

A monitoring application can check the URL and send an alert if the response code is an error.