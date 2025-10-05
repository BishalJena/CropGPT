# 🚀 Deploy ALL Components to DigitalOcean

Complete guide for deploying **Frontend + Backend + fs-gate (MCP Gateway)** on DigitalOcean.

---

## ✅ YES! Deploy All Three Components

Your CropGPT application has 3 main components, and you can deploy **ALL OF THEM** on DigitalOcean:

1. ✅ **Frontend** - React application (user interface)
2. ✅ **Backend** - FastAPI server (main API)
3. ✅ **fs-gate** - MCP Gateway (agricultural tools server)

---

## 🎯 Two Deployment Methods

### Method 1: App Platform (Recommended - All 3 Components) 🌟

Deploy everything as managed services with auto-scaling and SSL.

#### What You Get:
```
Frontend (React)          →  Static Site ($5/mo)
Backend (FastAPI)         →  Web Service ($12/mo)
fs-gate (MCP Gateway)     →  Web Service ($12/mo)
MongoDB                   →  Managed DB ($15/mo) or External
Redis                     →  Managed DB ($15/mo) or External
─────────────────────────────────────────────────
Total with DO Databases:     $59/month
Total with External DBs:     $29/month
```

#### Architecture:
```
┌─────────────────────────────────────────┐
│       DigitalOcean App Platform         │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────────────┐   ┌──────────────┐  │
│  │   Frontend   │   │   Backend    │  │
│  │   (React)    │──→│  (FastAPI)   │  │
│  │   Static     │   │ Web Service  │  │
│  └──────────────┘   └──────┬───────┘  │
│                             │           │
│                             ↓           │
│                    ┌──────────────┐    │
│                    │   fs-gate    │    │
│                    │ MCP Gateway  │    │
│                    │ Web Service  │    │
│                    └──────────────┘    │
│                                         │
└─────────────────────────────────────────┘
           ↓                ↓
    ┌──────────┐    ┌──────────┐
    │ MongoDB  │    │  Redis   │
    │ Atlas    │    │  Cloud   │
    │ (FREE)   │    │ (FREE)   │
    └──────────┘    └──────────┘
```

---

### Method 2: Droplet + Docker (All Components) 💻

Run everything on a single virtual machine using Docker Compose.

#### What You Get:
```
Droplet (2GB RAM)         →  $12-24/mo (all components)
  ├── Frontend (Docker)
  ├── Backend (Docker)
  ├── fs-gate (Docker)
  ├── MongoDB (Docker or External)
  ├── Redis (Docker or External)
  ├── Nginx (Reverse Proxy)
  ├── Prometheus (Monitoring)
  └── Grafana (Dashboard)
─────────────────────────────────────────────────
Total:                       $12-24/month
```

#### Architecture:
```
┌─────────────────────────────────────────────┐
│        DigitalOcean Droplet ($12-24/mo)     │
├─────────────────────────────────────────────┤
│                                             │
│  ┌───────────────────────────────────────┐ │
│  │         Nginx (Reverse Proxy)         │ │
│  └───────────────┬───────────────────────┘ │
│                  │                          │
│    ┌─────────────┼─────────────┐           │
│    ↓             ↓             ↓           │
│  ┌────┐      ┌────────┐    ┌────────┐     │
│  │React│      │FastAPI │    │fs-gate │     │
│  │:80 │      │:8000   │    │:10000  │     │
│  └────┘      └───┬────┘    └───┬────┘     │
│                  │             │            │
│                  ↓             ↓            │
│            ┌─────────┐   ┌─────────┐       │
│            │ MongoDB │   │  Redis  │       │
│            │ :27017  │   │  :6379  │       │
│            └─────────┘   └─────────┘       │
│                                             │
│  ┌────────────┐  ┌──────────────┐          │
│  │ Prometheus │  │   Grafana    │          │
│  │   :9090    │  │    :3001     │          │
│  └────────────┘  └──────────────┘          │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 📝 Quick Deployment: Method 1 (App Platform)

### Step 1: Sign Up for DigitalOcean

1. Go to: https://www.digitalocean.com
2. Sign up (get $200 free credit!)
3. Add payment method

### Step 2: Deploy Using app.yaml

Your repository already has `.do/app.yaml` configured with all 3 components!

```bash
# Option A: Use DigitalOcean Dashboard
1. Go to: https://cloud.digitalocean.com/apps
2. Click "Create App"
3. Select "GitHub"
4. Choose repository: BishalJena/CropGPT
5. Choose branch: main
6. Click "Next"
7. DigitalOcean will auto-detect .do/app.yaml
8. Review configuration (all 3 components included!)
9. Add environment variables (see below)
10. Click "Create Resources"

# Option B: Use doctl CLI
doctl auth init
doctl apps create --spec .do/app.yaml
```

### Step 3: Add Environment Variables

After app creation, add these secrets in the dashboard:

#### For Backend Service:
```env
MONGO_URL=<your-mongodb-connection-string>
REDIS_URL=<your-redis-connection-string>
JWT_SECRET=<generate-32-char-secret>
CEREBRAS_API_KEY=<your-cerebras-key>
OPENROUTER_API_KEY=<your-openrouter-key>
DEEPGRAM_API_KEY=<your-deepgram-key>
EXA_API_KEY=<your-exa-key>
DATAGOVIN_API_KEY=<your-datagovin-key>
```

#### For fs-gate Service:
```env
DATAGOVIN_API_KEY=<your-datagovin-key>
EXA_API_KEY=<your-exa-key>
```

### Step 4: Get Your URLs

After deployment (10-15 minutes), you'll get:

```
Frontend:     https://cropgpt-xxxxx.ondigitalocean.app
Backend:      https://cropgpt-backend-xxxxx.ondigitalocean.app
fs-gate:      https://cropgpt-mcp-gateway-xxxxx.ondigitalocean.app
```

**All URLs are auto-configured! The backend will automatically connect to fs-gate.**

---

## 📝 Quick Deployment: Method 2 (Droplet + Docker)

### Step 1: Create Droplet

```bash
# 1. Go to DigitalOcean Dashboard
# 2. Create Droplet
# 3. Choose:
#    - Ubuntu 22.04 LTS
#    - 2GB RAM ($12/mo) or 4GB RAM ($24/mo)
#    - Enable Monitoring
#    - Add SSH Key
# 4. Create Droplet
```

### Step 2: Setup Server

```bash
# SSH into your droplet
ssh root@your_droplet_ip

# Update system
apt update && apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Install Docker Compose
apt install docker-compose-plugin -y

# Clone your repository
git clone https://github.com/BishalJena/CropGPT.git
cd CropGPT
```

### Step 3: Configure Environment

```bash
# Create .env file
nano .env
```

Add these variables:
```env
# MongoDB
MONGO_ROOT_PASSWORD=your-secure-mongo-password

# Redis
REDIS_PASSWORD=your-secure-redis-password

# Authentication
JWT_SECRET=your-32-char-random-secret

# AI Services
CEREBRAS_API_KEY=your-cerebras-key
OPENROUTER_API_KEY=your-openrouter-key
DEEPGRAM_API_KEY=your-deepgram-key

# External APIs
EXA_API_KEY=your-exa-key
DATAGOVIN_API_KEY=your-datagovin-key

# Monitoring
GRAFANA_USER=admin
GRAFANA_PASSWORD=your-grafana-password
```

### Step 4: Deploy All Services

```bash
# Start all services (Frontend + Backend + fs-gate + DBs + Monitoring)
docker compose -f docker-compose.production.yml up -d

# Check status
docker compose -f docker-compose.production.yml ps

# You should see:
# ✅ agricultural-ai-frontend
# ✅ agricultural-ai-backend
# ✅ agricultural-ai-mcp-server (fs-gate)
# ✅ agricultural-ai-mongodb
# ✅ agricultural-ai-redis
# ✅ agricultural-ai-nginx
# ✅ agricultural-ai-prometheus
# ✅ agricultural-ai-grafana

# View logs
docker compose -f docker-compose.production.yml logs -f
```

### Step 5: Access Your Application

```
Frontend:     http://your_droplet_ip:3000
Backend API:  http://your_droplet_ip:8000
fs-gate:      http://your_droplet_ip:10000
Grafana:      http://your_droplet_ip:3001
Prometheus:   http://your_droplet_ip:9090
```

---

## 🆚 Method Comparison

| Feature | App Platform | Droplet + Docker |
|---------|-------------|------------------|
| **All 3 Components** | ✅ Yes | ✅ Yes |
| **Setup Time** | 20 minutes | 60 minutes |
| **Difficulty** | ⭐⭐ Easy | ⭐⭐⭐ Moderate |
| **Cost/month** | $29-59 | $12-24 |
| **Auto SSL** | ✅ Yes | Manual setup |
| **Auto Deploy** | ✅ Yes | Manual/webhook |
| **Monitoring** | Basic | ✅ Full (Grafana) |
| **Control** | Managed | ✅ Full control |
| **Scalability** | ✅ Easy | Manual |
| **Best For** | Production, Teams | Learning, Cost-saving |

---

## 💰 Cost Breakdown

### App Platform (All Components)

#### Option A: With DigitalOcean Databases
```
Frontend Static Site:      $5/month
Backend Web Service:       $12/month
fs-gate Web Service:       $12/month
MongoDB Managed:           $15/month
Redis Managed:             $15/month
─────────────────────────────────
Total:                     $59/month
```

#### Option B: With External Databases (Recommended)
```
Frontend Static Site:      $5/month
Backend Web Service:       $12/month
fs-gate Web Service:       $12/month
MongoDB Atlas (FREE):      $0/month
Redis Cloud (FREE):        $0/month
─────────────────────────────────
Total:                     $29/month ✨
```

### Droplet + Docker (All Components)

```
Droplet (2GB):             $12/month
Or Droplet (4GB):          $24/month
MongoDB Atlas (FREE):      $0/month
Redis Cloud (FREE):        $0/month
─────────────────────────────────
Total:                     $12-24/month 🏆 Cheapest!
```

---

## 🎯 My Recommendation

### For Your CropGPT Project:

#### 🥇 Best Choice: **App Platform with External DBs** ($29/month)

**Why:**
- ✅ All 3 components deployed
- ✅ Easy setup (20 minutes)
- ✅ Auto-deploy from GitHub
- ✅ Professional URLs with SSL
- ✅ Auto-scaling when you grow
- ✅ Monitoring included
- ✅ Perfect for demos/production

**Steps:**
1. Follow "Method 1" above
2. Use MongoDB Atlas (FREE 512MB)
3. Use Redis Cloud (FREE 30MB)
4. Deploy all 3 components
5. Done in 20 minutes!

---

#### 🥈 Alternative: **Droplet + Docker** ($12-24/month)

**Why:**
- ✅ Lowest cost
- ✅ Full control
- ✅ Learn Docker/DevOps
- ✅ Complete monitoring with Grafana
- ✅ Great for resume

**Steps:**
1. Follow "Method 2" above
2. One command deploys everything
3. Access Grafana dashboards
4. Perfect for learning!

---

## ✅ What's Already Configured

Your repository is **READY TO DEPLOY** with:

### 1. `.do/app.yaml` - App Platform Config
```yaml
✅ Frontend static site
✅ Backend web service  
✅ fs-gate web service (NOW ENABLED!)
✅ Auto-deployed from GitHub
✅ Environment variables template
✅ Health checks configured
✅ CORS properly set
```

### 2. `docker-compose.production.yml` - Droplet Config
```yaml
✅ Frontend container
✅ Backend container
✅ fs-gate container
✅ MongoDB container
✅ Redis container
✅ Nginx reverse proxy
✅ Prometheus + Grafana monitoring
✅ Health checks for all services
```

**Both methods deploy ALL 3 COMPONENTS!** 🎉

---

## 🚀 Quick Start (Choose Your Method)

### App Platform (Easy):
```bash
# 1. Go to DigitalOcean
https://cloud.digitalocean.com/apps

# 2. Create App from GitHub
Repository: BishalJena/CropGPT
Branch: main

# 3. Auto-detects .do/app.yaml
(All 3 components configured!)

# 4. Add environment variables
(See "Add Environment Variables" above)

# 5. Deploy!
(Wait 10-15 minutes)

# 6. Done!
All 3 components live with SSL!
```

### Droplet + Docker (Advanced):
```bash
# 1. Create Droplet
# 2. SSH into droplet
ssh root@your_droplet_ip

# 3. One-liner setup
curl -fsSL https://get.docker.com | sh && \
git clone https://github.com/BishalJena/CropGPT.git && \
cd CropGPT && \
nano .env  # Add your variables

# 4. Deploy everything
docker compose -f docker-compose.production.yml up -d

# 5. Done!
All 3 components + monitoring live!
```

---

## 📊 Service URLs After Deployment

### App Platform:
```
Frontend:     https://cropgpt-xxxxx.ondigitalocean.app
Backend API:  https://cropgpt-backend-xxxxx.ondigitalocean.app/api
fs-gate:      https://cropgpt-mcp-gateway-xxxxx.ondigitalocean.app
Health:       https://cropgpt-backend-xxxxx.ondigitalocean.app/api/health
```

### Droplet:
```
Frontend:     http://your_ip:3000
Backend API:  http://your_ip:8000/api
fs-gate:      http://your_ip:10000
Monitoring:   http://your_ip:3001 (Grafana)
Prometheus:   http://your_ip:9090
Nginx Proxy:  http://your_ip:80
```

---

## 🔗 Inter-Service Communication

### How Components Talk to Each Other:

#### App Platform (Automatic):
```
Frontend calls Backend via:
  ${backend.PUBLIC_URL}

Backend calls fs-gate via:
  ${mcp-gateway.PUBLIC_URL}

✨ All URLs auto-configured by DigitalOcean!
```

#### Droplet (Docker Network):
```
Frontend calls Backend via:
  http://backend:8000

Backend calls fs-gate via:
  http://mcp-server:10000

Nginx routes external traffic:
  :80 → Frontend
  :8000 → Backend API
  :10000 → fs-gate

✨ All networking handled by Docker Compose!
```

---

## ✨ Summary

### Can You Deploy All 3 Components?

# 🎉 YES! ABSOLUTELY! 🎉

### Which Method?

| Your Need | Best Choice |
|-----------|-------------|
| **Easiest & Professional** | App Platform |
| **Cheapest & Full Control** | Droplet + Docker |
| **Production Ready** | App Platform |
| **Learning/Portfolio** | Droplet + Docker |

### Both Are Fully Configured!

- ✅ `.do/app.yaml` - Ready for App Platform
- ✅ `docker-compose.production.yml` - Ready for Droplet
- ✅ All 3 components included
- ✅ Just add environment variables
- ✅ Deploy and go!

---

## 🎓 Next Steps

1. **Choose your method** (App Platform or Droplet)
2. **Follow the guide** above for your chosen method
3. **Add environment variables** (MongoDB, Redis, API keys)
4. **Deploy!** (20-60 minutes depending on method)
5. **Test all components** (Frontend, Backend, fs-gate)
6. **Share with farmers!** 🌾

---

## 📚 Additional Resources

- **Full Deployment Guide**: `DIGITALOCEAN_DEPLOYMENT_GUIDE.md`
- **Platform Comparison**: `DEPLOYMENT_COMPARISON.md`
- **DigitalOcean Docs**: https://docs.digitalocean.com/products/app-platform/

---

**Ready to deploy? Pick your method and let's go! 🚀**

**Questions? Just ask! I'm here to help! 💬**
