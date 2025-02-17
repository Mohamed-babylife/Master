
# Odoo Configuration and Backup Guide

## 1. Making Odoo Read Files from a Custom Directory:

To configure Odoo to read files from a custom directory, follow these steps:

### 1.1 Create the Directory and Set Permissions

```bash
sudo mkdir -p /mnt/extra-addons/enterprise
sudo chown -R odoo18:odoo18 /mnt/extra-addons
sudo chmod -R 755 /mnt/extra-addons
```

### 1.2 Update the Odoo Configuration File

Edit the Odoo configuration file (`/etc/odoo18.conf`) to add the custom directory to the `addons_path`:

```ini
addons_path = /opt/odoo18/odoo18/addons,/mnt/extra-addons/enterprise
```

### 1.3 Restart Odoo Service

Reload and restart Odoo to apply the changes:

```bash
sudo systemctl daemon-reload
sudo systemctl restart odoo18
```


## 2. Setting Up Automatic Database Backup:

To automate database backups for Odoo, follow the steps below:

### 2.1 Create Backup Directory and Set Permissions

```bash
sudo mkdir -p /var/backups/odoo
sudo chown odoo18:odoo18 /var/backups/odoo
sudo chmod 750 /var/backups/odoo
```

### 2.2 Create Backup Script

Create a new backup script (`/usr/local/bin/odoo_backup.sh`):

```bash
#!/bin/bash

# Database credentials
DB_NAME="your_database_name"
BACKUP_DIR="/var/backups/odoo"
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_FILE="$BACKUP_DIR/$DB_NAME-$DATE.sql.gz"

# Perform PostgreSQL database backup
sudo -u postgres pg_dump $DB_NAME | gzip > $BACKUP_FILE

# Set correct permissions
chown odoo18:odoo18 $BACKUP_FILE
chmod 640 $BACKUP_FILE

# Log the backup
echo "Backup for $DB_NAME completed at $DATE" >> /var/log/odoo_backup.log

# Remove backups older than 7 days
find $BACKUP_DIR -type f -name "*.sql.gz" -mtime +7 -exec rm {} \;
```

### 2.3 Make the Script Executable

```bash
sudo chmod +x /usr/local/bin/odoo_backup.sh
```

### 2.4 Schedule the Backup Using Cron

Edit the Odoo user’s crontab to schedule the backup at 2 AM every day:

```bash
sudo crontab -u odoo18 -e
```

Add the following line:

```bash
0 2 * * * /usr/local/bin/odoo_backup.sh >> /var/log/odoo_backup.log 2>&1
```

### 2.5 Run the Script Manually to Verify

```bash
sudo /usr/local/bin/odoo_backup.sh
```


## 3. Restoring the Database from a Backup:

To restore a database backup into a new database, follow these steps:

### 3.1 Switch to PostgreSQL User

```bash
sudo -i -u postgres
```

### 3.2 Create a New Database

```bash
createdb -O odoo18 odoo18
```

### 3.3 Restore the Backup Using `pg_restore`

```bash
pg_restore -d odoo18 -U odoo18 -v /opt/odoo18/backups/odoo18-2025-02-12_00-00-01.dump
```

### 3.4 Exit PostgreSQL User

```bash
exit
```

### 3.5 Restart Odoo

```bash
sudo systemctl restart odoo18
```


## 4. Matching Server Time with Local Time Zone:

To ensure the server’s time matches the local time zone:

### 4.1 Check the Current Time Zone

```bash
timedatectl
```

### 4.2 Set the Correct Time Zone

For example, to set the time zone to `America/New_York`:

```bash
sudo timedatectl set-timezone America/New_York
```

### 4.3 Enable NTP and Sync Time

```bash
sudo timedatectl set-ntp on
sudo systemctl restart systemd-timesyncd
```

### 4.4 Verify Time Synchronization

```bash
timedatectl status
```


## 5. Creating a Log File for the Backup Script:

To set up logging for the backup process, follow these steps:

### 5.1 Create the Log File and Set Permissions

```bash
sudo touch /var/log/odoo_backup.log
sudo chown odoo18:odoo18 /var/log/odoo_backup.log
sudo chmod 640 /var/log/odoo_backup.log
```

### 5.2 Ensure Backup Script Logs the Output

In the backup script (`/usr/local/bin/odoo_backup.sh`), include this line to log backup information:

```bash
echo "Backup for $DB_NAME completed at $DATE" >> /var/log/odoo_backup.log
```

### 5.3 Verify Logging by Running the Backup Script

```bash
sudo /usr/local/bin/odoo_backup.sh
```

### 5.4 Check the Log File

```bash
cat /var/log/odoo_backup.log
```

---

By following these steps, you'll have a complete setup for reading files from a custom directory, automating database backups, restoring from backups, synchronizing time, and creating log files for backup operations in Odoo.
