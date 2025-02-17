# Odoo System Setup Guide

## Step 1: Making Odoo Read Files from a Custom Directory
### Create the directory and set permissions:
```bash
sudo mkdir -p /mnt/extra-addons/enterprise
sudo chown -R odoo18:odoo18 /mnt/extra-addons
sudo chmod -R 755 /mnt/extra-addons
```

### Update the Odoo configuration file (/etc/odoo18.conf):
Add the following line under `[options]`:
```ini
addons_path = /opt/odoo18/odoo18/addons,/mnt/extra-addons/enterprise
```

### Restart Odoo service:
```bash
sudo systemctl daemon-reload
sudo systemctl restart odoo18
```

---

## Step 2: Set Up Automatic Database Backup
### Create the backup directory:
```bash
sudo mkdir -p /opt/odoo18/backups
sudo chown odoo18:odoo18 /opt/odoo18/backups
sudo chmod 750 /opt/odoo18/backups
```

### Create a backup script (/usr/local/bin/odoo_backup.sh):
#### For SQL compressed backup (.sql.gz):
```bash
#!/bin/bash
DB_NAME="odoo18"
BACKUP_DIR="/opt/odoo18/backups"
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_FILE="$BACKUP_DIR/$DB_NAME-$DATE.sql.gz"

sudo -u postgres pg_dump $DB_NAME | gzip > $BACKUP_FILE
chown odoo18:odoo18 $BACKUP_FILE
chmod 640 $BACKUP_FILE

echo "Backup for $DB_NAME completed at $DATE" >> /var/log/odoo_backup.log
find $BACKUP_DIR -type f -name "*.sql.gz" -mtime +3 -exec rm {} \;
```

#### For Custom PostgreSQL dump (.dump):
```bash
#!/bin/bash
DB_NAME="odoo18"
BACKUP_DIR="/opt/odoo18/backups"
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_FILE="$BACKUP_DIR/$DB_NAME-$DATE.dump"

sudo -u postgres pg_dump -Fc $DB_NAME -f $BACKUP_FILE
chown odoo18:odoo18 $BACKUP_FILE
chmod 640 $BACKUP_FILE

echo "Backup for $DB_NAME completed at $DATE" >> /var/log/odoo_backup.log
find $BACKUP_DIR -type f -name "*.dump" -mtime +3 -exec rm {} \;
```

### Make the script executable:
```bash
sudo chmod +x /usr/local/bin/odoo_backup.sh
```

### Schedule the backup with cron:
```bash
sudo crontab -u odoo18 -e
```
Add this line to schedule a daily backup at 2 AM:
```bash
0 2 * * * /usr/local/bin/odoo_backup.sh >> /var/log/odoo_backup.log 2>&1
```

### Run the script manually to verify:
```bash
sudo /usr/local/bin/odoo_backup.sh
```

---

## Step 3: Restoring the Database from a Backup
### Switch to PostgreSQL user:
```bash
sudo -i -u postgres
```
### Create a new database:
```bash
createdb -O odoo18 odoo18
```
### Restore the backup using pg_restore (if using .dump file):
```bash
pg_restore -d odoo18 -U odoo18 -v /opt/odoo18/backups/odoo18-2025-02-17_10-17-49.dump
```
### Exit PostgreSQL user:
```bash
exit
```
### Restart Odoo:
```bash
sudo systemctl restart odoo18
```

---

## Step 4: Matching Server Time with Local Time
### Check the current time zone:
```bash
timedatectl
```
### Set the correct time zone (example for America/New_York):
```bash
sudo timedatectl set-timezone America/New_York
```
### Enable NTP and sync time:
```bash
sudo timedatectl set-ntp on
sudo systemctl restart systemd-timesyncd
```
### Verify time synchronization:
```bash
timedatectl status
```

---

## Step 5: Creating a Log File for the Backup Script
### Create the log file and set permissions:
```bash
sudo touch /var/log/odoo_backup.log
sudo chown odoo18:odoo18 /var/log/odoo_backup.log
sudo chmod 640 /var/log/odoo_backup.log
```
### Ensure the backup script logs to the file:
```bash
echo "Backup for $DB_NAME completed at $DATE" >> /var/log/odoo_backup.log
```
### Run the backup script manually to verify logging:
```bash
sudo /usr/local/bin/odoo_backup.sh
```
### Check the log file:
```bash
cat /var/log/odoo_backup.log
```

---

## Step 6: Fixing Permissions on Backup Directory
### Grant odoo18 and root permissions to access and execute the backup script:
```bash
sudo chown -R odoo18:odoo18 /opt/odoo18/backups
sudo chmod -R 755 /opt/odoo18/backups
sudo chmod +x /usr/local/bin/odoo_backup.sh
sudo chown root:odoo18 /usr/local/bin/odoo_backup.sh
```
### Verify permissions:
```bash
ls -ld /opt/odoo18/backups
```