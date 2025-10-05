# 🎨 Complete Render Deployment Guide for CropGPT

Step-by-step guide to deploy your full-stack CropGPT application on Render.

## 🎯 Deployment Architecture

```
┌─────────────────────────┐
│  Render Static Site     │ ← Frontend (React)
│  Global CDN             │   ✅ Free Forever
└───────────┬─────────────┘
            │ HTTPS
            ↓
┌─────────────────────────┐
│  Render Web Service     │ ← Backend (FastAPI)
│  Python Runtime         │   ✅ $7/month
└───────────┬─────────────┘
            │
    ┌───────┴────────┐
    ↓                ↓
┌─────────┐    ┌──────────┐
│ MongoDB │    │  Redis   │
│ Atlas   │    │  Cloud   │
│ Free    │    │  Free    │
└─────────┘    └──────────┘
```

## 📋 Prerequisites

1. **GitHub Account** - Your repo pushed: ✅ https://github.com/BishalJena/CropGPT
2. **Render Account** - Sign up at: https://render.com (free)
3. **MongoDB Atlas** - Sign up at: https://www.mongodb.com/atlas (free tier)
4. **Redis Cloud** - Sign up at: https://redis.com/try-free (free tier)
5. **API Keys Ready** - Cerebras, OpenRouter, Deepgram, etc.

---

## Part 1: Set Up Cloud Databases

### Step 1: Create MongoDB Atlas Database (Free)

1. **Go to MongoDB Atlas**: https://www.mongodb.com/cloud/atlas/register
2. **Sign up** with Google/GitHub
3. **Create a Free Cluster**:
   - Click "Build a Database"
   - Choose "M0 Free" tier
   - Select region (choose closest to you)
   - Cluster name: `cropgpt-cluster`
   - Click "Create"

4. **Create Database User**:
   - Go to "Database Access"
   - Click "Add New Database User"
   - Username: `cropgpt`
   - Password: Generate strong password (save it!)
   - Database User Privileges: "Read and write to any database"
   - Click "Add User"

5. **Configure Network Access**:
   - Go to "Network Access"
   - Click "Add IP Address"
   - Click "Allow Access from Anywhere" (0.0.0.0/0)
   - Confirm

6. **Get Connection String**:
   - Go to "Database" → "Connect"
   - Choose "Connect your application"
   - Driver: Python, Version: 3.6 or later
   - Copy connection string:
   ```
   mongodb+srv://cropgpt:<password>@cropgpt-cluster.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
   - **Replace `<password>` with your actual password**
   - **Save this connection string!**

### Step 2: Create Redis Cloud Database (Free)

1. **Go to Redis Cloud**: https://redis.com/try-free
2. **Sign up** with Google/GitHub
3. **Create Free Database**:
   - Click "New Subscription"
   - Choose "Free" plan
   - Cloud: Any (AWS recommended)
   - Region: Same as MongoDB (for low latency)
   - Name: `cropgpt-redis`
   - Click "Create subscription"

4. **Create Database**:
   - Click "New database"
   - Database name: `cropgpt`
   - Click "Activate database"

5. **Get Connection Details**:
   - Go to your database
   - Copy "Public endpoint": `redis-xxxxx.c1.us-east-1-2.ec2.cloud.redislabs.com:12345`
   - Copy "Default user password"
   - **Connection string format**:
   ```
   redis://:<password>@redis-xxxxx.c1.us-east-1-2.ec2.cloud.redislabs.com:12345
   ```
   - **Save this connection string!**

---

## Part 2: Deploy Backend to Render

### Step 1: Create Render Account

1. Go to https://render.com
2. Click "Get Started"
3. Sign up with GitHub
4. Authorize Render to access your repositories

### Step 2: Create Backend Web Service

1. **Click "New +"** → **"Web Service"**

2. **Connect Repository**:
   - Select "Build and deploy from a Git repository"
   - Connect your GitHub account (if not already)
   - Select repository: `BishalJena/CropGPT`
   - Click "Connect"

3. **Configure Web Service**:

   **Basic Settings:**
   ```
   Name: cropgpt-backend
   Region: Oregon (US West) or closest to you
   Branch: main
   Root Directory: backend
   Runtime: Python 3
   ```

   **Build & Deploy:**
   ```
   Build Command: pip install -r requirements.txt
   Start Command: uvicorn server:app --host 0.0.0.0 --port $PORT
   ```

   **Instance Type:**
   ```
   Select: Starter ($7/month)
   Or: Free (for testing, spins down after 15 min inactivity)
   ```

4. **Add Environment Variables** - Click "Advanced" → "Add Environment Variable":

   **Required Variables:**
   ```env
   # Database (from MongoDB Atlas)
   MONGO_URL=mongodb+srv://cropgpt:<password>@cropgpt-cluster.xxxxx.mongodb.net/?retryWrites=true&w=majority
   DB_NAME=farmchat
   
   # Redis (from Redis Cloud)
   REDIS_URL=redis://:<password>@redis-xxxxx.c1.us-east-1-2.ec2.cloud.redislabs.com:12345
   
   # Authentication - Generate 32-char secret
   JWT_SECRET=<your-32-char-random-secret>
   JWT_EXPIRATION_HOURS=24
   
   # AI Services - YOUR API KEYS
   CEREBRAS_API_KEY=csk-xxxxxxxxxxxxx
   OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxx
   DEEPGRAM_API_KEY=xxxxxxxxxxxxx
   
   # External APIs
   EXA_API_KEY=xxxxxxxxxxxxx
   DATAGOVIN_API_KEY=xxxxxxxxxxxxx
   
   # MCP Gateway
   MCP_GATEWAY_URL=http://165.232.190.215:8811
   
   # CORS - Will update after frontend deployment
   CORS_ORIGINS=*
   
   # Server Configuration
   ENVIRONMENT=production
   LOG_LEVEL=INFO
   ```

   **Generate JWT_SECRET:**
   ```bash
   python -c "import secrets; print(secrets.token_urlsafe(32))"
   ```

5. **Create Web Service**:
   - Click "Create Web Service"
   - Render will start building (takes 3-5 minutes)
   - Wait for "Live" status ✅

6. **Get Backend URL**:
   - Top of page shows: `https://cropgpt-backend.onrender.com`
   - **Copy this URL - you'll need it for frontend!**

7. **Test Backend**:
   ```bash
   # Test health endpoint
   curl https://cropgpt-backend.onrender.com/api/health
   
   # Should return:
   {
     "status": "healthy",
     "timestamp": "...",
     "services": {
       "database": "healthy",
       "redis": "healthy"
     }
   }
   ```

---

## Part 3: Deploy Frontend to Render

### Step 1: Create Environment File for Frontend

Before deploying frontend, create this file locally:

**File**: `frontend/.env.production`
```env
REACT_APP_BACKEND_URL=https://cropgpt-backend.onrender.com
REACT_APP_VERSION=1.0.0
REACT_APP_ENVIRONMENT=production
REACT_APP_ENABLE_VOICE=true
REACT_APP_ENABLE_WORKFLOWS=true
REACT_APP_ENABLE_MARKETPLACE=true
REACT_APP_ENABLE_SCHEMES=true
```

**⚠️ Important**: Replace `cropgpt-backend.onrender.com` with your actual backend URL!

**Commit and push:**
```bash
git add frontend/.env.production
git commit -m "Add production environment configuration"
git push origin main
```

### Step 2: Create Static Site

1. **Click "New +"** → **"Static Site"**

2. **Connect Repository**:
   - Select repository: `BishalJena/CropGPT`
   - Click "Connect"

3. **Configure Static Site**:

   **Basic Settings:**
   ```
   Name: cropgpt
   Region: Oregon (US West) or closest to you
   Branch: main
   Root Directory: frontend
   ```

   **Build Settings:**
   ```
   Build Command: npm install --legacy-peer-deps && npm run build
   Publish Directory: build
   ```

   **Environment Variables** - Add these:
   ```env
   REACT_APP_BACKEND_URL=https://cropgpt-backend.onrender.com
   REACT_APP_VERSION=1.0.0
   REACT_APP_ENVIRONMENT=production
   REACT_APP_ENABLE_VOICE=true
   REACT_APP_ENABLE_WORKFLOWS=true
   REACT_APP_ENABLE_MARKETPLACE=true
   REACT_APP_ENABLE_SCHEMES=true
   ```

4. **Auto-Deploy Settings:**
   ```
   ✅ Auto-Deploy: Yes
   ```

5. **Create Static Site**:
   - Click "Create Static Site"
   - Render will build (takes 5-10 minutes)
   - Wait for "Live" status ✅

6. **Get Frontend URL**:
   - Shows: `https://cropgpt.onrender.com`
   - **This is your live app URL!** 🎉

---

## Part 4: Connect Frontend & Backend

### Update Backend CORS Settings

1. **Go to Backend Service** on Render Dashboard
2. **Environment** → **Add Environment Variable**
3. **Update CORS_ORIGINS**:
   ```env
   CORS_ORIGINS=https://cropgpt.onrender.com,https://cropgpt-*.onrender.com
   ```
4. Render will automatically redeploy (1-2 minutes)

---

## Part 5: Configure Rewrites for React Router

Your React app uses React Router, so you need to configure rewrites:

### Option A: Using render.yaml (Recommended)

Create this file in your repository root:

**File**: `render.yaml`
```yaml
services:
  # Backend Service
  - type: web
    name: cropgpt-backend
    runtime: python
    region: oregon
    plan: starter
    buildCommand: cd backend && pip install -r requirements.txt
    startCommand: cd backend && uvicorn server:app --host 0.0.0.0 --port $PORT
    envVars:
      - key: MONGO_URL
        sync: false
      - key: REDIS_URL
        sync: false
      - key: JWT_SECRET
        sync: false
      - key: CEREBRAS_API_KEY
        sync: false
      - key: OPENROUTER_API_KEY
        sync: false
      - key: DEEPGRAM_API_KEY
        sync: false
      - key: EXA_API_KEY
        sync: false
      - key: DATAGOVIN_API_KEY
        sync: false
      - key: MCP_GATEWAY_URL
        value: http://165.232.190.215:8811
      - key: CORS_ORIGINS
        value: https://cropgpt.onrender.com
      - key: ENVIRONMENT
        value: production
      - key: LOG_LEVEL
        value: INFO
      - key: DB_NAME
        value: farmchat
      - key: JWT_EXPIRATION_HOURS
        value: 24

  # Frontend Static Site
  - type: web
    name: cropgpt
    runtime: static
    region: oregon
    buildCommand: cd frontend && npm install --legacy-peer-deps && npm run build
    staticPublishPath: frontend/build
    routes:
      - type: rewrite
        source: /*
        destination: /index.html
    envVars:
      - key: REACT_APP_BACKEND_URL
        value: https://cropgpt-backend.onrender.com
      - key: REACT_APP_VERSION
        value: 1.0.0
      - key: REACT_APP_ENVIRONMENT
        value: production
```

**Commit and push:**
```bash
git add render.yaml
git commit -m "Add Render configuration file"
git push origin main
```

### Option B: Using Render Dashboard

1. Go to your **frontend static site**
2. Click **"Redirects/Rewrites"**
3. Add this rewrite rule:
   ```
   Source: /*
   Destination: /index.html
   Action: Rewrite
   ```
4. Save

---

## 🧪 Testing Your Deployment

### Test Backend

```bash
# Health check
curl https://cropgpt-backend.onrender.com/api/health

# API documentation
https://cropgpt-backend.onrender.com/docs

# Register a test user
curl -X POST https://cropgpt-backend.onrender.com/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "testpass123"
  }'
```

### Test Frontend

1. **Visit**: https://cropgpt.onrender.com
2. **Register** a new user
3. **Login**
4. **Send a message**: "What is the wheat price in Punjab?"
5. **Verify** you get an AI response

### Test CORS

```bash
curl -H "Origin: https://cropgpt.onrender.com" \
     -H "Access-Control-Request-Method: POST" \
     -H "Access-Control-Request-Headers: Content-Type" \
     -X OPTIONS \
     https://cropgpt-backend.onrender.com/api/chat
```

Should return CORS headers.

---

## 🔧 Configuration Files

### Option 1: Using render.yaml (Infrastructure as Code)

This is the recommended approach. The `render.yaml` file above defines your entire infrastructure.

**Benefits:**
- ✅ Version controlled
- ✅ Easy to replicate
- ✅ Automatic deploys
- ✅ Environment variables documented

### Option 2: Manual Configuration

Use the Render Dashboard as described in Parts 2 & 3.

**Benefits:**
- ✅ Visual interface
- ✅ Step-by-step control
- ✅ Easy for beginners

---

## 📊 Monitoring & Logs

### View Logs

**Backend:**
1. Go to your backend service
2. Click "Logs" tab
3. See real-time logs
4. Filter by: All, Info, Warning, Error

**Frontend:**
1. Go to your static site
2. Click "Events" tab
3. See deployment history
4. View build logs

### Metrics

1. Click "Metrics" tab
2. View:
   - Request count
   - Response times
   - Bandwidth usage
   - Error rates

### Set Up Notifications

1. Go to "Settings"
2. Click "Notifications"
3. Add email or Slack webhook
4. Get notified on:
   - Deploy failures
   - Service health issues
   - Resource limits

---

## 💰 Cost Breakdown

### Free Tier (Testing)
- **Frontend Static Site**: FREE ✅ (100GB bandwidth)
- **Backend Web Service**: FREE ✅ (spins down after 15 min)
- **MongoDB Atlas**: FREE (512MB)
- **Redis Cloud**: FREE (30MB)
- **Total**: $0/month

### Production Tier
- **Frontend Static Site**: FREE ✅
- **Backend Web Service**: $7/month (Starter - stays awake)
- **MongoDB Atlas**: $9/month (Shared M2) OR use free tier
- **Redis Cloud**: FREE (sufficient for most use)
- **Total**: $7-16/month

### Cost Comparison

| Service | Render | Railway | Vercel+Railway |
|---------|--------|---------|----------------|
| Frontend | FREE | N/A | FREE |
| Backend | $7/mo | $10-15/mo | $10-15/mo |
| Database | External | Included | External |
| **Total** | **$7/mo** | **$10-15/mo** | **$10-15/mo** |

**Winner: Render!** Most affordable with free static hosting. 🏆

---

## 🚀 Auto-Deploy Setup

Render automatically deploys when you push to GitHub!

### Enable Auto-Deploy

**Already enabled by default!** Just:

```bash
# Make changes
git add .
git commit -m "Update feature"
git push origin main

# Render auto-deploys in 2-5 minutes
```

### Deploy Hooks (Optional)

For manual or API-triggered deploys:

1. Go to service **Settings**
2. Click **"Deploy Hook"**
3. Copy URL: `https://api.render.com/deploy/srv-xxxxx`
4. Trigger deploy:
   ```bash
   curl https://api.render.com/deploy/srv-xxxxx
   ```

---

## 🎨 Custom Domain (Optional)

### Add Custom Domain to Frontend

1. Go to your static site
2. Click **"Settings"** → **"Custom Domain"**
3. Click **"Add Custom Domain"**
4. Enter: `cropgpt.com` (or your domain)
5. Render provides DNS records:
   ```
   Type: CNAME
   Name: cropgpt.com
   Value: cropgpt.onrender.com
   ```
6. **Add DNS record** at your domain registrar
7. Wait for DNS propagation (5-60 minutes)
8. ✅ SSL certificate auto-generated

### Add Custom Domain to Backend

1. Go to your backend service
2. **Settings** → **"Custom Domain"**
3. Enter: `api.cropgpt.com`
4. Add DNS record:
   ```
   Type: CNAME
   Name: api.cropgpt.com
   Value: cropgpt-backend.onrender.com
   ```
5. **Update frontend** `REACT_APP_BACKEND_URL`:
   ```env
   REACT_APP_BACKEND_URL=https://api.cropgpt.com
   ```

---

## 🔒 Security Best Practices

### 1. Secure Environment Variables

✅ Use Render's environment variables (encrypted at rest)
❌ Never commit `.env` files to GitHub

### 2. Rotate Secrets Regularly

```bash
# Generate new JWT secret every 90 days
python -c "import secrets; print(secrets.token_urlsafe(32))"

# Update in Render dashboard
```

### 3. Use Strong API Keys

All your API keys should be:
- Minimum 32 characters
- Random and unpredictable
- Never reused across services

### 4. Configure CORS Properly

```env
# Don't use * in production
CORS_ORIGINS=https://cropgpt.onrender.com,https://cropgpt-*.onrender.com

# If using custom domain
CORS_ORIGINS=https://cropgpt.com,https://www.cropgpt.com
```

### 5. Enable Rate Limiting

Your FastAPI backend already has rate limiting. Monitor in logs.

---

## 🐛 Troubleshooting

### Issue 1: Build Failed

**Error**: `npm install` fails or `pip install` fails

**Solution**:
```bash
# Test locally first
cd frontend
npm install --legacy-peer-deps
npm run build

cd ../backend
pip install -r requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8000
```

If works locally, check:
- Render build logs for specific error
- Node version (should be 16+)
- Python version (should be 3.8+)

### Issue 2: CORS Error

**Error**: "CORS policy: No 'Access-Control-Allow-Origin' header"

**Solution**:
1. Check `CORS_ORIGINS` in backend includes frontend URL
2. Format: `https://cropgpt.onrender.com` (no trailing slash)
3. Backend should auto-redeploy after changing env var
4. Clear browser cache and retry

### Issue 3: MongoDB Connection Failed

**Error**: "MongoNetworkError" or "Authentication failed"

**Solution**:
1. Verify connection string format:
   ```
   mongodb+srv://username:password@cluster.mongodb.net/database?retryWrites=true&w=majority
   ```
2. Check password (no special URL characters, or URL-encode them)
3. Verify network access (0.0.0.0/0 allowed)
4. Test connection locally:
   ```python
   from pymongo import MongoClient
   client = MongoClient("your-connection-string")
   print(client.server_info())
   ```

### Issue 4: Backend Slow/Timing Out

**Problem**: Free tier spins down after 15 min inactivity

**Solution**:
1. Upgrade to Starter plan ($7/month - always awake)
2. Or: Keep alive with ping service:
   ```bash
   # Use UptimeRobot or similar
   # Ping: https://cropgpt-backend.onrender.com/api/health
   # Every 14 minutes
   ```

### Issue 5: Frontend 404 Errors

**Problem**: React Router routes return 404

**Solution**:
1. Add rewrite rule (see Part 5)
2. Verify `staticPublishPath: frontend/build`
3. Check routes configuration in render.yaml

### Issue 6: Environment Variables Not Working

**Problem**: `REACT_APP_*` variables undefined

**Solution**:
1. Verify variables start with `REACT_APP_`
2. Rebuild frontend after adding variables
3. Check browser console: `console.log(process.env.REACT_APP_BACKEND_URL)`
4. Variables are embedded at build time, not runtime

---

## 📚 Additional Resources

- **Render Docs**: https://render.com/docs
- **FastAPI Deployment**: https://render.com/docs/deploy-fastapi
- **React Deployment**: https://render.com/docs/deploy-create-react-app
- **MongoDB Atlas**: https://www.mongodb.com/docs/atlas/
- **Redis Cloud**: https://redis.io/docs/

---

## ✅ Deployment Checklist

### Backend
- [ ] MongoDB Atlas created
- [ ] Redis Cloud created
- [ ] Connection strings copied
- [ ] Backend service created on Render
- [ ] Environment variables configured
- [ ] Backend deployed and live
- [ ] Health endpoint tested
- [ ] Backend URL copied

### Frontend
- [ ] `.env.production` created
- [ ] Environment file committed and pushed
- [ ] Static site created on Render
- [ ] Environment variables configured
- [ ] Frontend deployed and live
- [ ] Rewrite rules configured
- [ ] Frontend URL copied

### Integration
- [ ] CORS updated with frontend URL
- [ ] Backend redeployed
- [ ] End-to-end test passed
- [ ] User registration works
- [ ] Chat functionality works
- [ ] Voice features tested
- [ ] Custom domain configured (optional)

---

## 🎉 Success!

Your CropGPT application is now live on Render:

- **Frontend**: https://cropgpt.onrender.com
- **Backend**: https://cropgpt-backend.onrender.com
- **API Docs**: https://cropgpt-backend.onrender.com/docs

**Features**:
- ✅ Auto-deploy from GitHub
- ✅ Free SSL certificates
- ✅ Global CDN for frontend
- ✅ MongoDB + Redis integrated
- ✅ Environment variables secured
- ✅ Monitoring and logs

**Share your app with farmers and start making an impact! 🌾🚀**

---

## 💡 Next Steps

1. **Monitor Usage**: Check Render dashboard for metrics
2. **Add Analytics**: Google Analytics, Mixpanel
3. **Set Up Monitoring**: Sentry for error tracking
4. **Optimize Performance**: Enable caching, compress images
5. **Scale as Needed**: Upgrade plans based on traffic
6. **Add Features**: Voice input, workflows, schemes
7. **Get Feedback**: Test with real farmers

---

**Need Help?** Check:
- Render Docs: https://render.com/docs
- Render Community: https://community.render.com
- GitHub Issues: Check your deployment logs

**Happy Deploying! 🎨🚀**
