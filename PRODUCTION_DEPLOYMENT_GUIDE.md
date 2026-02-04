# Production Deployment Guide - Ubuntu 22.04

**Last Updated:** 2026-02-04  
**Target OS:** Ubuntu 22.04 LTS  
**Docker Version:** 24.0+  
**Docker Compose Version:** 2.20+

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Server Setup](#server-setup)
3. [Docker Installation](#docker-installation)
4. [Application Deployment](#application-deployment)
5. [SSL/TLS Configuration](#ssltls-configuration)
6. [Environment Configuration](#environment-configuration)
7. [Database Setup](#database-setup)
8. [Monitoring & Logging](#monitoring--logging)
9. [Backup Strategy](#backup-strategy)
10. [Security Hardening](#security-hardening)
11. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Server Requirements

**Minimum Specifications:**
- **CPU:** 4 cores (8 cores recommended)
- **RAM:** 8 GB (16 GB recommended)
- **Storage:** 100 GB SSD (200 GB recommended)
- **OS:** Ubuntu 22.04 LTS (fresh installation)
- **Network:** Static IP address with open ports 80, 443

**Domain Requirements:**
- Backend/Admin: `admin.yourdomain.com`
- Frontend/Customer: `www.yourdomain.com` or `yourdomain.com`
- DNS A records pointing to server IP

### Required Access

- Root or sudo access to the server
- SSH key-based authentication (password auth disabled)
- GitHub repository access (SSH key added to GitHub)
- Domain registrar access for DNS configuration
- Email SMTP credentials (for notifications)
- Cloudflare R2 or AWS S3 credentials (for file storage)

---

## Server Setup

### 1. Initial Server Configuration

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Set timezone
sudo timedatectl set-timezone UTC

# Set hostname
sudo hostmod set-hostname omni-production

# Create deployment user
sudo adduser deploy
sudo usermod -aG sudo deploy
sudo usermod -aG docker deploy  # Will be created after Docker installation

# Configure SSH for deploy user
sudo mkdir -p /home/deploy/.ssh
sudo cp ~/.ssh/authorized_keys /home/deploy/.ssh/
sudo chown -R deploy:deploy /home/deploy/.ssh
sudo chmod 700 /home/deploy/.ssh
sudo chmod 600 /home/deploy/.ssh/authorized_keys
```

### 2. Install Essential Tools

```bash
# Install required packages
sudo apt install -y \
    git \
    curl \
    wget \
    vim \
    htop \
    ufw \
    fail2ban \
    unattended-upgrades \
    certbot \
    python3-certbot-nginx
```

### 3. Configure Firewall

```bash
# Enable UFW firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# Check firewall status
sudo ufw status verbose
```

---

## Docker Installation

### 1. Install Docker Engine

```bash
# Remove old Docker versions
sudo apt remove docker docker-engine docker.io containerd runc

# Install Docker dependencies
sudo apt update
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify Docker installation
docker --version
docker compose version
```

### 2. Configure Docker

```bash
# Add deploy user to docker group
sudo usermod -aG docker deploy

# Enable Docker to start on boot
sudo systemctl enable docker
sudo systemctl enable containerd

# Configure Docker daemon
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}
EOF

# Restart Docker
sudo systemctl restart docker

# Verify Docker is running
sudo systemctl status docker
```

### 3. Log out and log back in

```bash
# Log out and log back in as deploy user for docker group to take effect
exit
# SSH back in as deploy user
ssh deploy@your-server-ip
```

---

## Application Deployment

### 1. Clone Repository

```bash
# Switch to deploy user
su - deploy

# Create application directory
mkdir -p /home/deploy/apps
cd /home/deploy/apps

# Clone the repository (use SSH)
git clone git@github.com:your-org/omni.git
cd omni

# Checkout production branch (or main/master)
git checkout main
```

### 2. Create Production Environment Files

#### Backend Environment (.env)

```bash
# Copy production template
cd /home/deploy/apps/omni/omnistock
cp .env.prod .env

# Edit the .env file
nano .env
```

**Required Configuration:**

```env
APP_NAME=OmniStock
APP_ENV=production
APP_KEY=  # Will be generated later
APP_DEBUG=false
APP_URL=https://admin.yourdomain.com
APP_TIMEZONE=UTC

LOG_CHANNEL=stack
LOG_STACK=daily
LOG_LEVEL=warning

DB_CONNECTION=pgsql
DB_HOST=postgres
DB_PORT=5432
DB_DATABASE=omnistock_prod
DB_USERNAME=omnistock_user
DB_PASSWORD=STRONG_RANDOM_PASSWORD_HERE

SESSION_DRIVER=database
SESSION_LIFETIME=120

FILESYSTEM_DISK=r2
QUEUE_CONNECTION=redis

CACHE_STORE=redis
CACHE_PREFIX=omnistock_prod_cache

MEMCACHED_HOST=memcached

REDIS_CLIENT=predis
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your-smtp-username
MAIL_PASSWORD=your-smtp-password
MAIL_FROM_ADDRESS="noreply@yourdomain.com"
MAIL_FROM_NAME="${APP_NAME}"

# Cloudflare R2 Configuration
R2_ACCESS_KEY_ID=your-r2-access-key
R2_SECRET_ACCESS_KEY=your-r2-secret-key
R2_BUCKET=your-bucket-name
R2_ENDPOINT=https://your-account-id.r2.cloudflarestorage.com
R2_PUBLIC_URL=https://your-public-url.com
R2_REGION=auto

# Bugsnag (optional - for error tracking)
BUGSNAG_API_KEY=your-bugsnag-api-key
```

#### Frontend Environment (.env.local)

```bash
cd /home/deploy/apps/omni/omni-ecom-nextjs

# Create .env.local file
nano .env.local
```

**Required Configuration:**

```env
# API Configuration
NEXT_PUBLIC_API_URL=https://admin.yourdomain.com/api
NEXT_PUBLIC_API_KEY=ZEfUnq1GWDEFbu15cfpZjVGlqZuVA2Cv
NEXT_PUBLIC_CHANNEL_DOMAIN=yourdomain.com

# App Configuration
NEXT_PUBLIC_APP_NAME=Omni E-Commerce
NEXT_PUBLIC_APP_URL=https://www.yourdomain.com
```

### 3. Create Production Docker Compose File

```bash
cd /home/deploy/apps/omni

# Create production docker-compose override
nano docker-compose.prod.yml
```

**Production Docker Compose Configuration:**

```yaml
version: '3.8'

services:
  postgres:
    environment:
      POSTGRES_DB: omnistock_prod
      POSTGRES_USER: omnistock_user
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - /var/lib/docker/volumes/omni-postgres:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    restart: unless-stopped
    volumes:
      - /var/lib/docker/volumes/omni-redis:/data

  memcached:
    restart: unless-stopped

  backend:
    build:
      context: ./omnistock
      dockerfile: Dockerfile.prod
    environment:
      APP_ENV: production
      APP_DEBUG: "false"
    restart: unless-stopped
    volumes:
      - /var/lib/docker/volumes/omni-backend-storage:/var/www/html/storage
      - /var/lib/docker/volumes/omni-backend-logs:/var/www/html/storage/logs

  frontend:
    build:
      context: ./omni-ecom-nextjs
      dockerfile: Dockerfile.prod
    environment:
      NODE_ENV: production
      NEXT_PUBLIC_API_URL: https://admin.yourdomain.com/api
      NEXT_PUBLIC_API_KEY: ${NEXT_PUBLIC_API_KEY}
      NEXT_PUBLIC_CHANNEL_DOMAIN: yourdomain.com
    restart: unless-stopped

  queue:
    restart: unless-stopped

  scheduler:
    restart: unless-stopped
```

### 4. Create Production Dockerfiles

#### Backend Production Dockerfile

```bash
cd /home/deploy/apps/omni/omnistock
nano Dockerfile.prod
```

**Dockerfile.prod content:**

```dockerfile
# Multi-stage build for production
FROM php:8.4-fpm-alpine AS base

WORKDIR /var/www/html

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    wget \
    netcat-openbsd \
    libpng-dev \
    libjpeg-turbo-dev \
    freetype-dev \
    libzip-dev \
    zip \
    unzip \
    postgresql-dev \
    nodejs \
    npm \
    bash \
    nginx \
    supervisor

# Install PHP extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    pdo pdo_pgsql pgsql gd zip bcmath opcache

# Install Redis extension
RUN apk add --no-cache pcre-dev $PHPIZE_DEPS \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del pcre-dev $PHPIZE_DEPS

# Configure PHP for production
RUN echo "opcache.enable=1" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.memory_consumption=256" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.max_accelerated_files=20000" >> /usr/local/etc/php/conf.d/opcache.ini \
    && echo "opcache.validate_timestamps=0" >> /usr/local/etc/php/conf.d/opcache.ini

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Copy application files
COPY . /var/www/html

# Install dependencies
RUN composer install --no-interaction --no-dev --optimize-autoloader --no-scripts \
    && npm install \
    && npm run build

# Set permissions
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html/storage \
    && chmod -R 755 /var/www/html/bootstrap/cache

# Configure Nginx
COPY docker/nginx/default.conf /etc/nginx/http.d/default.conf

# Create startup script
COPY docker/start-prod.sh /usr/local/bin/start.sh
RUN chmod +x /usr/local/bin/start.sh

EXPOSE 8000

CMD ["/usr/local/bin/start.sh"]
```

#### Frontend Production Dockerfile

```bash
cd /home/deploy/apps/omni/omni-ecom-nextjs
nano Dockerfile.prod
```

**Dockerfile.prod content:**

```dockerfile
# Multi-stage build for production
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

# Build arguments for environment variables
ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_API_KEY
ARG NEXT_PUBLIC_CHANNEL_DOMAIN
ARG NEXT_PUBLIC_APP_NAME
ARG NEXT_PUBLIC_APP_URL

ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_KEY=$NEXT_PUBLIC_API_KEY
ENV NEXT_PUBLIC_CHANNEL_DOMAIN=$NEXT_PUBLIC_CHANNEL_DOMAIN
ENV NEXT_PUBLIC_APP_NAME=$NEXT_PUBLIC_APP_NAME
ENV NEXT_PUBLIC_APP_URL=$NEXT_PUBLIC_APP_URL
ENV NODE_ENV=production

RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
```

### 5. Create Supporting Configuration Files

#### Nginx Configuration for Backend

```bash
mkdir -p /home/deploy/apps/omni/docker/nginx
nano /home/deploy/apps/omni/docker/nginx/default.conf
```

**Nginx configuration:**

```nginx
server {
    listen 8000;
    server_name _;
    root /var/www/html/public;
    index index.php;

    client_max_body_size 100M;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

#### Backend Startup Script

```bash
nano /home/deploy/apps/omni/docker/start-prod.sh
```

**Startup script:**

```bash
#!/bin/bash
set -e

# Start PHP-FPM in background
php-fpm -D

# Wait for database
echo "Waiting for database..."
while ! nc -z postgres 5432; do sleep 1; done
echo "Database is ready!"

# Run package discovery
php artisan package:discover --ansi

# Run migrations
php artisan migrate --force

# Cache configuration
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Start Nginx in foreground
nginx -g 'daemon off;'
```

---

## SSL/TLS Configuration

### 1. Install Nginx Reverse Proxy

```bash
# Install Nginx on host (not in Docker)
sudo apt install -y nginx

# Remove default site
sudo rm /etc/nginx/sites-enabled/default
```

### 2. Configure Nginx for Backend (Admin)

```bash
sudo nano /etc/nginx/sites-available/admin.yourdomain.com
```

**Nginx configuration:**

```nginx
server {
    listen 80;
    server_name admin.yourdomain.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name admin.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/admin.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/admin.yourdomain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 100M;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3. Configure Nginx for Frontend (Customer)

```bash
sudo nano /etc/nginx/sites-available/www.yourdomain.com
```

**Nginx configuration:**

```nginx
server {
    listen 80;
    server_name www.yourdomain.com yourdomain.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://www.yourdomain.com$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name www.yourdomain.com yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/www.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.yourdomain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 100M;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 4. Enable Sites and Obtain SSL Certificates

```bash
# Enable sites
sudo ln -s /etc/nginx/sites-available/admin.yourdomain.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/www.yourdomain.com /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Obtain SSL certificates
sudo certbot --nginx -d admin.yourdomain.com
sudo certbot --nginx -d www.yourdomain.com -d yourdomain.com

# Set up auto-renewal
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

---

## Environment Configuration

### 1. Generate Application Key

```bash
cd /home/deploy/apps/omni

# Start only the backend container temporarily
docker compose up -d postgres redis memcached
sleep 10

# Generate APP_KEY
docker compose run --rm backend php artisan key:generate --show

# Copy the generated key and update omnistock/.env
# Example: base64:GENERATED_KEY_HERE
```

### 2. Create Environment Variables File

```bash
# Create .env file for docker-compose
nano /home/deploy/apps/omni/.env
```

**Content:**

```env
# PostgreSQL
POSTGRES_PASSWORD=your-strong-postgres-password

# Next.js
NEXT_PUBLIC_API_KEY=ZEfUnq1GWDEFbu15cfpZjVGlqZuVA2Cv
```

---

## Database Setup

### 1. Initialize Database

```bash
cd /home/deploy/apps/omni

# Start database services
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d postgres redis memcached

# Wait for database to be ready
sleep 15

# Run migrations
docker compose -f docker-compose.yml -f docker-compose.prod.yml exec backend php artisan migrate --force

# Seed database with initial data
docker compose -f docker-compose.yml -f docker-compose.prod.yml exec backend php artisan db:seed --force
```

### 2. Create Database Backup Script

```bash
sudo mkdir -p /home/deploy/backups
nano /home/deploy/backups/backup-database.sh
```

**Backup script:**

```bash
#!/bin/bash
BACKUP_DIR="/home/deploy/backups/database"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="omnistock_prod_${TIMESTAMP}.sql"

mkdir -p $BACKUP_DIR

docker exec omni-postgres pg_dump -U omnistock_user omnistock_prod > "${BACKUP_DIR}/${BACKUP_FILE}"

# Compress backup
gzip "${BACKUP_DIR}/${BACKUP_FILE}"

# Keep only last 30 days of backups
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: ${BACKUP_FILE}.gz"
```

```bash
chmod +x /home/deploy/backups/backup-database.sh

# Add to crontab (daily at 2 AM)
crontab -e
# Add this line:
# 0 2 * * * /home/deploy/backups/backup-database.sh >> /home/deploy/backups/backup.log 2>&1
```

---

## Monitoring & Logging

### 1. Set Up Log Rotation

```bash
sudo nano /etc/logrotate.d/omni-docker
```

**Content:**

```
/var/lib/docker/containers/*/*.log {
    rotate 7
    daily
    compress
    missingok
    delaycompress
    copytruncate
}
```

### 2. Monitor Docker Containers

```bash
# Create monitoring script
nano /home/deploy/monitor-containers.sh
```

**Monitoring script:**

```bash
#!/bin/bash

# Check if all containers are running
CONTAINERS=("omni-postgres" "omni-redis" "omni-memcached" "omni-backend" "omni-frontend" "omni-queue" "omni-scheduler")

for container in "${CONTAINERS[@]}"; do
    if ! docker ps | grep -q $container; then
        echo "$(date): Container $container is not running!" | tee -a /home/deploy/monitor.log
        # Optionally send alert email
        # echo "Container $container is down" | mail -s "Docker Alert" admin@yourdomain.com
    fi
done
```

```bash
chmod +x /home/deploy/monitor-containers.sh

# Add to crontab (every 5 minutes)
crontab -e
# Add this line:
# */5 * * * * /home/deploy/monitor-containers.sh
```

### 3. View Logs

```bash
# View all container logs
docker compose logs -f

# View specific service logs
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f queue

# View Laravel logs
docker compose exec backend tail -f storage/logs/laravel-$(date +%Y-%m-%d).log

# View Nginx access logs
sudo tail -f /var/log/nginx/access.log

# View Nginx error logs
sudo tail -f /var/log/nginx/error.log
```

---

## Backup Strategy

### 1. Database Backups

Already configured in [Database Setup](#database-setup) section.

### 2. Application Files Backup

```bash
nano /home/deploy/backups/backup-files.sh
```

**Backup script:**

```bash
#!/bin/bash
BACKUP_DIR="/home/deploy/backups/files"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
APP_DIR="/home/deploy/apps/omni"

mkdir -p $BACKUP_DIR

# Backup storage volumes
tar -czf "${BACKUP_DIR}/storage_${TIMESTAMP}.tar.gz" \
    /var/lib/docker/volumes/omni-backend-storage \
    /var/lib/docker/volumes/omni-backend-logs

# Keep only last 7 days of file backups
find $BACKUP_DIR -name "storage_*.tar.gz" -mtime +7 -delete

echo "File backup completed: storage_${TIMESTAMP}.tar.gz"
```

```bash
chmod +x /home/deploy/backups/backup-files.sh

# Add to crontab (weekly on Sunday at 3 AM)
crontab -e
# Add this line:
# 0 3 * * 0 /home/deploy/backups/backup-files.sh >> /home/deploy/backups/backup.log 2>&1
```

### 3. Off-site Backup (Optional)

```bash
# Install rclone for cloud backup
curl https://rclone.org/install.sh | sudo bash

# Configure rclone (follow prompts)
rclone config

# Create backup sync script
nano /home/deploy/backups/sync-to-cloud.sh
```

**Cloud sync script:**

```bash
#!/bin/bash
# Sync backups to cloud storage (e.g., AWS S3, Google Drive, Dropbox)
rclone sync /home/deploy/backups remote:omni-backups --exclude "*.log"
```

---

## Security Hardening

### 1. Configure Fail2Ban

```bash
# Create Nginx jail
sudo nano /etc/fail2ban/jail.d/nginx.conf
```

**Content:**

```ini
[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log

[nginx-noscript]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 6

[nginx-badbots]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 2

[nginx-noproxy]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 2
```

```bash
# Restart Fail2Ban
sudo systemctl restart fail2ban
sudo fail2ban-client status
```

### 2. Secure SSH

```bash
sudo nano /etc/ssh/sshd_config
```

**Update these settings:**

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
```

```bash
# Restart SSH
sudo systemctl restart sshd
```

### 3. Enable Automatic Security Updates

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

### 4. Set Up Docker Security

```bash
# Limit Docker daemon exposure
sudo nano /etc/docker/daemon.json
```

**Add:**

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "userns-remap": "default",
  "no-new-privileges": true
}
```

```bash
sudo systemctl restart docker
```

---

## Complete Deployment Steps

### Step-by-Step Production Deployment

```bash
# 1. Navigate to application directory
cd /home/deploy/apps/omni

# 2. Pull latest code
git pull origin main

# 3. Build Docker images
docker compose -f docker-compose.yml -f docker-compose.prod.yml build --no-cache

# 4. Stop existing containers (if any)
docker compose -f docker-compose.yml -f docker-compose.prod.yml down

# 5. Start all services
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 6. Wait for services to be healthy
sleep 30

# 7. Check container status
docker compose ps

# 8. Run migrations
docker compose exec backend php artisan migrate --force

# 9. Clear and cache configuration
docker compose exec backend php artisan config:cache
docker compose exec backend php artisan route:cache
docker compose exec backend php artisan view:cache

# 10. Verify services are running
curl -I http://localhost:8000/up
curl -I http://localhost:3000

# 11. Check logs for errors
docker compose logs --tail=50 backend
docker compose logs --tail=50 frontend

# 12. Test SSL endpoints
curl -I https://admin.yourdomain.com
curl -I https://www.yourdomain.com
```

### Post-Deployment Verification

```bash
# 1. Test backend API
curl https://admin.yourdomain.com/api/health

# 2. Test frontend
curl https://www.yourdomain.com

# 3. Test database connection
docker compose exec backend php artisan tinker
# In tinker:
# DB::connection()->getPdo();
# exit

# 4. Test queue worker
docker compose exec backend php artisan queue:work --once

# 5. Test scheduler
docker compose exec backend php artisan schedule:run

# 6. Monitor resource usage
docker stats --no-stream
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Container Won't Start

```bash
# Check container logs
docker compose logs backend

# Check Docker daemon logs
sudo journalctl -u docker -n 100

# Restart Docker service
sudo systemctl restart docker
```

#### 2. Database Connection Failed

```bash
# Check if PostgreSQL is running
docker compose ps postgres

# Check PostgreSQL logs
docker compose logs postgres

# Verify database credentials in .env
docker compose exec backend cat .env | grep DB_

# Test database connection
docker compose exec postgres psql -U omnistock_user -d omnistock_prod -c "SELECT 1;"
```

#### 3. Permission Denied Errors

```bash
# Fix storage permissions
docker compose exec backend chown -R www-data:www-data /var/www/html/storage
docker compose exec backend chmod -R 755 /var/www/html/storage
```

#### 4. SSL Certificate Issues

```bash
# Renew certificates manually
sudo certbot renew --force-renewal

# Check certificate expiry
sudo certbot certificates

# Test SSL configuration
sudo nginx -t
```

#### 5. High Memory Usage

```bash
# Check container resource usage
docker stats

# Restart specific service
docker compose restart backend

# Clear application cache
docker compose exec backend php artisan cache:clear
docker compose exec backend php artisan config:clear
```

#### 6. Queue Jobs Not Processing

```bash
# Check queue worker status
docker compose logs queue

# Restart queue worker
docker compose restart queue

# Check failed jobs
docker compose exec backend php artisan queue:failed

# Retry failed jobs
docker compose exec backend php artisan queue:retry all
```

#### 7. Frontend Build Errors

```bash
# Rebuild frontend
docker compose build --no-cache frontend

# Check Node.js version
docker compose exec frontend node --version

# Clear Next.js cache
docker compose exec frontend rm -rf .next
docker compose restart frontend
```

---

## Maintenance Commands

### Regular Maintenance Tasks

```bash
# Update application code
cd /home/deploy/apps/omni
git pull origin main
docker compose -f docker-compose.yml -f docker-compose.prod.yml build
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Clear all caches
docker compose exec backend php artisan cache:clear
docker compose exec backend php artisan config:clear
docker compose exec backend php artisan route:clear
docker compose exec backend php artisan view:clear

# Optimize application
docker compose exec backend php artisan config:cache
docker compose exec backend php artisan route:cache
docker compose exec backend php artisan view:cache

# Clean up Docker
docker system prune -a --volumes -f

# Update system packages
sudo apt update && sudo apt upgrade -y

# Restart all services
docker compose -f docker-compose.yml -f docker-compose.prod.yml restart
```

---

## Performance Optimization

### 1. Enable OPcache

Already configured in Dockerfile.prod

### 2. Configure Redis Persistence

```bash
# Edit Redis configuration
nano /home/deploy/apps/omni/docker/redis/redis.conf
```

**Add:**

```
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec
```

### 3. Database Query Optimization

```bash
# Enable query logging temporarily
docker compose exec backend php artisan db:monitor

# Analyze slow queries
docker compose exec postgres psql -U omnistock_user -d omnistock_prod
# In psql:
# SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
```

---

## Support and Resources

### Useful Commands Reference

```bash
# View all running containers
docker compose ps

# View container resource usage
docker stats

# Access container shell
docker compose exec backend sh
docker compose exec frontend sh

# View real-time logs
docker compose logs -f

# Restart specific service
docker compose restart backend

# Stop all services
docker compose down

# Start all services
docker compose up -d

# Rebuild and restart
docker compose up -d --build
```

### Important File Locations

- **Application:** `/home/deploy/apps/omni`
- **Backend .env:** `/home/deploy/apps/omni/omnistock/.env`
- **Frontend .env:** `/home/deploy/apps/omni/omni-ecom-nextjs/.env.local`
- **Nginx configs:** `/etc/nginx/sites-available/`
- **SSL certificates:** `/etc/letsencrypt/live/`
- **Docker volumes:** `/var/lib/docker/volumes/`
- **Backups:** `/home/deploy/backups/`
- **Logs:** `/var/log/nginx/` and Docker container logs

---

## Conclusion

This guide provides a comprehensive production deployment setup for the Omni E-Commerce platform on Ubuntu 22.04. Follow each section carefully and customize the configuration values according to your specific requirements.

**Important Reminders:**
- Always use strong, unique passwords
- Keep SSL certificates up to date
- Regularly backup database and files
- Monitor application logs and performance
- Keep Docker images and system packages updated
- Test deployments in staging environment first

For additional support, refer to the project documentation in the `omni-ecom-docs` repository.

---

**Document Version:** 1.0
**Last Updated:** 2026-02-04
**Maintained By:** DevOps Team


