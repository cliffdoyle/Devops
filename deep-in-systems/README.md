# Deep-in-System Project - Server Administration

## Author
**Username:** cliff  
**Hostname:** cliff-host  
**Date:** December 2024

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Virtual Machine Setup](#virtual-machine-setup)
3. [Network Configuration](#network-configuration)
4. [Security Configuration](#security-configuration)
5. [User Management](#user-management)
6. [Services Installation](#services-installation)
7. [Backup System](#backup-system)
8. [Testing & Verification](#testing-and-verification)
9. [Troubleshooting](#troubleshooting)

---

## Project Overview

This project demonstrates the complete setup and administration of an Ubuntu 24.04 LTS server with:
- Secure SSH configuration
- Firewall management
- User access control
- FTP server with restricted access
- MySQL database server
- WordPress CMS installation
- Automated backup system

---

## Virtual Machine Setup

### System Specifications
- **OS:** Ubuntu 24.04.1 LTS
- **VM Disk Size:** 30GB
- **Hostname:** cliff-host
- **Username:** cliff

### Disk Partitioning

| Partition | Size | Purpose |
|-----------|------|---------|
| swap | 4GB | Swap space |
| / | 15GB | Root filesystem |
| /home | 5GB | User home directories |
| /backup | 6GB | Backup storage |

### Installation Commands
```bash
# During Ubuntu installation, manual partitioning was selected
# Partitions were created according to specifications above
# Hostname set to: cliff-host
# User created: cliff
```

---

## Network Configuration

### Static IP Configuration

**IP Address:** 10.200.45.50  
**Gateway:** 10.200.45.1  
**DNS:** 8.8.8.8

### Configuration File
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Configuration:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 10.200.45.50/24
      gateway4: 10.200.45.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Apply configuration:
```bash
sudo netplan apply
```

### Verification
```bash
ip addr show enp0s3
ping -c 5 google.com
```

---

## Security Configuration

### SSH Security

#### Change SSH Port to 2222
```bash
sudo nano /etc/ssh/sshd_config
```

Modifications:
```conf
Port 2222
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication yes
```

Restart SSH:
```bash
sudo systemctl restart sshd
```

### Firewall Configuration

#### Enable and Configure UFW
```bash
sudo ufw enable
sudo ufw allow 2222/tcp comment 'SSH'
sudo ufw allow 80/tcp comment 'HTTP for WordPress'
sudo ufw allow 21/tcp comment 'FTP'
sudo ufw allow 40000:40100/tcp comment 'FTP Passive Mode'
```

#### Verify Firewall Status
```bash
sudo ufw status numbered
```

**Open Ports Justification:**
- **2222/tcp:** SSH access for remote administration
- **80/tcp:** HTTP for WordPress website
- **21/tcp:** FTP control connection
- **40000-40100/tcp:** FTP passive mode data transfer

---

## User Management

### User 1: luffy (SSH Key Authentication, Sudoer)
```bash
# Create user
sudo adduser luffy

# Add to sudo group
sudo usermod -aG sudo luffy

# Setup SSH key authentication
sudo mkdir -p /home/luffy/.ssh
sudo nano /home/luffy/.ssh/authorized_keys
# Paste public key here

# Set permissions
sudo chown -R luffy:luffy /home/luffy/.ssh
sudo chmod 700 /home/luffy/.ssh
sudo chmod 600 /home/luffy/.ssh/authorized_keys
```

**SSH Key Generated on Host:**
```bash
ssh-keygen -t ed25519 -C "luffy_key" -f ~/.ssh/luffy_final
```

**Test Connection:**
```bash
ssh -p 2222 luffy@10.200.45.50
```

### User 2: zoro (Password Authentication, Non-Sudoer)
```bash
# Create user
sudo adduser zoro
# Set password during creation: [custom password]
```

**Test Connection:**
```bash
ssh -p 2222 zoro@10.200.45.50
```

### User 3: nami (FTP Only, Read-Only /backup Access)
```bash
# Create user
sudo adduser nami
# Set password: [custom password]

# Disable shell login
sudo usermod -s /usr/sbin/nologin nami
```

---

## Services Installation

### 1. FTP Server (vsftpd)

#### Installation
```bash
sudo apt update
sudo apt install vsftpd -y
```

#### Configuration
```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
sudo nano /etc/vsftpd.conf
```

**Key Configuration Lines:**
```conf
listen=YES
anonymous_enable=NO
local_enable=YES
write_enable=NO
chroot_local_user=YES
allow_writeable_chroot=YES
local_root=/backup

# Passive mode
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100
pasv_address=10.200.45.50
```

#### Restart Service
```bash
sudo systemctl restart vsftpd
sudo systemctl enable vsftpd
```

#### Test FTP Access
```bash
ftp 10.200.45.50
# Login: nami
# Password: [custom password]
```

---

### 2. MySQL Database Server

#### Installation
```bash
sudo apt install mysql-server -y
```

#### Secure Installation
```bash
sudo mysql_secure_installation
```

Configuration choices:
- VALIDATE PASSWORD: No
- Remove anonymous users: Yes
- Disallow root login remotely: Yes
- Remove test database: Yes
- Reload privilege tables: Yes

#### Create WordPress Database and User
```bash
sudo mysql
```
```sql
CREATE DATABASE wordpress;
CREATE USER 'cliff'@'localhost' IDENTIFIED BY 'kombewa';
GRANT ALL PRIVILEGES ON wordpress.* TO 'cliff'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### Verify Configuration
```bash
sudo mysql -e "SELECT user,host FROM mysql.user;"
```

**Security Notes:**
- Root user only accessible from localhost
- No remote MySQL connections allowed (bind-address=127.0.0.1)
- Dedicated user 'cliff' for WordPress with limited privileges

---

### 3. Apache Web Server and PHP

#### Installation
```bash
sudo apt install apache2 -y
sudo apt install php libapache2-mod-php php-mysql -y
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
```

#### Enable and Start Apache
```bash
sudo systemctl enable apache2
sudo systemctl restart apache2
```

---

### 4. WordPress Installation

#### Download WordPress
```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
```

#### Copy to Web Root
```bash
sudo cp -r wordpress/* /var/www/html/
sudo rm /var/www/html/index.html
```

#### Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

#### Configure WordPress
```bash
cd /var/www/html/
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

**Database Configuration:**
```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'cliff' );
define( 'DB_PASSWORD', 'kombewa' );
define( 'DB_HOST', 'localhost' );
```

#### Secure wp-config.php
```bash
sudo chmod 640 /var/www/html/wp-config.php
sudo chown www-data:www-data /var/www/html/wp-config.php
```

#### Prevent Public Access to wp-config.php
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Add inside `<VirtualHost>` block:
```apache
<Files wp-config.php>
    Require all denied
</Files>
```

Restart Apache:
```bash
sudo systemctl restart apache2
```

#### Complete Installation via Browser
Navigate to: `http://10.200.45.50/`

Complete the WordPress installation wizard.

---

## Backup System

### Backup Script

#### Create Script
```bash
sudo nano /usr/local/bin/wordpress_backup.sh
```

**Script Content:**
```bash
#!/bin/bash

# Backup configuration
BACKUP_DIR="/backup"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
DB_NAME="wordpress"
DB_USER="cliff"
DB_PASS="kombewa"
LOG_FILE="/var/log/backup.log"
BACKUP_FILE="wordpress_db_backup_${DATE}.tar.gz"

# Create backup directory if it doesn't exist
mkdir -p ${BACKUP_DIR}

# Export MySQL database (without tablespaces to avoid privilege issues)
mysqldump -u${DB_USER} -p${DB_PASS} --single-transaction --no-tablespaces ${DB_NAME} > ${BACKUP_DIR}/wordpress_db_${DATE}.sql

# Create tar archive of the SQL dump
tar -czf ${BACKUP_DIR}/${BACKUP_FILE} -C ${BACKUP_DIR} wordpress_db_${DATE}.sql

# Remove the plain SQL file (keep only compressed)
rm ${BACKUP_DIR}/wordpress_db_${DATE}.sql

# Set proper permissions for FTP access
chmod 644 ${BACKUP_DIR}/${BACKUP_FILE}

# Log the backup
echo "$(date '+%Y-%m-%d %H:%M:%S') - Backup successful: ${BACKUP_FILE}" >> ${LOG_FILE}

# Optional: Remove backups older than 30 days to save space
find ${BACKUP_DIR} -name "wordpress_db_backup_*.tar.gz" -mtime +30 -delete
```

#### Make Executable
```bash
sudo chmod +x /usr/local/bin/wordpress_backup.sh
```

#### Test Script
```bash
sudo /usr/local/bin/wordpress_backup.sh
ls -lh /backup/
cat /var/log/backup.log
```

### Cron Job Configuration

#### Create Cron Job
```bash
sudo crontab -e
```

Add line:
```cron
0 0 * * * /usr/local/bin/wordpress_backup.sh
```

This runs daily at 00:00 (midnight).

#### Verify Cron Job
```bash
sudo crontab -l
```

---

## Testing and Verification

### System Services Status
```bash
sudo systemctl status apache2
sudo systemctl status mysql
sudo systemctl status vsftpd
sudo systemctl status sshd
```

### Network and Firewall
```bash
ip addr show
sudo ufw status numbered
ping -c 3 google.com
```

### SSH Access Tests
```bash
# From host machine:
ssh -p 2222 luffy@10.200.45.50  # Should work with key
ssh -p 2222 zoro@10.200.45.50   # Should work with password
ssh -p 2222 root@10.200.45.50   # Should be denied
```

### WordPress Tests
- Access site: `http://10.200.45.50/`
- Login to admin: `http://10.200.45.50/wp-admin`
- Test wp-config protection: `http://10.200.45.50/wp-config.php` (should return 403)

### FTP Tests
```bash
ftp 10.200.45.50
# Login: nami / [password]
# Commands:
ls
get wordpress_db_backup_[date].tar.gz
bye
```

### Backup System Tests
```bash
# Manual backup
sudo /usr/local/bin/wordpress_backup.sh

# Check backups
ls -lh /backup/

# Check log
cat /var/log/backup.log

# Verify cron
sudo crontab -l
```

### MySQL Security Tests
```bash
# Verify users
sudo mysql -e "SELECT user,host FROM mysql.user;"

# Test WordPress connection
mysql -u cliff -pkombewa wordpress -e "SHOW TABLES;"
```

---

## Troubleshooting

### Common Issues and Solutions

#### SSH Connection Issues
**Problem:** Cannot connect via SSH  
**Solution:**
```bash
sudo systemctl status sshd
sudo ufw allow 2222/tcp
sudo systemctl restart sshd
```

#### WordPress Database Connection Error
**Problem:** Error establishing database connection  
**Solution:**
- Verify MySQL is running: `sudo systemctl status mysql`
- Check credentials in `/var/www/html/wp-config.php`
- Test database connection: `mysql -u cliff -pkombewa wordpress`

#### FTP Timeout on ls Command
**Problem:** FTP ls command hangs  
**Solution:**
- Use active mode: In FTP, type `passive` to toggle
- Ensure passive ports open: `sudo ufw allow 40000:40100/tcp`
- Restart vsftpd: `sudo systemctl restart vsftpd`

#### Backup Script Fails
**Problem:** mysqldump permission denied  
**Solution:**
```bash
sudo mysql
GRANT SELECT, LOCK TABLES, SHOW VIEW ON wordpress.* TO 'cliff'@'localhost';
FLUSH PRIVILEGES;
```

#### Apache Not Starting
**Problem:** Apache fails to start  
**Solution:**
```bash
sudo apache2ctl configtest
sudo systemctl status apache2
sudo journalctl -u apache2 -n 50
```

---

## Summary

This project successfully demonstrates:
- ✅ Ubuntu server installation with custom partitioning
- ✅ Static network configuration
- ✅ SSH hardening (custom port, key-based auth, root disabled)
- ✅ Firewall configuration with justified rules
- ✅ User management with different access levels
- ✅ FTP server with restricted access
- ✅ MySQL database with security best practices
- ✅ WordPress CMS installation and configuration
- ✅ Automated backup system with cron scheduling
- ✅ All services tested and verified

---

## Key Commands Reference
```bash
# Service Management
sudo systemctl status [service]
sudo systemctl restart [service]
sudo systemctl enable [service]

# Firewall
sudo ufw status
sudo ufw allow [port]/tcp

# MySQL
sudo mysql
mysql -u [user] -p [database]

# User Management
sudo adduser [username]
sudo usermod -aG sudo [username]

# File Permissions
sudo chown [user]:[group] [file]
sudo chmod [permissions] [file]

# Logs
sudo tail -f /var/log/[logfile]
sudo journalctl -u [service]

# Cron
sudo crontab -e
sudo crontab -l

# Backup
sudo /usr/local/bin/wordpress_backup.sh
ls -lh /backup/
cat /var/log/backup.log
```

---

## Conclusion

All project requirements have been successfully implemented and tested. The server is secure, fully functional, and ready for production use with automated backups.

**VM Export:** deep-in-system.ova  
**SHA1 Hash:** [See deep-in-system.sha1 file]