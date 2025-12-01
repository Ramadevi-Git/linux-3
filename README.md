Linux.demo
Level 1 – Basic (Foundational Skills)
Set up users, groups for dev team
sudo useradd rama
sudo useradd john
sudo useradd priya

# Add users to dev group
sudo usermod -aG dev rama
sudo usermod -aG dev john

# Add users to qa group
sudo usermod -aG qa priya
Manage permissions for project directories
sudo groupadd devteam
sudo usermod -aG devteam username1
sudo usermod -aG devteam username2
groups username1
sudo mkdir -p /projects/app1
sudo chown -R :devteam /projects/app1
sudo chmod -R 770 /projects/app1
sudo chmod g+s /projects/app1
sudo chmod +t /projects/app1
ls -ld /projects/app1

-Install required packages (git, nginx, java)
```bash
sudo yum update -y

# Install Git
sudo yum install -y git

# Install Nginx
sudo amazon-linux-extras install nginx1 -y
sudo systemctl enable nginx
sudo systemctl start nginx

# Install Java (OpenJDK 11)
sudo yum install -y java-11-openjdk java-11-openjdk-devel

# Verify installations
git --version
nginx -v
java -version

-Check system info (memory, CPU, disks)
```bash
free -h
watch -n 1 free -h
cat /proc/meminfo
lscpu
cat /proc/cpuinfo
uptime
top
df -h
lsblk
sudo fdisk -l
iostat
sudo yum install sysstat -y
sudo apt install sysstat -y
sudo dmidecode | less
ip a
ip r
nload

## Level 2 – Intermediate (Daily DevOps Tasks)
- Automate backups with Cron
```bash
sudo nano /usr/local/bin/project-backup.sh
#!/bin/bash

# Source directory
SOURCE="/projects/app1"

# Backup directory
DEST="/backup/app1"

# Create backup directory if not exists
mkdir -p $DEST

# Filename: app1-backup-YYYYMMDD-HHMM.tar.gz
DATE=$(date +%Y%m%d-%H%M)
FILENAME="app1-backup-$DATE.tar.gz"

# Create archive
tar -czf $DEST/$FILENAME $SOURCE

# Optional: Delete backups older than 7 days
find $DEST -type f -mtime +7 -name "*.tar.gz" -delete
sudo chmod +x /usr/local/bin/project-backup.sh
sudo mkdir -p /backup/app1
sudo chown $USER:$USER /backup/app1
crontab -e
0 2 * * * /usr/local/bin/project-backup.sh >> /var/log/project-backup.log 2>&1
0 * * * * /usr/local/bin/project-backup.sh
0 23 * * SUN /usr/local/bin/project-backup.sh
*/30 * * * * /usr/local/bin/project-backup.sh
systemctl status crond   # RHEL/CentOS/Amazon Linux
systemctl status cron    # Ubuntu/Debian
sudo cat /var/log/cron
sudo grep CRON /var/log/syslog

- Create shell scripts: Log cleanup, service restart, health checks
```bash
#!/bin/bash

LOG_DIR="/var/log/app"
RETENTION_DAYS=7
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "$DATE - Starting log cleanup..." >> /var/log/log-cleanup.log

# Cleanup
find $LOG_DIR -type f -mtime +$RETENTION_DAYS -print -delete >> /var/log/log-cleanup.log 2>&1

echo "$DATE - Log cleanup completed." >> /var/log/log-cleanup.log
sudo chmod +x /usr/local/bin/log-cleanup.sh
#!/bin/bash

SERVICE="nginx"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "$DATE - Restarting $SERVICE..." >> /var/log/service-restart.log

# Restart service
systemctl restart $SERVICE

# Check status
if systemctl is-active --quiet $SERVICE; then
    echo "$DATE - $SERVICE restarted successfully." >> /var/log/service-restart.log
else
    echo "$DATE - ERROR: $SERVICE failed to restart!" >> /var/log/service-restart.log
    exit 1
fi
sudo chmod +x /usr/local/bin/service-restart.sh
SERVICE="jenkins"
#!/bin/bash

URL="http://localhost:80"
DATE=$(date '+%Y-%m-%d %H:%M:%S')
LOGFILE="/var/log/health-check.log"

STATUS=$(curl -s -o /dev/null -w "%{http_code}" $URL)

if [ "$STATUS" -eq 200 ]; then
    echo "$DATE - Health Check OK (HTTP $STATUS)" >> $LOGFILE
else
    echo "$DATE - Health Check FAILED (HTTP $STATUS)" >> $LOGFILE
    # Optional auto-restart:
    # systemctl restart nginx
fi
sudo chmod +x /usr/local/bin/health-check.sh
crontab -e
0 2 * * * /usr/local/bin/log-cleanup.sh
0 3 * * * /usr/local/bin/service-restart.sh
*/5 * * * * /usr/local/bin/health-check.sh





 



