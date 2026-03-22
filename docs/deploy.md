# ACSCloud Deployment Guide

## Overview

This guide covers the complete deployment process for ACSCloud on Linux servers. The system consists of multiple components working together to provide TR-069 device management capabilities.

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Internet / WAN                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Nginx (Port 9090)                    │
│                   Reverse Proxy + SSL Termination           │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ ACS Service   │    │  API Service  │    │  Static Files │
│   (Port 7711) │    │   (Port 8888) │    │   (Port 9090) │
└───────────────┘    └───────────────┘    └───────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    ┌─────────────┐                          │
│                    │    MySQL    │                          │
│                    │   (Port    │                          │
│                    │   3306)     │                          │
│                    └─────────────┘                          │
│                    ┌─────────────┐                          │
│                    │    Redis    │                          │
│                    │   (Port    │                          │
│                    │   6379)     │                          │
│                    └─────────────┘                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      CPE Devices                            │
│      (ONT, Router, Gateway, IoT Devices via Port 80/443)    │
└─────────────────────────────────────────────────────────────┘
```

## Prerequisites

### Hardware Requirements

| Device Scale | CPU | RAM | Disk | Network |
|--------------|-----|-----|------|---------|
| < 1,000 devices | 2 cores | 4 GB | 50 GB | 100Mbps |
| 1,000 - 10,000 devices | 4 cores | 8 GB | 100 GB | 1Gbps |
| 10,000 - 100,000 devices | 8 cores | 16 GB | 200 GB | 1Gbps |
| > 100,000 devices | 16 cores+ | 32 GB+ | 500 GB+ | 10Gbps |

### Software Requirements

| Component | Version | Notes |
|-----------|---------|-------|
| OS | CentOS 7+ / Ubuntu 18+ / Debian 10+ | 64-bit Linux |
| JDK | 1.8 (JDK 8u231+) | Must be installed |
| MySQL | 5.7+ | Must set `lower_case_table_names=1` |
| Redis | 3.0+ | Default port 6379 |
| Nginx | 1.12+ | Optional, for reverse proxy |

---

## Step 1: Environment Setup

### 1.1 Create Directory Structure

```bash
mkdir -p /home/acs/{log,api/log,firmware,configfile}
chmod 777 -R /home/acs
```

| Directory | Purpose |
|-----------|---------|
| /home/acs/log | ACS service logs |
| /home/acs/api/log | API service logs |
| /home/acs/firmware | Firmware upgrade files |
| /home/acs/configfile | Configuration files for distribution |

### 1.2 Install JDK 1.8

```bash
cd /home/acs
tar xvfz jdk-8u231-linux-x64.tar.gz
```

Configure environment variables in `/etc/profile`:
```bash
export JAVA_HOME=/home/acs/jdk1.8.0_231
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

Apply changes:
```bash
source /etc/profile
```

Verify installation:
```bash
java -version
```

---

## Step 2: Database Setup

### 2.1 Install MySQL 5.7

For CentOS 7:
```bash
yum install -y mysql-community-server
systemctl start mysqld
systemctl enable mysqld
```

### 2.2 Configure MySQL

**Important**: Edit `/etc/my.cnf` to enable case-insensitive table names:

```ini
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
lower_case_table_names=1
```

Restart MySQL:
```bash
systemctl restart mysqld
```

### 2.3 Initialize Database

```bash
mysql -u root -p

-- Create database
CREATE DATABASE IF NOT EXISTS ACS DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
USE ACS;

-- Import initialization script
SOURCE /home/acs/acs.sql;

-- Create user (optional)
CREATE USER 'acs'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON ACS.* TO 'acs'@'localhost';
FLUSH PRIVILEGES;
```

**Default Credentials**: admin / 123456 (change after first login)

---

## Step 3: Redis Setup

```bash
yum install redis
systemctl start redis
systemctl enable redis
```

Verify:
```bash
redis-cli ping
# Should return: PONG
```

---

## Step 4: Application Configuration

### 4.1 Configure Database Connection

Edit `application.yml` or create `api.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/ACS?useUnicode=true&characterEncoding=utf8&useSSL=false
spring.datasource.username=root
spring.datasource.password=P@ssword2019
```

### 4.2 Configure Image Storage

```properties
global.imgHost=http://your-domain:8888/api/user/
global.imgPath=/opt/
```

### 4.3 Required Directories

```bash
mkdir -p /opt
chmod 777 /opt
```

---

## Step 5: Start Services

### 5.1 Start ACS Service

```bash
cd /home/acs
chmod +x start.sh
./start.sh
```

Verify startup:
```bash
tail -f /home/acs/log/acs.log
```

### 5.2 Start API Service

```bash
cd /home/acs/api
chmod +x startApi.sh
./startApi.sh
```

### 5.3 Startup Script Reference

```bash
#!/bin/sh
export JAVA_HOME="/home/acs/jdk1.8.0_231"
export JRE_HOME="$JAVA_HOME/jre"
export PATH=$JAVA_HOME/bin:$PATH

# JVM parameters - adjust based on server resources
export JAVA_OPTS=" \
  -server \
  -Xms512M -Xmx512M \
  -Xmn256M \
  -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m \
  -XX:SurvivorRatio=4 \
  -Xss256k \
  -XX:+UseParNewGC \
  -XX:+UseConcMarkSweepGC \
  -XX:CMSInitiatingOccupancyFraction=60 \
  -XX:+PrintGCDetails \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/home/acs/log/oom-error_dump.log"

nohup $JAVA_HOME/bin/java $JAVA_OPTS \
  -Dloader.path=/home/acs/api/lib \
  -jar /home/acs/api/cwmpcloudapi-0.0.1-SNAPSHOT.jar \
  >/dev/null 2>&1 &
```

---

## Step 6: Nginx Configuration

### 6.1 Install Nginx

```bash
yum install nginx
```

### 6.2 Configure Nginx

Edit `/etc/nginx/nginx.conf`:

```nginx
upstream acscloud {
    ip_hash;
    server 127.0.0.1:7711;
}

server {
    listen       9090;
    server_name  your-domain.com;
    root         /usr/share/nginx/html;

    # Web Console
    location / {
        proxy_pass http://acscloud;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        client_max_body_size 100m;
    }

    # Device Endpoint (TR-069 CWMP)
    location /ACS-server/ACS {
        proxy_pass http://acscloud/ACS-server/ACS;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        client_max_body_size 100m;
    }

    # RMS Super Admin (optional)
    location /RMS-server/RMS {
        proxy_pass http://acscloud/ACS-server/ACS/superrms;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        client_max_body_size 100m;
    }
}
```

### 6.3 Start Nginx

```bash
systemctl start nginx
systemctl enable nginx
```

---

## Step 7: Firewall Configuration

### 7.1 CentOS 7 (firewalld)

```bash
firewall-cmd --permanent --add-port=9090/tcp   # Web Console
firewall-cmd --permanent --add-port=8888/tcp   # API Service
firewall-cmd --permanent --add-port=80/tcp     # HTTP (for CPE devices)
firewall-cmd --permanent --add-port=443/tcp    # HTTPS (for CPE devices)
firewall-cmd --reload
```

### 7.2 Ubuntu (ufw)

```bash
ufw allow 9090/tcp
ufw allow 8888/tcp
ufw allow 80/tcp
ufw allow 443/tcp
```

---

## Step 8: License Configuration

### 8.1 Obtain CPU Information

Visit: `http://your-domain:9090/acscloud/newaccount.viewCpu.action`

Copy the displayed CPU information.

### 8.2 Request License

Contact the author with your CPU information to obtain a license key.

### 8.3 Apply License

Visit: `http://your-domain:9090/acscloud/newaccount.applyLicense.action?licenseKey=YOUR_LICENSE_KEY`

---

## Step 9: Auto Start on Boot

Create `/home/acs/autostartall.sh`:

```bash
#!/bin/bash
/home/acs/start.sh
/home/acs/api/startApi.sh
```

Set permissions:
```bash
chmod +x /home/acs/autostartall.sh
```

Add to `/etc/rc.d/rc.local`:

```bash
echo "/home/acs/autostartall.sh" >> /etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local
```

---

## Step 10: Verify Deployment

### 10.1 Check Services

```bash
# Check if services are running
ps aux | grep java
netstat -tlnp | grep -E '7711|8888|9090'
```

### 10.2 Test Endpoints

| Service | URL | Expected |
|---------|-----|----------|
| Admin Console | http://your-domain:9090/acscloud | Login page |
| Device Endpoint | http://your-domain:9090/ACS-server/ACS | SOAP response |
| API Health | http://your-domain:8888/api/ | JSON response |

### 10.3 Test Device Registration

Configure a CPE device with ACS URL:
```
http://your-domain:9090/ACS-server/ACS
```

Device should appear in the admin console after successful registration.

---

## Troubleshooting

### Device Cannot Register

1. Check firewall: `firewall-cmd --list-all`
2. Verify ACS URL is accessible from device
3. Check logs: `tail -f /home/acs/log/acs.log`
4. Verify device supports TR-069 protocol

### Database Connection Failed

1. Check MySQL is running: `systemctl status mysqld`
2. Verify credentials in configuration
3. Confirm `lower_case_table_names=1` is set
4. Check MySQL logs: `/var/log/mysqld.log`

### API Returns 500 Error

1. Check API logs: `tail -f /home/acs/api/log/api.log`
2. Verify Redis is running: `redis-cli ping`
3. Check database connectivity

### Firmware Upgrade Failed

1. Verify firmware URL is publicly accessible
2. Check firmware file size matches configuration
3. Confirm device model supports the firmware version
4. Check device is online

---

## Security Recommendations

1. **Change Default Passwords** - Change admin password after first login
2. **Enable HTTPS** - Configure SSL certificate in Nginx
3. **Firewall** - Only expose necessary ports
4. **Regular Updates** - Keep system and dependencies updated
5. **Backup** - Regular database backups
6. **Monitoring** - Set up monitoring for service health

---

## Backup & Recovery

### Database Backup

```bash
mysqldump -u root -p ACS > ACS_backup_$(date +%Y%m%d).sql
```

### Restore Database

```bash
mysql -u root -p ACS < ACS_backup_20240101.sql
```

---

## Performance Tuning

### For Large-Scale Deployments

1. **Increase JVM Heap**:
   ```bash
   -Xms4G -Xmx4G -Xmn2G
   ```

2. **Optimize MySQL**:
   ```ini
   innodb_buffer_pool_size = 2G
   max_connections = 500
   ```

3. **Enable Redis Persistence**:
   ```bash
   redis-cli config set appendonly yes
   ```

---

## Support

For deployment issues, please create an issue on GitHub or contact the author.
