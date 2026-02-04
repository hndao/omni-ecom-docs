# Production Deployment Checklist

**Last Updated:** 2026-02-04  
**Target:** Ubuntu 22.04 LTS

Use this checklist to ensure all steps are completed during production deployment.

---

## Pre-Deployment Checklist

### Server Preparation
- [ ] Ubuntu 22.04 LTS installed and updated
- [ ] Static IP address configured
- [ ] DNS A records created and propagated
  - [ ] `admin.yourdomain.com` → Server IP
  - [ ] `www.yourdomain.com` → Server IP
  - [ ] `yourdomain.com` → Server IP
- [ ] SSH key-based authentication configured
- [ ] Root/sudo access verified
- [ ] Firewall (UFW) configured (ports 22, 80, 443)
- [ ] Fail2Ban installed and configured
- [ ] Deploy user created with sudo access

### Required Credentials
- [ ] GitHub SSH key added to repository
- [ ] SMTP credentials obtained
- [ ] Cloudflare R2 / AWS S3 credentials obtained
- [ ] Strong database passwords generated
- [ ] SSL certificate email configured
- [ ] Bugsnag API key (optional)

### Docker Installation
- [ ] Docker Engine installed (24.0+)
- [ ] Docker Compose plugin installed (2.20+)
- [ ] Deploy user added to docker group
- [ ] Docker daemon configured
- [ ] Docker service enabled on boot

---

## Deployment Checklist

### 1. Application Setup
- [ ] Repository cloned to `/home/deploy/apps/omni`
- [ ] Production branch checked out
- [ ] Backend `.env` file created from `.env.prod`
- [ ] Frontend `.env.local` file created
- [ ] All environment variables configured
- [ ] APP_KEY generated for Laravel
- [ ] Strong passwords set for all services

### 2. Docker Configuration
- [ ] `docker-compose.prod.yml` created
- [ ] `Dockerfile.prod` created for backend
- [ ] `Dockerfile.prod` created for frontend
- [ ] Nginx configuration created for backend
- [ ] Backend startup script created
- [ ] Docker volumes directories created

### 3. SSL/TLS Setup
- [ ] Nginx installed on host
- [ ] Backend Nginx config created (`/etc/nginx/sites-available/admin.yourdomain.com`)
- [ ] Frontend Nginx config created (`/etc/nginx/sites-available/www.yourdomain.com`)
- [ ] Sites enabled in `/etc/nginx/sites-enabled/`
- [ ] Nginx configuration tested (`nginx -t`)
- [ ] SSL certificates obtained via Certbot
- [ ] Certificate auto-renewal configured
- [ ] HTTPS redirects working

### 4. Database Setup
- [ ] PostgreSQL container started
- [ ] Database migrations run successfully
- [ ] Database seeded with initial data
- [ ] System admin account created
- [ ] Database backup script created
- [ ] Backup cron job configured

### 5. Application Deployment
- [ ] All Docker images built successfully
- [ ] All containers started
- [ ] All containers healthy
- [ ] Backend accessible via HTTPS
- [ ] Frontend accessible via HTTPS
- [ ] API endpoints responding correctly
- [ ] Queue worker processing jobs
- [ ] Scheduler running tasks

### 6. Security Hardening
- [ ] SSH password authentication disabled
- [ ] Root login disabled
- [ ] Fail2Ban configured for Nginx
- [ ] Automatic security updates enabled
- [ ] Docker security settings applied
- [ ] File permissions verified
- [ ] Sensitive files protected

### 7. Monitoring & Logging
- [ ] Log rotation configured
- [ ] Container monitoring script created
- [ ] Monitoring cron job configured
- [ ] Application logs accessible
- [ ] Nginx logs accessible
- [ ] Error tracking configured (Bugsnag)

### 8. Backup Strategy
- [ ] Database backup script created and tested
- [ ] File backup script created and tested
- [ ] Backup cron jobs configured
- [ ] Backup retention policy set
- [ ] Off-site backup configured (optional)
- [ ] Restore procedure tested

---

## Post-Deployment Verification

### Functional Testing
- [ ] Backend login works (`https://admin.yourdomain.com`)
- [ ] Frontend loads correctly (`https://www.yourdomain.com`)
- [ ] API endpoints respond correctly
- [ ] Database queries execute successfully
- [ ] File uploads work (Cloudflare R2/S3)
- [ ] Email sending works (SMTP)
- [ ] Queue jobs process correctly
- [ ] Scheduled tasks run on time
- [ ] Session management works
- [ ] Cache system functioning

### Performance Testing
- [ ] Page load times acceptable (< 3 seconds)
- [ ] API response times acceptable (< 500ms)
- [ ] Database queries optimized
- [ ] OPcache enabled and working
- [ ] Redis cache working
- [ ] Resource usage within limits
- [ ] No memory leaks detected

### Security Testing
- [ ] HTTPS enforced on all pages
- [ ] SSL certificate valid (A+ rating)
- [ ] Security headers configured
- [ ] CORS configured correctly
- [ ] Rate limiting working
- [ ] SQL injection protection verified
- [ ] XSS protection verified
- [ ] CSRF protection verified

---

## Maintenance Schedule

### Daily
- [ ] Check container status
- [ ] Review error logs
- [ ] Monitor disk space
- [ ] Check backup completion

### Weekly
- [ ] Review application logs
- [ ] Check SSL certificate expiry
- [ ] Review failed queue jobs
- [ ] Test backup restoration

### Monthly
- [ ] Update system packages
- [ ] Update Docker images
- [ ] Review security logs
- [ ] Performance optimization review
- [ ] Database optimization

---

## Emergency Contacts

| Role | Name | Contact |
|------|------|---------|
| DevOps Lead | | |
| Backend Developer | | |
| Frontend Developer | | |
| Database Admin | | |
| System Admin | | |

---

## Rollback Procedure

If deployment fails:

1. **Stop new containers:**
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.prod.yml down
   ```

2. **Restore database backup:**
   ```bash
   gunzip /home/deploy/backups/database/latest_backup.sql.gz
   docker exec -i omni-postgres psql -U omnistock_user omnistock_prod < latest_backup.sql
   ```

3. **Revert to previous code:**
   ```bash
   git checkout <previous-commit-hash>
   ```

4. **Rebuild and restart:**
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
   ```

---

## Sign-Off

| Task | Completed By | Date | Signature |
|------|--------------|------|-----------|
| Server Setup | | | |
| Docker Installation | | | |
| Application Deployment | | | |
| SSL Configuration | | | |
| Security Hardening | | | |
| Testing & Verification | | | |
| Documentation Updated | | | |

---

**Deployment Date:** _______________  
**Deployed By:** _______________  
**Verified By:** _______________


