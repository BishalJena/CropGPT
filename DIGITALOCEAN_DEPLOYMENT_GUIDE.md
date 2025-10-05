# 🌊 Complete DigitalOcean Deployment Guide for CropGPT

Deploy your full-stack CropGPT application on DigitalOcean using App Platform or Droplets.

## 🎯 DigitalOcean Deployment Options

You have **TWO excellent options**:

### Option 1: App Platform (Recommended - Easiest) 🌟
- ✅ Like Render/Railway (managed PaaS)
- ✅ Auto-deploy from GitHub
- ✅ Easy setup (15-20 minutes)
- ✅ Auto-scaling and monitoring
- 💰 **$12-18/month**

### Option 2: Droplet + Docker (Advanced)
- ✅ Full control (VPS)
- ✅ Use your existing Docker setup
- ✅ More configuration options
- ✅ Best for learning Docker
- 💰 **$12-24/month**

**This guide covers BOTH methods!**

---

## 📋 Prerequisites

1. **GitHub Account** - Repo pushed: ✅ https://github.com/BishalJena/CropGPT
2. **DigitalOcean Account** - Sign up at: https://www.digitalocean.com
3. **API Keys** - Cerebras, OpenRouter, Deepgram, etc.
4. **MongoDB & Redis** - We'll set these up on DO or use managed services

---

# Method 1: App Platform Deployment (Recommended)

## Part 1: Create DigitalOcean Account

1. **Go to**: https://www.digitalocean.com
2. **Sign up** with email or GitHub
3. **Verify email**
4. **Add payment method** (get $200 credit for 60 days!)

---

## Part 2: Create Managed Databases

### Step 1: Create MongoDB Database

1. **Go to**: Databases → Create Database Cluster
2. **Configuration**:
   ```
   Database Engine: MongoDB
   Version: 6.0 or latest
   Plan: Basic ($15/month)
   Node: 1 vCPU, 1GB RAM
   Datacenter: New York (or closest)
   Cluster Name: cropgpt-mongodb
   ```
3. **Create Database Cluster** (takes 5 minutes)
4. **Get Connection Details**:
   - Click on database → Connection Details
   - Copy **Connection String**:
   ```
   mongodb://doadmin:<password>@cropgpt-mongodb-do-user-xxxxx.b.db.ondigitalocean.com:27017/admin?authSource=admin&tls=true
   ```
5. **Create Database**:
   - Click "Users & Databases" tab
   - Add Database: `farmchat`
   - Add User: `cropgpt` (optional)

### Step 2: Create Redis Database

1. **Go to**: Databases → Create Database Cluster
2. **Configuration**:
   ```
   Database Engine: Redis
   Version: 7.x or latest
   Plan: Basic ($15/month)
   Node: 1 vCPU, 1GB RAM
   Datacenter: Same as MongoDB
   Cluster Name: cropgpt-redis
   ```
3. **Create Database Cluster** (takes 3 minutes)
4. **Get Connection Details**:
   - Click on database → Connection Details
   - Copy **Connection String**:
   ```
   rediss://default:<password>@cropgpt-redis-do-user-xxxxx.b.db.ondigitalocean.com:25061
   ```

---

## Part 3: Deploy Backend with App Platform

### Step 1: Create App

1. **Go to**: Apps → Create App
2. **Choose Source**: GitHub
3. **Authorize** DigitalOcean to access GitHub
4. **Select Repository**: `BishalJena/CropGPT`
5. **Select Branch**: `main`
6. **Autodeploy**: ✅ Enable

### Step 2: Configure Backend Service

1. **Edit** the detected service or **Add Component** → Web Service

2. **Source Settings**:
   ```
   Type: Web Service
   Name: cropgpt-backend
   Branch: main
   Source Directory: /backend
   ```

3. **Build Settings**:
   ```
   Build Command: pip install -r requirements.txt
   ```
   
   Or leave empty (DigitalOcean auto-detects)

4. **Run Settings**:
   ```
   Run Command: uvicorn server:app --host 0.0.0.0 --port 8080
   HTTP Port: 8080
   HTTP Request Routes: /
   ```

5. **Health Check**:
   ```
   HTTP Path: /api/health
   ```

6. **Instance Size**:
   ```
   Basic: $12/month (1 vCPU, 1GB RAM)
   Or Professional: $24/month (2 vCPU, 2GB RAM)
   ```

7. **Environment Variables** - Click "Edit" next to the component:

   ```env
   # Database (from DigitalOcean Managed Database)
   MONGO_URL=${cropgpt-mongodb.DATABASE_URL}
   DB_NAME=farmchat
   
   # Redis (from DigitalOcean Managed Database)
   REDIS_URL=${cropgpt-redis.DATABASE_URL}
   
   # Authentication
   JWT_SECRET=<generate-32-char-secret>
   JWT_EXPIRATION_HOURS=24
   
   # AI Service API Keys
   CEREBRAS_API_KEY=<your-cerebras-key>
   OPENROUTER_API_KEY=<your-openrouter-key>
   DEEPGRAM_API_KEY=<your-deepgram-key>
   
   # External APIs
   EXA_API_KEY=<your-exa-key>
   DATAGOVIN_API_KEY=<your-datagovin-key>
   
   # MCP Gateway
   MCP_GATEWAY_URL=http://165.232.190.215:8811
   
   # CORS (update after frontend deployment)
   CORS_ORIGINS=*
   
   # Server Configuration
   ENVIRONMENT=production
   LOG_LEVEL=INFO
   PORT=8080
   ```

   **Generate JWT_SECRET**:
   ```bash
   python -c "import secrets; print(secrets.token_urlsafe(32))"
   ```

8. **Link Databases** (Alternative to manual connection strings):
   - Click "Add Resource"
   - Select your MongoDB cluster
   - Select your Redis cluster
   - DigitalOcean auto-injects connection strings

### Step 3: Configure Frontend Service

1. **Add Component** → Static Site

2. **Source Settings**:
   ```
   Type: Static Site
   Name: cropgpt-frontend
   Branch: main
   Source Directory: /frontend
   ```

3. **Build Settings**:
   ```
   Build Command: npm install --legacy-peer-deps && npm run build
   Output Directory: build
   ```

4. **Environment Variables**:
   ```env
   REACT_APP_BACKEND_URL=https://cropgpt-backend-xxxxx.ondigitalocean.app
   REACT_APP_VERSION=1.0.0
   REACT_APP_ENVIRONMENT=production
   REACT_APP_ENABLE_VOICE=true
   REACT_APP_ENABLE_WORKFLOWS=true
   REACT_APP_ENABLE_MARKETPLACE=true
   REACT_APP_ENABLE_SCHEMES=true
   ```

5. **Routes** - Click "Add Route":
   ```
   Path: /*
   Component: cropgpt-frontend
   ```

### Step 4: Review and Deploy

1. **Review** all settings
2. **App Name**: `cropgpt`
3. **Region**: New York (or same as databases)
4. **Click**: "Create Resources"
5. **Wait**: 10-15 minutes for first deploy

### Step 5: Get URLs and Update CORS

1. **Backend URL**: `https://cropgpt-backend-xxxxx.ondigitalocean.app`
2. **Frontend URL**: `https://cropgpt-xxxxx.ondigitalocean.app`

3. **Update Backend CORS**:
   - Go to backend component
   - Settings → Environment Variables
   - Update `CORS_ORIGINS`:
   ```
   CORS_ORIGINS=https://cropgpt-xxxxx.ondigitalocean.app,https://*.ondigitalocean.app
   ```

4. **Update Frontend Backend URL**:
   - Go to frontend component
   - Settings → Environment Variables
   - Update `REACT_APP_BACKEND_URL`:
   ```
   REACT_APP_BACKEND_URL=https://cropgpt-backend-xxxxx.ondigitalocean.app
   ```

5. **Redeploy** both components

---

# Method 2: Droplet + Docker Deployment (Advanced)

## Part 1: Create Droplet

### Step 1: Create New Droplet

1. **Go to**: Droplets → Create Droplet
2. **Choose Image**:
   ```
   Distribution: Ubuntu 22.04 LTS
   ```
3. **Choose Size**:
   ```
   Basic: $12/month (2 vCPU, 2GB RAM, 50GB SSD)
   Or Regular: $24/month (2 vCPU, 4GB RAM, 80GB SSD)
   ```
4. **Choose Datacenter**: New York or closest to you
5. **Authentication**:
   - ✅ SSH Keys (recommended - add your SSH key)
   - Or: Password (create strong password)
6. **Hostname**: `cropgpt-server`
7. **Enable**:
   - ✅ Monitoring
   - ✅ Backups (+20% cost, recommended)
8. **Create Droplet**

### Step 2: Initial Server Setup

1. **SSH into your droplet**:
   ```bash
   ssh root@your_droplet_ip
   ```

2. **Update system**:
   ```bash
   apt update && apt upgrade -y
   ```

3. **Install Docker**:
   ```bash
   # Install Docker
   curl -fsSL https://get.docker.com -o get-docker.sh
   sh get-docker.sh
   
   # Start Docker
   systemctl start docker
   systemctl enable docker
   
   # Verify installation
   docker --version
   docker compose version
   ```

4. **Install additional tools**:
   ```bash
   apt install -y git nginx certbot python3-certbot-nginx
   ```

5. **Create application user** (security best practice):
   ```bash
   adduser cropgpt
   usermod -aG docker cropgpt
   usermod -aG sudo cropgpt
   ```

---

## Part 2: Deploy Application with Docker

### Step 1: Clone Repository

```bash
# Switch to app user
su - cropgpt

# Clone repository
git clone https://github.com/BishalJena/CropGPT.git
cd CropGPT
```

### Step 2: Configure Environment Variables

```bash
# Create environment file
nano .env
```

**Add these variables**:
```env
# MongoDB (use DigitalOcean Managed DB or Docker)
MONGO_URL=mongodb://admin:password@mongodb:27017/farmchat?authSource=admin
MONGO_ROOT_PASSWORD=your-secure-mongo-password
DB_NAME=farmchat

# Redis (use DigitalOcean Managed DB or Docker)
REDIS_URL=redis://:your-redis-password@redis:6379/0
REDIS_PASSWORD=your-secure-redis-password

# Authentication
JWT_SECRET=your-32-char-random-secret
JWT_EXPIRATION_HOURS=24

# AI Service API Keys
CEREBRAS_API_KEY=your-cerebras-key
OPENROUTER_API_KEY=your-openrouter-key
DEEPGRAM_API_KEY=your-deepgram-key

# External APIs
EXA_API_KEY=your-exa-key
DATAGOVIN_API_KEY=your-datagovin-key

# MCP Gateway
MCP_GATEWAY_URL=http://mcp-server:10000

# CORS (update with your domain)
CORS_ORIGINS=https://your-domain.com,http://your_droplet_ip

# Server Configuration
ENVIRONMENT=production
LOG_LEVEL=INFO
APP_VERSION=1.0.0

# Monitoring
GRAFANA_USER=admin
GRAFANA_PASSWORD=your-grafana-password
```

**Save and exit**: `Ctrl+X`, `Y`, `Enter`

### Step 3: Deploy with Docker Compose

```bash
# Build and start all services
docker compose -f docker-compose.production.yml up -d

# View logs
docker compose -f docker-compose.production.yml logs -f

# Check running containers
docker ps

# You should see:
# - MongoDB
# - Redis
# - Backend (FastAPI)
# - Frontend (Nginx)
# - MCP Server
# - Prometheus
# - Grafana
```

### Step 4: Verify Deployment

```bash
# Test backend
curl http://localhost:8000/api/health

# Test frontend
curl http://localhost:3000

# Test monitoring
curl http://localhost:9090  # Prometheus
curl http://localhost:3001  # Grafana
```

---

## Part 3: Configure Nginx Reverse Proxy

### Step 1: Configure Nginx

```bash
# Create Nginx configuration
sudo nano /etc/nginx/sites-available/cropgpt
```

**Add this configuration**:
```nginx
# Backend API
server {
    listen 80;
    server_name api.your-domain.com;  # Or use your droplet IP
    
    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Frontend
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;  # Or your droplet IP
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Save and exit**: `Ctrl+X`, `Y`, `Enter`

### Step 2: Enable Configuration

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/cropgpt /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

### Step 3: Configure SSL with Let's Encrypt

```bash
# Get SSL certificates (replace with your actual domain)
sudo certbot --nginx -d your-domain.com -d www.your-domain.com -d api.your-domain.com

# Follow prompts:
# - Enter email
# - Agree to terms
# - Redirect HTTP to HTTPS: Yes

# Certbot auto-configures Nginx for HTTPS
```

### Step 4: Set Up Auto-Renewal

```bash
# Test renewal
sudo certbot renew --dry-run

# Add cron job (runs daily)
sudo crontab -e

# Add this line:
0 3 * * * certbot renew --quiet && systemctl reload nginx
```

---

## Part 4: Configure Firewall

```bash
# Configure UFW firewall
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable

# Verify firewall
sudo ufw status
```

---

## 🎯 Testing Your Deployment

### Test App Platform Deployment

```bash
# Test backend
curl https://cropgpt-backend-xxxxx.ondigitalocean.app/api/health

# Test frontend
https://cropgpt-xxxxx.ondigitalocean.app

# Test authentication
curl -X POST https://cropgpt-backend-xxxxx.ondigitalocean.app/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass123"}'
```

### Test Droplet Deployment

```bash
# Test backend
curl https://api.your-domain.com/api/health

# Test frontend
https://your-domain.com

# Test monitoring
https://your-domain.com:3001  # Grafana
https://your-domain.com:9090  # Prometheus
```

---

## 📊 Monitoring & Maintenance

### App Platform

1. **Go to**: Your app in DigitalOcean dashboard
2. **Insights** tab: View metrics
3. **Activity** tab: View deployments
4. **Logs**: Real-time application logs
5. **Alerts**: Set up alerts for issues

### Droplet + Docker

```bash
# View logs
docker compose -f docker-compose.production.yml logs -f backend
docker compose -f docker-compose.production.yml logs -f frontend

# View resource usage
docker stats

# Access Grafana
https://your-domain.com:3001
# Login: admin / your-grafana-password

# Access Prometheus
https://your-domain.com:9090
```

---

## 💰 Cost Comparison

### App Platform
```
Frontend:         $5/month (Static Site)
Backend:          $12/month (Basic Web Service)
MongoDB:          $15/month (Managed Database)
Redis:            $15/month (Managed Database)
────────────────────────────
Total:            $47/month
```

### Droplet + Docker
```
Droplet:          $12/month (2GB RAM)
Or Better:        $24/month (4GB RAM)
Backups:          +20% ($2.40-4.80)
MongoDB Atlas:    FREE (or $9/month)
Redis Cloud:      FREE
────────────────────────────
Total:            $14-34/month
```

**💡 Droplet is cheaper but requires more management**

---

## 🔄 Auto-Deploy Setup

### App Platform

**Already enabled by default!**
```bash
git add .
git commit -m "Update"
git push origin main
# Auto-deploys in 3-5 minutes
```

### Droplet

**Set up GitHub webhook** or **cron job**:

```bash
# Create deploy script
nano ~/CropGPT/deploy.sh
```

```bash
#!/bin/bash
cd ~/CropGPT
git pull origin main
docker compose -f docker-compose.production.yml pull
docker compose -f docker-compose.production.yml up -d
docker system prune -f
```

```bash
# Make executable
chmod +x ~/CropGPT/deploy.sh

# Test
./deploy.sh

# Add to cron (check for updates every hour)
crontab -e
# Add:
0 * * * * cd ~/CropGPT && git fetch && [ $(git rev-parse HEAD) != $(git rev-parse @{u}) ] && ./deploy.sh
```

---

## 🎨 Custom Domain Setup

### For App Platform

1. **Go to**: Your app → Settings
2. **Domains** → Add Domain
3. **Enter domain**: `cropgpt.com`
4. **Add DNS records** at your registrar:
   ```
   Type: CNAME
   Name: @
   Value: cropgpt-xxxxx.ondigitalocean.app
   
   Type: CNAME
   Name: www
   Value: cropgpt-xxxxx.ondigitalocean.app
   ```
5. **Verify** (takes 5-60 minutes)
6. **SSL**: Auto-configured by DigitalOcean

### For Droplet

1. **Point DNS** to your droplet IP:
   ```
   Type: A
   Name: @
   Value: your_droplet_ip
   
   Type: A
   Name: www
   Value: your_droplet_ip
   
   Type: A
   Name: api
   Value: your_droplet_ip
   ```
2. **SSL already configured** with certbot (Part 3, Step 3)

---

## 🔒 Security Best Practices

### App Platform

✅ Automatically secured by DigitalOcean:
- HTTPS enforced
- DDoS protection
- Firewall managed
- Automatic security updates

### Droplet

**Configure manually**:

```bash
# 1. Keep system updated
sudo apt update && sudo apt upgrade -y

# 2. Configure firewall (already done)
sudo ufw status

# 3. Disable root login
sudo nano /etc/ssh/sshd_config
# Set: PermitRootLogin no
sudo systemctl restart sshd

# 4. Install fail2ban (brute-force protection)
sudo apt install fail2ban -y
sudo systemctl enable fail2ban

# 5. Set up automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades

# 6. Monitor logs
sudo tail -f /var/log/auth.log
```

---

## 🐛 Troubleshooting

### App Platform Issues

**Build Fails**:
```bash
# Check build logs in dashboard
# Verify package.json and requirements.txt
# Check environment variables
```

**Service Won't Start**:
```bash
# Check Runtime logs
# Verify Start Command
# Check Health Check endpoint
# Verify port (must be 8080)
```

### Droplet Issues

**Docker Container Crashes**:
```bash
# Check logs
docker compose -f docker-compose.production.yml logs backend

# Restart services
docker compose -f docker-compose.production.yml restart backend

# Full restart
docker compose -f docker-compose.production.yml down
docker compose -f docker-compose.production.yml up -d
```

**Out of Memory**:
```bash
# Check usage
docker stats

# Upgrade droplet size
# Or optimize containers
```

**SSL Certificate Issues**:
```bash
# Renew manually
sudo certbot renew

# Check expiry
sudo certbot certificates
```

---

## ✅ Deployment Checklist

### App Platform
- [ ] DigitalOcean account created
- [ ] MongoDB Managed Database created
- [ ] Redis Managed Database created
- [ ] Backend service configured
- [ ] Frontend static site configured
- [ ] Environment variables set
- [ ] Databases linked to app
- [ ] CORS configured
- [ ] Custom domain added (optional)
- [ ] App deployed and tested

### Droplet
- [ ] Droplet created
- [ ] SSH access configured
- [ ] Docker installed
- [ ] Repository cloned
- [ ] Environment variables configured
- [ ] Docker Compose deployed
- [ ] Nginx configured
- [ ] SSL certificates installed
- [ ] Firewall configured
- [ ] Monitoring set up
- [ ] Backups enabled
- [ ] Auto-deploy configured

---

## 🎉 Success!

Your CropGPT is now live on DigitalOcean!

### App Platform:
- **Frontend**: `https://cropgpt-xxxxx.ondigitalocean.app`
- **Backend**: `https://cropgpt-backend-xxxxx.ondigitalocean.app`

### Droplet:
- **Frontend**: `https://your-domain.com`
- **Backend API**: `https://api.your-domain.com`
- **Grafana**: `https://your-domain.com:3001`

**Share with farmers and make an impact! 🌾🚀**

---

## 💡 Which Method Should You Choose?

| Feature | App Platform | Droplet |
|---------|-------------|---------|
| **Ease of Setup** | ⭐⭐⭐⭐⭐ Easy | ⭐⭐⭐ Moderate |
| **Cost** | $47/month | $14-34/month |
| **Control** | Limited | Full |
| **Maintenance** | Low | High |
| **Scalability** | Easy | Manual |
| **Learning** | Less learning | More learning |

**Recommendation**:
- **Beginners** → App Platform
- **Cost-conscious** → Droplet
- **Production** → App Platform
- **Learning Docker** → Droplet

---

**Need Help?** Check:
- DigitalOcean Docs: https://docs.digitalocean.com
- Community: https://www.digitalocean.com/community
- Your deployment logs

**Happy Deploying! 🌊🚀**
