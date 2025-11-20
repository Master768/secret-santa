# 🚀 Deployment Guide

This guide covers deploying your Secret Santa application to production using MongoDB Atlas and various hosting platforms.

## Part 1: MongoDB Atlas Setup

### Step 1: Create MongoDB Atlas Account

1. Go to [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
2. Click "Try Free" or "Sign In"
3. Create an account or sign in with Google/GitHub

### Step 2: Create a Free Cluster

1. After signing in, click "**Build a Database**"
2. Choose "**FREE**" shared cluster (M0 Sandbox)
3. Choose your preferred cloud provider and region (closest to your users)
4. Click "**Create Cluster**" (takes 3-5 minutes)

### Step 3: Create Database User

1. In the left sidebar, click "**Database Access**"
2. Click "**Add New Database User**"
3. Choose "**Password**" authentication
4. Enter a username (e.g., `secretsanta_user`)
5. Click "**Autogenerate Secure Password**" and save it somewhere safe
6. Set privileges to "**Atlas Admin**" or "**Read and write to any database**"
7. Click "**Add User**"

### Step 4: Configure Network Access

1. In the left sidebar, click "**Network Access**"
2. Click "**Add IP Address**"
3. Click "**Allow Access from Anywhere**" (0.0.0.0/0)
   - Note: For production, restrict to specific IPs
4. Click "**Confirm**"

### Step 5: Get Connection String

1. Go back to "**Database**" in the left sidebar
2. Click "**Connect**" button on your cluster
3. Choose "**Connect your application**"
4. Select Driver: **Python**, Version: **3.12 or later**
5. Copy the connection string (looks like):
   ```
   mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
6. Replace `<username>` with your database username
7. Replace `<password>` with your database password
8. Save this connection string - you'll need it!

### Step 6: Update Your .env File

Update your local `.env` file with the MongoDB Atlas connection string:

```env
MONGO_URL=mongodb+srv://secretsanta_user:YOUR_PASSWORD@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
DB_NAME=secret_santa_db
CORS_ORIGINS=*
```

Test locally to ensure it works:
```bash
uvicorn backend.main:app --reload
```

---

## Part 2: Deploying to Hosting Platforms

### Option A: Render (Recommended - Free Tier)

#### Prerequisites
- GitHub account
- Code pushed to GitHub repository
- MongoDB Atlas configured

#### Steps

1. **Push code to GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/Master768/secret-santa.git
   git push -u origin main
   ```

2. **Create Render account**
   - Go to [render.com](https://render.com)
   - Sign up with GitHub

3. **Create New Web Service**
   - Click "**New +**" → "**Web Service**"
   - Connect your GitHub repository
   - Select your Secret Santa repository

4. **Configure Web Service**
   - **Name**: `secret-santa-app` (or your choice)
   - **Environment**: `Python 3`
   - **Build Command**: `pip install -r backend/requirements.txt`
   - **Start Command**: `uvicorn backend.main:app --host 0.0.0.0 --port $PORT`
   - **Instance Type**: Free

5. **Add Environment Variables**
   Click "Advanced" → "Add Environment Variable":
   
   | Key | Value |
   |-----|-------|
   | `MONGO_URL` | Your MongoDB Atlas connection string |
   | `DB_NAME` | `secret_santa_db` |
   | `CORS_ORIGINS` | `*` (or your frontend URL) |
   | `PYTHON_VERSION` | `3.11.0` |

6. **Deploy**
   - Click "**Create Web Service**"
   - Wait for deployment (3-5 minutes)
   - Your app will be at `https://secret-santa-app.onrender.com`

7. **Update Frontend (if separate)**
   If serving frontend separately, update WebSocket and API URLs in `frontend/script.js`

---

### Option B: Railway

#### Steps

1. **Push to GitHub** (same as Render)

2. **Create Railway account**
   - Go to [railway.app](https://railway.app)
   - Sign up with GitHub

3. **Create New Project**
   - Click "**New Project**"
   - Select "**Deploy from GitHub repo**"
   - Choose your repository

4. **Configure Build**
   Railway auto-detects Python. Add environment variables:
   - Go to "**Variables**" tab
   - Add: `MONGO_URL`, `DB_NAME`, `CORS_ORIGINS`

5. **Configure Start Command**
   - Go to "**Settings**" → "**Deploy**"
   - Custom Start Command: `uvicorn backend.main:app --host 0.0.0.0 --port $PORT`

6. **Deploy**
   - Railway auto-deploys
   - Get your URL from "**Settings**" → "**Domains**"

---

### Option C: Heroku

#### Prerequisites
- Heroku account and CLI installed

#### Steps

1. **Create Procfile**
   Create `Procfile` in project root:
   ```
   web: uvicorn backend.main:app --host 0.0.0.0 --port $PORT
   ```

2. **Create runtime.txt**
   Create `runtime.txt` in project root:
   ```
   python-3.11.0
   ```

3. **Deploy to Heroku**
   ```bash
   heroku login
   heroku create secret-santa-app
   
   # Set environment variables
   heroku config:set MONGO_URL="mongodb+srv://..."
   heroku config:set DB_NAME="secret_santa_db"
   heroku config:set CORS_ORIGINS="*"
   
   # Deploy
   git push heroku main
   ```

4. **Open app**
   ```bash
   heroku open
   ```

---

## Part 3: Frontend Deployment

### If Backend and Frontend are Separate

#### Deploy Frontend to Netlify/Vercel

1. **Update API URLs** in `frontend/script.js`:
   ```javascript
   const API_URL = 'https://your-backend-url.onrender.com';
   const WS_URL = 'wss://your-backend-url.onrender.com';
   ```

2. **Deploy to Netlify**:
   - Drag and drop `frontend` folder to [netlify.com/drop](https://app.netlify.com/drop)

3. **Deploy to Vercel**:
   ```bash
   npx vercel frontend
   ```

4. **Update CORS** in backend `.env` or environment variables:
   ```env
   CORS_ORIGINS=https://your-frontend.netlify.app,https://your-frontend.vercel.app
   ```

---

## Troubleshooting

### Database Connection Issues

**Error: "Failed to connect to MongoDB"**
- Verify connection string is correct
- Check username/password don't have special characters (URL encode if needed)
- Ensure IP whitelist includes 0.0.0.0/0 in MongoDB Atlas
- Check database user has proper permissions

### Deployment Errors

**Build fails**
- Ensure `requirements.txt` is in `backend/` directory
- Check Python version compatibility
- View deployment logs for specific errors

**App crashes on startup**
- Check environment variables are set correctly
- Verify MongoDB connection string
- Check logs for detailed error messages

**WebSocket connection fails**
- Use `wss://` (not `ws://`) for HTTPS deployments
- Check CORS settings allow your frontend domain

### Performance Issues

**Free tier limitations**:
- Render free tier spins down after 15 minutes of inactivity
- First request after spin-down takes 30-60 seconds
- Upgrade to paid tier for 24/7 uptime

---

## Security Best Practices

1. **Never commit `.env` files** - Already in `.gitignore`
2. **Use strong database passwords** - Auto-generated recommended
3. **Restrict IP access** in production MongoDB Atlas
4. **Set specific CORS_ORIGINS** instead of `*` in production
5. **Enable HTTPS** - Automatic on Render/Railway/Heroku
6. **Monitor database usage** - Free tier has 512MB limit

---

## Monitoring

### MongoDB Atlas
- View metrics in Atlas dashboard
- Set up alerts for storage/connection limits

### Render/Railway/Heroku
- Check deployment logs for errors
- Monitor uptime and response times
- Set up health check alerts

---

## Cost Breakdown

| Service | Free Tier | Limitations |
|---------|-----------|------------|
| MongoDB Atlas | ✅ Free | 512MB storage, Shared CPU |
| Render | ✅ Free | Spins down, 750hrs/month |
| Railway | ✅ $5 credit | $5/month credit |
| Heroku | ❌ Paid | $7+/month |
| Netlify | ✅ Free | 100GB bandwidth |
| Vercel | ✅ Free | 100GB bandwidth |

**Recommended Free Stack**: MongoDB Atlas + Render + Netlify/Vercel

---

## Support

If you encounter issues:
1. Check deployment logs
2. Verify environment variables
3. Test MongoDB connection locally
4. Review this guide's troubleshooting section

🎄 Happy deploying!
