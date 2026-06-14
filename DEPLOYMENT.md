# 🚀 Deployment Guide — Hostinger Business AI Plan

This guide walks you through deploying the Prandhara ERP application on **Hostinger's Business AI plan** (managed Node.js hosting).

---

## 📋 Prerequisites

- A **Hostinger** account with the **Business AI** plan (or higher)
- A **MongoDB Atlas** account (or any MongoDB provider)
- Your project pushed to a **GitHub** repository (recommended)
- A **Razorpay** account (for payments)

---

## 🗺 Deployment Overview

Hostinger's Business AI plan supports **managed Node.js hosting**. You'll deploy the backend as a Node.js application. The frontend is built to static files and served by the backend in production.

```
┌─────────────────────────────────┐
│         Hostinger Server        │
│                                 │
│  Node.js App (backend)          │
│  ├── /api/* → API routes        │
│  ├── /uploads/* → File uploads  │
│  └── /* → Serves frontend SPA   │
│                                 │
│  Persistent Storage:            │
│  └── uploads/ (file uploads)    │
│                                 │
│  External:                      │
│  └── MongoDB Atlas (database)   │
└─────────────────────────────────┘
```

---

## 🔧 Step 1 — Prepare Environment Variables

Log into your **Hostinger hPanel** → **Node.js** → **Environment Variables** and set these variables:

| Variable | Description | Example Value |
|---|---|---|
| `NODE_ENV` | Production mode | `production` |
| `PORT` | Server port (use Hostinger's assigned port) | `5000` |
| `MONGODB_URI` | MongoDB connection string | `mongodb+srv://user:pass@cluster.mongodb.net/db` |
| `JWT_SECRET` | Random secret for JWT signing | `aB9xK2mP7qR4vW8nL1cJ3fT6yH0` |
| `JWT_EXPIRES_IN` | Token expiry | `7d` |
| `EMAIL_HOST` | SMTP host | `smtp.gmail.com` |
| `EMAIL_PORT` | SMTP port | `587` |
| `EMAIL_USER` | SMTP email | `your@email.com` |
| `EMAIL_PASS` | SMTP app password | `xxxx xxxx xxxx xxxx` |
| `CLIENT_URL` | Your production domain | `https://yourdomain.com` |
| `RAZORPAY_KEY_ID` | Razorpay live key | `rzp_live_xxxx` |
| `RAZORPAY_KEY_SECRET` | Razorpay secret | `xxxx` |

> ⚠️ **Security**: Never commit real secrets to Git. Use `.env` locally and set environment variables via hPanel in production.

---

## 📦 Step 2 — Deploy via GitHub (Recommended)

### 2a. Connect your repository

1. In **hPanel**, go to **Hosting** → **Manage** → **Node.js**
2. Click **Create project** or select your project
3. Choose **GitHub** as deployment method
4. Authorize Hostinger to access your repository
5. Select your repository and branch (e.g., `main`)

### 2b. Configure the Node.js project

In the Node.js setup wizard:

| Setting | Value |
|---|---|
| **Entry point** | `backend/src/server.js` |
| **Application mode** | `Production` |
| **Build command** | `cd frontend && npm install && npm run build` |
| **Start command** | `node backend/src/server.js` |

### 2c. Set environment variables

Add all the variables from **Step 1** in the hPanel Environment Variables section.

### 2d. Deploy

Click **Deploy** and wait for the build to complete.

---

## 📁 Step 3 — Deploy via ZIP Upload (Alternative)

If you prefer manual upload:

### 3a. Build the frontend locally

```bash
cd frontend
npm install && npm run build
```

This creates the `dist/` folder at the project root with the production build.

### 3b. Upload to Hostinger

1. Compress your project (excluding `node_modules/` and `.git/`)
2. In hPanel, go to **File Manager** → Upload → Extract
3. Or use **FTP/SFTP** to upload files to your hosting directory

### 3c. Install dependencies & configure

SSH into your hosting (or use hPanel terminal):

```bash
cd backend
npm install --production

cd ../frontend
npm install && npm run build
```

### 3d. Start the application

```bash
cd backend
NODE_ENV=production node src/server.js
```

Or use PM2 (if available on your plan):

```bash
cd backend
pm2 start ecosystem.config.js
```

---

## 🗄 Step 4 — Set Up MongoDB Atlas

### 4a. Create a cluster

1. Go to [MongoDB Atlas](https://cloud.mongodb.com)
2. Create a free **M0 cluster** (or higher)
3. Choose a cloud provider and region close to Hostinger's servers

### 4b. Configure network access

1. Go to **Network Access** → **Add IP Address**
2. Click **Allow Access from Anywhere** (`0.0.0.0/0`) for simplicity
3. Or add Hostinger's server IP if known

### 4c. Create a database user

1. Go to **Database Access** → **Add New Database User**
2. Set username and password (save these!)
3. Set role to **Read and Write to any database**

### 4d. Get connection string

1. Click **Connect** → **Connect your application**
2. Copy the connection string
3. Replace `<password>` with your user's password
4. Set this as `MONGODB_URI` in Hostinger's environment variables

---

## 🔄 Step 5 — Persistent Storage for Uploads

File uploads (product images, proof files) are stored in `backend/uploads/`. 

**On Hostinger's managed hosting**, your project files persist between restarts, so uploads should persist. However, **redeployments from GitHub may overwrite uploads**.

### Recommended: Use external storage

For production, consider using **Cloudinary** or **AWS S3** for uploads. Currently, the app saves to the local filesystem via Multer. To switch to cloud storage, you would need to modify the upload middleware (`backend/src/middleware/upload.js`).

---

## ✅ Step 6 — Verify Deployment

1. Visit your domain (e.g., `https://yourdomain.com`)
2. Confirm the frontend loads properly
3. Test the health endpoint: `https://yourdomain.com/api/health`
4. Try logging in with admin credentials
5. Test file uploads on the products page

---

## 🔍 Troubleshooting

### Backend shows "MongoDB connection failed"
- Verify `MONGODB_URI` is set correctly in hPanel
- Check Atlas **Network Access** — ensure your IP or `0.0.0.0/0` is whitelisted
- Ensure the Atlas cluster is not paused (free tiers auto-pause after 60 days)

### "Port already in use" error
- Another process may be using the port
- Try a different `PORT` value in environment variables
- Ensure you're not running the app twice

### Frontend shows blank page or 404
- Make sure the frontend build succeeded
- The frontend `dist/` folder must exist at the project root (`dist/`)
- The backend serves static files if the `dist/` folder exists

### CORS errors in browser console
- Ensure `CLIENT_URL` is set to your exact production domain (with `https://`)
- Restart the Node.js application after changing env vars

### PM2 not available
- Hostinger's managed Node.js hosting may use its own process manager
- Use `node src/server.js` as the start command instead

---

## 📚 Additional Resources

- [Hostinger Node.js Hosting Docs](https://support.hostinger.com/en/articles/1586487-how-to-set-up-a-node-js-application-on-hosting)
- [MongoDB Atlas Documentation](https://www.mongodb.com/docs/atlas/)
- [Razorpay Integration Guide](https://razorpay.com/docs/)

---

## 🏁 Quick Reference

```bash
# Local development
cd frontend && npm run dev    # Terminal 1
cd backend && npm run dev     # Terminal 2

# Production build
cd frontend && npm run build

# Production start
cd backend && NODE_ENV=production node src/server.js

# Using PM2
cd backend && pm2 start ecosystem.config.js
```
