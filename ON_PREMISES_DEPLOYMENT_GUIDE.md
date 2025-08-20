# On-Premises Deployment Guide
## TimeTracker Pro Enterprise Self-Hosted Installation

### Overview
This guide provides comprehensive instructions for deploying TimeTracker Pro on your own infrastructure, moving from Replit's cloud environment to a self-hosted enterprise deployment.

---

## Prerequisites and System Requirements

### Hardware Requirements
**Minimum Specifications:**
- **CPU**: 4 cores, 2.4 GHz
- **RAM**: 8 GB minimum, 16 GB recommended
- **Storage**: 100 GB SSD minimum, 500 GB recommended
- **Network**: 1 Gbps connection

**Recommended Production Specifications:**
- **CPU**: 8 cores, 3.0 GHz or higher
- **RAM**: 32 GB
- **Storage**: 1 TB NVMe SSD with RAID 1
- **Network**: 10 Gbps connection
- **Load Balancer**: Hardware or software load balancer for high availability

### Operating System Support
**Supported Platforms:**
- Ubuntu 20.04 LTS or 22.04 LTS (recommended)
- CentOS 8 / RHEL 8+
- Docker with Kubernetes (containerized deployment)
- Windows Server 2019+ (with Docker Desktop)

---

## Infrastructure Architecture

### Single Server Deployment
```
┌─────────────────────────────────────────┐
│            Application Server           │
├─────────────────────────────────────────┤
│  Reverse Proxy (Nginx/Apache)          │
│  ├─ SSL Termination                     │
│  ├─ Static File Serving                 │
│  └─ Load Balancing (future)             │
├─────────────────────────────────────────┤
│  Application Layer                      │
│  ├─ Node.js Runtime                     │
│  ├─ TimeTracker Pro                     │
│  └─ Process Manager (PM2)               │
├─────────────────────────────────────────┤
│  Database Layer                         │
│  ├─ PostgreSQL 14+                      │
│  ├─ Redis (session store)               │
│  └─ Backup Storage                      │
└─────────────────────────────────────────┘
```

### High Availability Deployment
```
┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │────│   Load Balancer │
│    (Primary)    │    │   (Secondary)   │
└─────────────────┘    └─────────────────┘
         │                       │
         └───────────┬───────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───▼────┐    ┌──────▼──┐    ┌───────▼─┐
│App     │    │App      │    │App      │
│Server 1│    │Server 2 │    │Server 3 │
└────────┘    └─────────┘    └─────────┘
         │                │           │
         └────────────────┼───────────┘
                          │
              ┌───────────▼───────────┐
              │     Database Cluster  │
              │  ┌─────────────────┐  │
              │  │ PostgreSQL      │  │
              │  │ Primary/Replica │  │
              │  └─────────────────┘  │
              │  ┌─────────────────┐  │
              │  │ Redis Cluster   │  │
              │  └─────────────────┘  │
              └─────────────────────────┘
```

---

## Pre-Installation Setup

### 1. System Preparation
**Ubuntu/Debian Systems:**
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl wget git build-essential software-properties-common

# Configure firewall
sudo ufw enable
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 5432/tcp  # PostgreSQL (if external access needed)
```

**CentOS/RHEL Systems:**
```bash
# Update system packages
sudo dnf update -y

# Install essential packages
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y curl wget git

# Configure firewall
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=22/tcp
sudo firewall-cmd --reload
```

### 2. Database Installation

#### PostgreSQL Setup
**Installation:**
```bash
# Ubuntu/Debian
sudo apt install -y postgresql postgresql-contrib

# CentOS/RHEL
sudo dnf install -y postgresql postgresql-server postgresql-contrib
sudo postgresql-setup --initdb
```

**Configuration:**
```bash
# Switch to postgres user
sudo -u postgres psql

-- Create database and user
CREATE DATABASE timetracker_pro;
CREATE USER timetracker_user WITH ENCRYPTED PASSWORD 'secure_password_here';
GRANT ALL PRIVILEGES ON DATABASE timetracker_pro TO timetracker_user;
ALTER USER timetracker_user CREATEDB;
\q
```

**PostgreSQL Configuration Files:**
```bash
# Edit postgresql.conf
sudo nano /etc/postgresql/14/main/postgresql.conf

# Key settings for production:
listen_addresses = 'localhost'  # or specific IPs
max_connections = 100
shared_buffers = 256MB          # 25% of RAM
effective_cache_size = 1GB      # 75% of RAM
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100

# Edit pg_hba.conf for authentication
sudo nano /etc/postgresql/14/main/pg_hba.conf

# Add application access:
local   timetracker_pro    timetracker_user                    md5
host    timetracker_pro    timetracker_user    127.0.0.1/32    md5
```

#### Redis Installation (Optional for Session Store)
```bash
# Ubuntu/Debian
sudo apt install -y redis-server

# CentOS/RHEL
sudo dnf install -y redis
sudo systemctl enable redis
sudo systemctl start redis

# Configure Redis
sudo nano /etc/redis/redis.conf
# Uncomment and set:
# requireauth your_redis_password
# maxmemory 256mb
# maxmemory-policy allkeys-lru
```

### 3. Node.js Installation
**Using NodeSource Repository:**
```bash
# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version  # Should show v20.x.x
npm --version   # Should show 10.x.x
```

**Alternative - Using Node Version Manager (NVM):**
```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc

# Install and use Node.js 20
nvm install 20
nvm use 20
nvm alias default 20
```

---

## Application Deployment

### 1. Application Setup
**Extract and Prepare Application:**
```bash
# Create application directory
sudo mkdir -p /opt/timetracker-pro
sudo chown $USER:$USER /opt/timetracker-pro
cd /opt/timetracker-pro

# Extract your production package
tar -xzf /path/to/timetracker-pro-production-package.tar.gz
mv source-code/* .
rm -rf source-code

# Install dependencies
npm ci --only=production

# Install PM2 process manager globally
sudo npm install -g pm2
```

### 2. Environment Configuration
**Create Production Environment File:**
```bash
# Create .env file
nano .env

# Essential configuration:
NODE_ENV=production
PORT=3000

# Database Configuration
DATABASE_URL=postgresql://timetracker_user:secure_password_here@localhost:5432/timetracker_pro

# Session Configuration
SESSION_SECRET=your_very_secure_session_secret_here_minimum_32_chars
COOKIE_SECURE=true
COOKIE_HTTP_ONLY=true

# Application Configuration
APP_NAME=TimeTracker Pro
APP_URL=https://your-domain.com
TIMEZONE=America/New_York

# Security Configuration
BCRYPT_ROUNDS=12
MAX_LOGIN_ATTEMPTS=5
LOCKOUT_TIME=900000  # 15 minutes

# Optional: Redis Session Store
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=your_redis_password

# Optional: Email Configuration
SMTP_HOST=smtp.your-company.com
SMTP_PORT=587
SMTP_USER=noreply@your-company.com
SMTP_PASS=your_smtp_password

# Logging Configuration
LOG_LEVEL=info
LOG_FILE=/var/log/timetracker-pro/app.log

# Set secure permissions
chmod 600 .env
```

### 3. Database Migration
**Initialize Database Schema:**
```bash
# Run database migrations
npm run db:push

# Verify schema creation
npm run db:studio  # Optional: opens Drizzle Studio for verification
```

**Create Initial Admin User:**
```bash
# Connect to database
sudo -u postgres psql timetracker_pro

-- Create initial admin user
INSERT INTO users (id, email, role, created_at, updated_at) 
VALUES ('admin-user-1', 'admin@your-company.com', 'admin', NOW(), NOW());

-- Verify user creation
SELECT * FROM users WHERE role = 'admin';
\q
```

### 4. Build Application
**Production Build:**
```bash
# Build frontend assets
npm run build

# Verify build output
ls -la dist/  # Should contain built assets
```

---

## Process Management

### 1. PM2 Configuration
**Create PM2 Ecosystem File:**
```bash
nano ecosystem.config.js

module.exports = {
  apps: [{
    name: 'timetracker-pro',
    script: 'dist/index.js',
    instances: 'max',  // Use all CPU cores
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: '/var/log/timetracker-pro/err.log',
    out_file: '/var/log/timetracker-pro/out.log',
    log_file: '/var/log/timetracker-pro/combined.log',
    time: true,
    max_memory_restart: '1G',
    node_args: '--max-old-space-size=1024'
  }]
};
```

**Start Application with PM2:**
```bash
# Create log directory
sudo mkdir -p /var/log/timetracker-pro
sudo chown $USER:$USER /var/log/timetracker-pro

# Start application
pm2 start ecosystem.config.js --env production

# Setup PM2 startup script
pm2 startup
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u $USER --hp $HOME

# Save PM2 process list
pm2 save
```

### 2. Systemd Service (Alternative)
**Create Systemd Service File:**
```bash
sudo nano /etc/systemd/system/timetracker-pro.service

[Unit]
Description=TimeTracker Pro Application
After=network.target postgresql.service

[Service]
Type=simple
User=timetracker
WorkingDirectory=/opt/timetracker-pro
Environment=NODE_ENV=production
ExecStart=/usr/bin/node dist/index.js
Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=timetracker-pro

[Install]
WantedBy=multi-user.target
```

**Enable and Start Service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable timetracker-pro
sudo systemctl start timetracker-pro
sudo systemctl status timetracker-pro
```

---

## Reverse Proxy Configuration

### 1. Nginx Setup
**Install Nginx:**
```bash
# Ubuntu/Debian
sudo apt install -y nginx

# CentOS/RHEL
sudo dnf install -y nginx
```

**Nginx Configuration:**
```bash
sudo nano /etc/nginx/sites-available/timetracker-pro

server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    # SSL Configuration
    ssl_certificate /etc/ssl/certs/timetracker-pro.crt;
    ssl_certificate_key /etc/ssl/private/timetracker-pro.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;

    # Security Headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Static file serving (if needed)
    location /static/ {
        alias /opt/timetracker-pro/dist/public/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
    }
}
```

**Enable Site:**
```bash
sudo ln -s /etc/nginx/sites-available/timetracker-pro /etc/nginx/sites-enabled/
sudo nginx -t  # Test configuration
sudo systemctl reload nginx
```

### 2. SSL Certificate Setup
**Using Let's Encrypt (Recommended):**
```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Setup auto-renewal
sudo crontab -e
# Add line:
0 12 * * * /usr/bin/certbot renew --quiet
```

**Using Self-Signed Certificate (Development):**
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/timetracker-pro.key \
    -out /etc/ssl/certs/timetracker-pro.crt \
    -subj "/CN=your-domain.com"
```

---

## Security Hardening

### 1. System Security
**Firewall Configuration:**
```bash
# UFW (Ubuntu/Debian)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw enable

# Fail2Ban for intrusion prevention
sudo apt install -y fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Configure SSH security
sudo nano /etc/ssh/sshd_config
# Recommended settings:
# PermitRootLogin no
# PasswordAuthentication no
# PubkeyAuthentication yes
# AllowUsers your_username
```

### 2. Database Security
**PostgreSQL Security:**
```bash
# Create dedicated database user
sudo -u postgres createuser --no-createdb --no-createrole --no-superuser timetracker_app

# Set strong password
sudo -u postgres psql
ALTER USER timetracker_app WITH PASSWORD 'very_strong_password_here';
GRANT ALL PRIVILEGES ON DATABASE timetracker_pro TO timetracker_app;
\q

# Configure PostgreSQL for security
sudo nano /etc/postgresql/14/main/postgresql.conf
# ssl = on
# ssl_cert_file = '/etc/ssl/certs/server.crt'
# ssl_key_file = '/etc/ssl/private/server.key'
```

### 3. Application Security
**Security Configuration:**
```bash
# Set proper file permissions
sudo chown -R timetracker:timetracker /opt/timetracker-pro
sudo chmod -R 750 /opt/timetracker-pro
sudo chmod 600 /opt/timetracker-pro/.env

# Create dedicated user
sudo useradd --system --shell /bin/false --home /opt/timetracker-pro timetracker
```

---

## Monitoring and Logging

### 1. Application Monitoring
**PM2 Monitoring:**
```bash
# Monitor application
pm2 monit

# View logs
pm2 logs timetracker-pro

# Application metrics
pm2 show timetracker-pro
```

**Log Rotation:**
```bash
sudo nano /etc/logrotate.d/timetracker-pro

/var/log/timetracker-pro/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 timetracker timetracker
    postrotate
        pm2 reloadLogs
    endscript
}
```

### 2. System Monitoring
**Install Monitoring Tools:**
```bash
# System monitoring
sudo apt install -y htop iotop nethogs

# Optional: Install Prometheus Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xzf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
```

### 3. Database Monitoring
**PostgreSQL Monitoring:**
```bash
# Install pg_stat_statements
sudo -u postgres psql timetracker_pro
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
\q

# Monitor database performance
sudo -u postgres psql -c "SELECT query, calls, total_time, mean_time FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;"
```

---

## Backup and Recovery

### 1. Database Backup
**Automated Backup Script:**
```bash
sudo nano /opt/scripts/backup-database.sh

#!/bin/bash
BACKUP_DIR="/opt/backups/database"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="timetracker_pro"

mkdir -p $BACKUP_DIR

# Create backup
pg_dump -h localhost -U timetracker_user -d $DB_NAME | gzip > $BACKUP_DIR/timetracker_backup_$DATE.sql.gz

# Remove backups older than 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: timetracker_backup_$DATE.sql.gz"
```

**Schedule Daily Backups:**
```bash
sudo chmod +x /opt/scripts/backup-database.sh
sudo crontab -e
# Add line for daily backup at 2 AM:
0 2 * * * /opt/scripts/backup-database.sh
```

### 2. Application Backup
**Full Application Backup:**
```bash
sudo nano /opt/scripts/backup-application.sh

#!/bin/bash
BACKUP_DIR="/opt/backups/application"
DATE=$(date +%Y%m%d_%H%M%S)
APP_DIR="/opt/timetracker-pro"

mkdir -p $BACKUP_DIR

# Backup application files (excluding node_modules)
tar --exclude='node_modules' --exclude='dist' -czf $BACKUP_DIR/app_backup_$DATE.tar.gz -C /opt timetracker-pro

# Remove backups older than 7 days
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Application backup completed: app_backup_$DATE.tar.gz"
```

### 3. Recovery Procedures
**Database Recovery:**
```bash
# Stop application
pm2 stop timetracker-pro

# Restore database
gunzip -c /opt/backups/database/timetracker_backup_YYYYMMDD_HHMMSS.sql.gz | sudo -u postgres psql timetracker_pro

# Start application
pm2 start timetracker-pro
```

---

## Maintenance Procedures

### 1. Regular Maintenance Tasks
**Weekly Tasks:**
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Check disk space
df -h

# Review application logs
pm2 logs timetracker-pro --lines 100

# Database maintenance
sudo -u postgres psql timetracker_pro -c "VACUUM ANALYZE;"
```

**Monthly Tasks:**
```bash
# Security updates
sudo apt list --upgradable | grep security

# Certificate renewal check
sudo certbot certificates

# Backup verification
ls -la /opt/backups/database/ | tail -5
```

### 2. Performance Optimization
**Database Tuning:**
```bash
# Analyze database performance
sudo -u postgres psql timetracker_pro
SELECT schemaname,tablename,attname,n_distinct,correlation FROM pg_stats WHERE tablename = 'time_entries';

# Reindex tables monthly
REINDEX DATABASE timetracker_pro;
```

**Application Optimization:**
```bash
# Monitor PM2 processes
pm2 monit

# Restart application weekly
pm2 restart timetracker-pro

# Clear application logs
pm2 flush timetracker-pro
```

---

## Troubleshooting Guide

### Common Issues and Solutions

**1. Application Won't Start**
```bash
# Check application logs
pm2 logs timetracker-pro

# Check environment variables
pm2 env timetracker-pro

# Verify database connection
npm run db:check  # If available
```

**2. Database Connection Issues**
```bash
# Test database connection
sudo -u postgres psql timetracker_pro -c "SELECT version();"

# Check PostgreSQL status
sudo systemctl status postgresql

# Review PostgreSQL logs
sudo tail -f /var/log/postgresql/postgresql-14-main.log
```

**3. Nginx Issues**
```bash
# Test Nginx configuration
sudo nginx -t

# Check Nginx logs
sudo tail -f /var/log/nginx/error.log

# Verify upstream connection
curl -I http://localhost:3000
```

**4. SSL Certificate Issues**
```bash
# Check certificate status
sudo certbot certificates

# Test SSL configuration
openssl s_client -connect your-domain.com:443

# Renew certificate manually
sudo certbot renew --dry-run
```

---

## High Availability Setup (Optional)

### Load Balancer Configuration
**HAProxy Setup:**
```bash
# Install HAProxy
sudo apt install -y haproxy

# Configure HAProxy
sudo nano /etc/haproxy/haproxy.cfg

global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend timetracker_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/timetracker-pro.pem
    redirect scheme https if !{ ssl_fc }
    default_backend timetracker_backend

backend timetracker_backend
    balance roundrobin
    option httpchk GET /health
    server app1 10.0.1.10:3000 check
    server app2 10.0.1.11:3000 check
    server app3 10.0.1.12:3000 check
```

### Database Clustering
**PostgreSQL Primary/Replica Setup:**
```bash
# Primary server configuration
sudo nano /etc/postgresql/14/main/postgresql.conf
wal_level = replica
max_wal_senders = 3
wal_keep_segments = 64

# Replica server setup
sudo -u postgres pg_basebackup -h primary-server-ip -D /var/lib/postgresql/14/main -U replication -P -v -R -X stream -C -S replica1
```

This comprehensive guide provides everything needed to deploy TimeTracker Pro on-premises with enterprise-grade security, monitoring, and high availability options.