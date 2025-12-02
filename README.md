Linux.demo
Level 1 – Basic (Foundational Skills)
Set up users, groups for dev team
sudo useradd rama
sudo useradd john
sudo useradd priya

![alt text](../eveidence/image-1.png)




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

- Manage logs under /var/log
```bash
ls -lh /var/log
cat /var/log/messages
tail -f /var/log/messages
nl /var/log/secure | less
cat /etc/logrotate.conf
/etc/logrotate.d/
sudo logrotate -f /etc/logrotate.conf
sudo nano /etc/logrotate.d/cb-logrotate
/var/log/cb.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 0644 root root
}
sudo find /var/log -type f -mtime +15 -exec rm -f {} \;
sudo truncate -s 0 /var/log/messages
gzip /var/log/secure-20240110
gunzip secure-20240110.gz
ls -l /var/log
sudo chown root:root /var/log/custom.log
sudo chmod 640 /var/log/custom.log
grep -i "error" /var/log/messages
grep "Failed password" /var/log/secure

- Monitor system performance and troubleshoot services
```bash
top
htop      # install: sudo yum/apt install htop -y
lscpu
uptime
free -h
ps aux --sort=-%mem | head
df -h
du -sh /*
iostat 1
ss -tulpn
nload
systemctl status nginx
systemctl status jenkins
systemctl status docker
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
journalctl -u nginx --since "10 minutes ago"
journalctl -u nginx -n 100
journalctl -u nginx -f
sudo nginx -t
sudo apachectl configtest
dockerd --debug
ps aux --sort=-%cpu | head
kill -9 <PID>
dmesg | grep -i oom
top -o %MEM
du -sh /var/log/* | sort -h
sudo journalctl --vacuum-size=100M
tail -f /var/log/messages        # RHEL/Amazon Linux
tail -f /var/log/syslog          # Ubuntu
tail -f /var/log/jenkins/jenkins.log
ss -tulpn | grep 8080
curl -I http://localhost

## Level 3 – Advanced (Production-Ready Linux Admin)
- Create custom systemd service for your application
```bash
#!/bin/bash
echo "Application starting..."
/usr/bin/java -jar /opt/myapp/myapp.jar
sudo chmod +x /opt/myapp/start.sh
sudo nano /etc/systemd/system/myapp.service
[Unit]
Description=My Custom Application Service
After=network.target

[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh

# Restart on failure
Restart=always
RestartSec=5

# Environment variables (optional)
Environment="JAVA_HOME=/usr/lib/jvm/jre"

# Log output (stored in journalctl)
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
sudo systemctl status myapp.service
journalctl -u myapp -f

- SSH hardening for security
```bash
sudo nano /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
sudo systemctl restart sshd
PermitRootLogin no
sudo systemctl restart sshd
Port 2222
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --reload
sudo systemctl restart sshd
AllowUsers devops adminuser
AllowGroups sshusers
HostKeyAlgorithms ssh-ed25519,ssh-rsa
KexAlgorithms curve25519-sha256
MACs hmac-sha2-512,hmac-sha2-256
Protocol 2
ClientAliveInterval 300
ClientAliveCountMax 2
sudo yum install epel-release -y
sudo yum install fail2ban -y
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status sshd
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="YOUR.IP.HERE" port protocol="tcp" port="22" accept' --permanent
sudo firewall-cmd --reload
sudo ufw allow from YOUR.IP.HERE to any port 22
sudo tail -f /var/log/secure
grep "Failed password" /var/log/secure
X11Forwarding no
sudo yum install google-authenticator -y
auth required pam_google_authenticator.so
ChallengeResponseAuthentication yes

- LVM setup for storage scaling
```bash
lsblk
sudo pvcreate /dev/sdb
sudo pvs
sudo vgcreate myvg /dev/sdb
sudo vgs
sudo lvcreate -L 10G -n mylv myvg
sudo lvs
sudo mkfs.ext4 /dev/myvg/mylv
sudo mount /dev/myvg/mylv /data
df -h
/dev/myvg/mylv   /data    ext4    defaults    0 0
sudo mount -a
sudo pvcreate /dev/sdc
sudo vgextend myvg /dev/sdc
sudo lvextend -L +5G /dev/myvg/mylv
sudo resize2fs /dev/myvg/mylv
sudo xfs_growfs /data
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay

- Configure firewall rules
```bash
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo systemctl status firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=3306/tcp
sudo firewall-cmd --permanent --remove-port=8080/tcp
sudo firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='10.0.0.5' accept"
sudo firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.1.10' drop"
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 8080/tcp
sudo ufw deny 23/tcp
sudo ufw deny from 192.168.1.50
sudo ufw reload
sudo ufw status verbose

- Implement logrotate for app logs
```bash
sudo nano /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
    create 0640 root root
}
sudo logrotate --debug /etc/logrotate.d/myapp
sudo logrotate -f /etc/logrotate.d/myapp
cat /etc/cron.daily/logrotate
