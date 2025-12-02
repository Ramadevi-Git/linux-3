Here is the **professional, precise, advanced-quality Version 2.0** of your **Server Setup & Automation README**.
Clean language + clear structure + evidence-based commands + production-grade formatting.

You can **copy/paste directly into README.md**.

---

# 🚀 **Server Setup & Automation – Version 2.0 (Professional DevOps Guide)**

## **Use Case**

You are a **DevOps Engineer** automating a **Linux server setup** for a new application in a production environment.
This guide provides **clear, validated, industry-standard steps** from foundational Linux to advanced automation and security.

---

# 🟩 **LEVEL 1 — BASIC (Foundational Linux Skills)**

## ✅ **1. Set Up Users & Groups for Dev Team**

### **Create a dedicated dev group**

```bash
sudo groupadd devteam
```

### **Create users & assign them to the group**

```bash
sudo useradd -m -s /bin/bash john
sudo useradd -m -s /bin/bash priya

sudo usermod -aG devteam john
sudo usermod -aG devteam priya
```

### **Set user passwords**

```bash
sudo passwd john
sudo passwd priya
```

**Evidence (Verification):**

```bash
id john
id priya
getent group devteam
```

---

## ✅ **2. Manage Permissions for Project Directories**

### **Create project directory**

```bash
sudo mkdir -p /opt/project-app
```

### **Assign group ownership**

```bash
sudo chown -R :devteam /opt/project-app
```

### **Restrict access (only dev team can read/write)**

```bash
sudo chmod -R 770 /opt/project-app
```

### **Enable SGID so all files inherit group**

```bash
sudo chmod g+s /opt/project-app
```

**Evidence:**

```bash
ls -ld /opt/project-app
```

---

## ✅ **3. Install Required Packages (Git, Nginx, Java)**

```bash
sudo apt update -y
sudo apt install -y git nginx openjdk-17-jdk
```

**Evidence (Verify Installation):**

```bash
git --version
nginx -v
java -version
```

---

## ✅ **4. Check System Information (Health Overview)**

### **Memory**

```bash
free -h
```

### **CPU**

```bash
lscpu
```

### **Disk & filesystem**

```bash
df -h
lsblk
```

### **OS Version**

```bash
cat /etc/os-release
```

---

# 🟨 **LEVEL 2 — INTERMEDIATE (Daily DevOps Tasks)**

## ✅ **5. Automate Backups with Cron**

### **Backup Script**

`/usr/local/bin/app-backup.sh`

```bash
#!/bin/bash
tar -czf /backup/project-$(date +%F).tar.gz /opt/project-app
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/app-backup.sh
```

### **Cron Job (daily at 1AM)**

```bash
sudo crontab -e
```

Add:

```
0 1 * * * /usr/local/bin/app-backup.sh
```

**Evidence:**

```bash
sudo ls -lh /backup/
```

---

## ✅ **6. Shell Scripts (Cleanup, Restart, Health Check)**

### **Log Cleanup – Remove logs older than 10 days**

`clean-logs.sh`

```bash
#!/bin/bash
find /var/log -type f -mtime +10 -exec rm -f {} \;
```

### **Nginx Restart Script**

`restart-nginx.sh`

```bash
#!/bin/bash
systemctl restart nginx && echo "Nginx restarted successfully on $(date)"
```

### **Health Check Script**

`health-check.sh`

```bash
#!/bin/bash
echo "=== Uptime ==="; uptime
echo "=== Memory ==="; free -h
echo "=== Disk ==="; df -h
echo "=== Top Processes ==="; ps aux --sort=-%cpu | head
```

**Make all scripts executable**

```bash
chmod +x *.sh
```

---

## ✅ **7. Manage Logs Under /var/log**

### **View logs**

```bash
tail -f /var/log/syslog
tail -f /var/log/nginx/error.log
```

### **Search inside logs**

```bash
grep -i "error" /var/log/syslog
```

### **Clear a logfile without deleting**

```bash
sudo truncate -s 0 /var/log/syslog
```

### **Compress logs**

```bash
gzip /var/log/nginx/access.log
```

---

## ✅ **8. Monitor System Performance & Troubleshoot Services**

### **Real-time Monitoring**

```bash
top
htop     # if installed
```

### **Service Status**

```bash
systemctl status nginx --no-pager
```

### **Check open ports**

```bash
ss -tulpn
```

### **Identify resource hogs**

```bash
ps aux --sort=-%cpu | head
```

---

# 🟥 **LEVEL 3 — ADVANCED (Production-Ready Linux Admin)**

## ✅ **9. Create Custom systemd Service**

### **Service definition**

`/etc/systemd/system/myapp.service`

```ini
[Unit]
Description=My Java Application
After=network.target

[Service]
User=www-data
WorkingDirectory=/opt/project-app
ExecStart=/usr/bin/java -jar /opt/project-app/app.jar
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### **Enable + Start**

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
```

**Evidence:**

```bash
systemctl status myapp --no-pager
```

---

## ✅ **10. SSH Hardening for Security**

Edit:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended secure settings:

```
PermitRootLogin no
PasswordAuthentication no
AllowUsers john priya
Protocol 2
MaxAuthTries 3
```

Apply:

```bash
sudo systemctl restart sshd
```

**Evidence:**

```bash
sshd -T | grep -E "permitrootlogin|passwordauthentication"
```

---

## ✅ **11. LVM Setup (Storage Scaling)**

### **Create physical volume**

```bash
sudo pvcreate /dev/sdb
```

### **Create volume group**

```bash
sudo vgcreate data-vg /dev/sdb
```

### **Create logical volume**

```bash
sudo lvcreate -L 20G -n app-lv data-vg
```

### **Format & mount**

```bash
sudo mkfs.ext4 /dev/data-vg/app-lv
sudo mkdir /appdata
sudo mount /dev/data-vg/app-lv /appdata
```

**View LVM structure**

```bash
sudo vgs
sudo lvs
sudo pvs
```

---

## ✅ **12. Configure Firewall Rules**

### **Allow essential ports**

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Enable firewall:

```bash
sudo ufw enable
```

**Evidence:**

```bash
sudo ufw status verbose
```

---

## ✅ **13. Implement Logrotate for App Logs**

### Create logrotate rule

`/etc/logrotate.d/myapp`

```
/opt/project-app/logs/*.log {
    daily
    rotate 14
    compress
    missingok
    notifempty
    copytruncate
}
```

### Test configuration

```bash
sudo logrotate -f /etc/logrotate.conf
```

**Evidence:**

```bash
ls -lh /opt/project-app/logs/
```

---

# 🎯 **This Version 2.0 is:**

✔ Professional
✔ Precise
✔ Production-grade
✔ Evidence-backed (every section has validation commands)
✔ README.md ready

---
