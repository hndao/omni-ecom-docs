# Production Deployment - Quick Start Guide

**Last Updated:** 2026-02-04  
**Estimated Time:** 2-3 hours  
**Difficulty:** Intermediate

This is a condensed version of the full deployment guide. For detailed explanations, see [PRODUCTION_DEPLOYMENT_GUIDE.md](PRODUCTION_DEPLOYMENT_GUIDE.md).

---

## Prerequisites

- Ubuntu 22.04 LTS server with root access
- Domain names configured (DNS A records)
- GitHub SSH access
- SMTP credentials
- Cloudflare R2 or AWS S3 credentials

---

## Quick Deployment Steps

### 1. Initial Server Setup (15 minutes)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y git curl wget vim htop ufw fail2ban unattended-upgrades certbot python3-certbot-nginx

# Configure firewall
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# Create deploy user
sudo adduser deploy
sudo usermod -aG sudo deploy
```

### 2. Install Docker (10 minutes)

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add deploy user to docker group
sudo usermod -aG docker deploy

# Enable Docker
sudo systemctl enable docker
sudo systemctl start docker

# Verify installation
docker --version
docker compose version
```

### 3. Clone Repository (5 minutes)

```bash
# Switch to deploy user
su - deploy

# Clone repository
mkdir -p /home/deploy/apps
cd /home/deploy/apps
git clone git@github.com:your-org/omni.git
cd omni
```

### 4. Configure Environment (20 minutes)

**Backend (.env):**

```bash
cd /home/deploy/apps/omni/omnistock
cp .env.prod .env
nano .env
```

Update these critical values:
- `APP_KEY=` (will generate later)
- `APP_URL=https://admin.yourdomain.com`
- `DB_HOST=postgres`
- `DB_DATABASE=omnistock_prod`
- `DB_USERNAME=omnistock_user`
- `DB_PASSWORD=STRONG_PASSWORD_HERE`
- `REDIS_HOST=redis`
- `MEMCACHED_HOST=memcached`
- `R2_ACCESS_KEY_ID=your-key`
- `R2_SECRET_ACCESS_KEY=your-secret`
- `R2_BUCKET=your-bucket`
- `R2_ENDPOINT=your-endpoint`
- `MAIL_HOST=smtp.example.com`
- `MAIL_USERNAME=your-username`
- `MAIL_PASSWORD=your-password`

**Frontend (.env.local):**

```bash
cd /home/deploy/apps/omni/omni-ecom-nextjs
nano .env.local
```

```env
NEXT_PUBLIC_API_URL=https://admin.yourdomain.com/api
NEXT_PUBLIC_API_KEY=ZEfUnq1GWDEFbu15cfpZjVGlqZuVA2Cv
NEXT_PUBLIC_CHANNEL_DOMAIN=yourdomain.com
NEXT_PUBLIC_APP_NAME=Omni E-Commerce
NEXT_PUBLIC_APP_URL=https://www.yourdomain.com
```

### 5. Generate APP_KEY (5 minutes)

```bash
cd /home/deploy/apps/omni

# Start database temporarily
docker compose up -d postgres redis memcached
sleep 15

# Generate key
docker compose run --rm backend php artisan key:generate --show

# Copy the output (e.g., base64:xxx...) and update omnistock/.env
nano omnistock/.env
# Set APP_KEY=base64:xxx...

# Stop temporary containers
docker compose down
```

### 6. Install Nginx and Configure SSL (20 minutes)

```bash
# Install Nginx
sudo apt install -y nginx

# Create backend config
sudo nano /etc/nginx/sites-available/admin.yourdomain.com
```

Paste this configuration (replace `yourdomain.com`):

```nginx
server {
    listen 80;
    server_name admin.yourdomain.com;
    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Create frontend config:**

```bash
sudo nano /etc/nginx/sites-available/www.yourdomain.com
```

```nginx
server {
    listen 80;
    server_name www.yourdomain.com yourdomain.com;
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

**Enable sites and get SSL:**

```bash
# Enable sites
sudo ln -s /etc/nginx/sites-available/admin.yourdomain.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/www.yourdomain.com /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default

# Test and reload
sudo nginx -t
sudo systemctl reload nginx

# Get SSL certificates
sudo certbot --nginx -d admin.yourdomain.com
sudo certbot --nginx -d www.yourdomain.com -d yourdomain.com
```

### 7. Deploy Application (30 minutes)

```bash
cd /home/deploy/apps/omni

# Build and start all services
docker compose build --no-cache
docker compose up -d

# Wait for services to start
sleep 30

# Check status
docker compose ps

# Run migrations
docker compose exec backend php artisan migrate --force

# Seed database
docker compose exec backend php artisan db:seed --force

# Cache configuration
docker compose exec backend php artisan config:cache
docker compose exec backend php artisan route:cache
docker compose exec backend php artisan view:cache
```

### 8. Verify Deployment (10 minutes)

```bash
# Check containers
docker compose ps

# Test backend
curl -I https://admin.yourdomain.com
curl https://admin.yourdomain.com/api/health

# Test frontend
curl -I https://www.yourdomain.com

# Check logs
docker compose logs --tail=50 backend
docker compose logs --tail=50 frontend

# Monitor resources
docker stats --no-stream
```

---

## Post-Deployment

### Set Up Backups

```bash
# Create backup directory
mkdir -p /home/deploy/backups

# Create database backup script
cat > /home/deploy/backups/backup-db.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/deploy/backups/database"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
docker exec omni-postgres pg_dump -U omnistock_user omnistock_prod | gzip > "${BACKUP_DIR}/backup_${TIMESTAMP}.sql.gz"
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
EOF

chmod +x /home/deploy/backups/backup-db.sh

# Add to crontab
crontab -e
# Add: 0 2 * * * /home/deploy/backups/backup-db.sh
```

### Monitor Containers

```bash
# Create monitoring script
cat > /home/deploy/monitor.sh << 'EOF'
#!/bin/bash
CONTAINERS=("omni-postgres" "omni-redis" "omni-backend" "omni-frontend" "omni-queue" "omni-scheduler")
for container in "${CONTAINERS[@]}"; do
    if ! docker ps | grep -q $container; then
        echo "$(date): $container is down!" >> /home/deploy/monitor.log
    fi
done
EOF

chmod +x /home/deploy/monitor.sh

# Add to crontab
crontab -e
# Add: */5 * * * * /home/deploy/monitor.sh
```

---

## Default Credentials

**System Administrator:**
- Email: `sysadmin@omnistock.local`
- Password: `password`

⚠️ **IMPORTANT:** Change this password immediately after first login!

```bash
docker compose exec backend php artisan tinker
# In tinker:
# $user = User::where('email', 'sysadmin@omnistock.local')->first();
# $user->password = Hash::make('your-new-secure-password');
# $user->save();
# exit
```

---

## Troubleshooting

**Containers won't start:**
```bash
docker compose logs
docker compose down
docker compose up -d
```

**Database connection failed:**
```bash
docker compose exec backend cat .env | grep DB_
docker compose logs postgres
```

**SSL issues:**
```bash
sudo certbot renew --force-renewal
sudo nginx -t
sudo systemctl reload nginx
```

---

## Next Steps

1. ✅ Change default admin password
2. ✅ Test all functionality
3. ✅ Set up monitoring alerts
4. ✅ Configure off-site backups
5. ✅ Review security settings
6. ✅ Update documentation with actual credentials (securely)

---

## Support

For detailed information, see:
- [PRODUCTION_DEPLOYMENT_GUIDE.md](PRODUCTION_DEPLOYMENT_GUIDE.md) - Full deployment guide
- [PRODUCTION_DEPLOYMENT_CHECKLIST.md](PRODUCTION_DEPLOYMENT_CHECKLIST.md) - Deployment checklist
- [AI_PROJECT_OVERVIEW.md](AI_PROJECT_OVERVIEW.md) - Project overview
- [AI_TECH_STACK.md](AI_TECH_STACK.md) - Technology stack details

---

**Deployment Complete!** 🎉

Your Omni E-Commerce platform is now running in production.

