# ISP Backend Deployment Guide

## Overview
This guide covers deploying your Django backend to Render and setting up a Planet Scale MySQL database.

---

## Step 1: Prepare Your Repository

### 1.1 Commit Your Changes
```bash
git add .
git commit -m "Add deployment files for Render and Planet Scale"
git push origin main
```

### 1.2 Verify Key Files Are Present
- ✅ `Procfile` - Web server startup configuration
- ✅ `render.yaml` - Render infrastructure configuration
- ✅ `requirements.txt` - Python dependencies with gunicorn
- ✅ `manage.py` - Django management script

---

## Step 2: Set Up Planet Scale Database

### 2.1 Create Planet Scale Account
1. Go to https://planetscale.com
2. Sign up with GitHub account
3. Click "Create a new database"

### 2.2 Create Database
1. Database name: `isp-automation-system`
2. Region: Choose closest to Render (typically US regions)
3. Click "Create database"

### 2.3 Get Connection Details
1. Click your database name
2. Click "Connect" button
3. Select "Django" from the dropdown
4. Copy the connection string

The connection string will look like:
```
mysql://[USERNAME]:[PASSWORD]@[HOST]/isp_automation_system
```

### 2.4 Extract Connection Details
From the connection string, extract:
- **DB_HOST**: The host portion (e.g., `xxx.us-east-2.psdb.cloud`)
- **DB_USER**: Username (e.g., `xxxxxxxx`)
- **DB_PASSWORD**: Password
- **DB_NAME**: `isp_automation_system`
- **DB_PORT**: `3306`

---

## Step 3: Deploy Backend to Render

### 3.1 Connect GitHub Repository
1. Go to https://render.com
2. Sign up or log in with GitHub
3. Click "New +" → "Web Service"
4. Select "Connect a repository"
5. Find your ISP repository and click "Connect"

### 3.2 Configure Web Service
1. **Name**: `isp-backend` (or your preferred name)
2. **Region**: Choose same region as Planet Scale
3. **Branch**: `main`
4. **Runtime**: `Python 3`
5. **Build Command**: `pip install -r requirements.txt`
6. **Start Command**: `gunicorn netflow_backend.wsgi:application --bind 0.0.0.0:$PORT`

### 3.3 Set Environment Variables
Click "Advanced" → "Add Environment Variable" for each:

**Critical Variables:**
```
DJANGO_SECRET_KEY = [Generate a random secure key]
DJANGO_DEBUG = 0
DB_HOST = [From Planet Scale connection string]
DB_USER = [From Planet Scale connection string]
DB_PASSWORD = [From Planet Scale connection string]
DB_NAME = isp_automation_system
DB_PORT = 3306
```

**Optional but Recommended:**
```
GEMINI_API_KEY = [Your Gemini API key if using AI features]
EMAIL_HOST_USER = [Gmail address if sending emails]
EMAIL_HOST_PASSWORD = [Gmail app password]
```

### 3.4 Generate DJANGO_SECRET_KEY
Run this in your terminal:
```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

---

## Step 4: Run Initial Migrations

### 4.1 SSH into Render Service
1. In Render dashboard, go to your web service
2. Click "Shell" tab (top menu)
3. Run migrations:
```bash
python manage.py migrate
```

### 4.2 Create Superuser (Optional)
```bash
python manage.py createsuperuser
```

### 4.3 Collect Static Files (Optional)
```bash
python manage.py collectstatic --noinput
```

---

## Step 5: Update Frontend Environment Variables

After deployment, update your Vercel frontend with the backend URL:

### 5.1 Get Backend URL
Your backend URL will be displayed on the Render dashboard:
- Example: `https://isp-backend.onrender.com`

### 5.2 Update Vercel Frontend
1. Go to Vercel dashboard → Your project
2. Settings → Environment Variables
3. Add or update:
```
VITE_API_URL = https://isp-backend.onrender.com
```

### 5.3 Update CORS Settings
In `netflow_backend/settings.py`, update:
```python
CORS_ALLOWED_ORIGINS = [
    "https://your-frontend.vercel.app",  # Your actual Vercel URL
    "http://localhost:3000",
]
```

Then commit and push to trigger a redeploy.

---

## Step 6: Update Frontend CORS Configuration

### 6.1 Check Frontend API Calls
Make sure all API calls in your frontend use the environment variable:

Example (React/TypeScript):
```typescript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

export const fetchData = async () => {
  const response = await fetch(`${API_URL}/api/endpoint/`, {
    headers: {
      'Authorization': `Token ${token}`,
    }
  });
  return response.json();
};
```

---

## Troubleshooting

### Issue: "502 Bad Gateway" on Render
**Solution:**
1. Check Render logs: Dashboard → Logs tab
2. Verify all environment variables are set correctly
3. Run migrations manually via Shell: `python manage.py migrate`
4. Restart the service

### Issue: "Connection refused" to database
**Solution:**
1. Verify Planet Scale database is active
2. Check DB_HOST, DB_USER, DB_PASSWORD in Render
3. Ensure database name is correct: `isp_automation_system`
4. Test connection locally with the same credentials

### Issue: CORS errors in frontend
**Solution:**
1. Update `CORS_ALLOWED_ORIGINS` in `settings.py` with your Vercel URL
2. Redeploy backend by pushing to GitHub
3. Check browser console for exact origin being blocked
4. Ensure frontend has correct API_URL environment variable

### Issue: Static files not loading
**Solution:**
```bash
# SSH into Render and run:
python manage.py collectstatic --noinput
```

### Check Backend Logs
```bash
# In Render Dashboard → Logs tab
# Look for any error messages
```

---

## Database Backups

### Planet Scale Automatic Backups
- Planet Scale automatically backs up your data
- Backups are retained for 30 days on free plan
- Access backups in Planet Scale dashboard

### Manual Backup
```bash
# SSH into Render and export database:
mysqldump -h $DB_HOST -u $DB_USER -p$DB_PASSWORD $DB_NAME > backup.sql
```

---

## Deployment Checklist

- [ ] Repository pushed to GitHub
- [ ] Procfile created and contains correct commands
- [ ] render.yaml configured with web service
- [ ] requirements.txt includes gunicorn, mysqlclient, python-dotenv
- [ ] settings.py updated for production (ALLOWED_HOSTS, CORS, DEBUG=0)
- [ ] Planet Scale database created and credentials obtained
- [ ] Render service connected to GitHub repository
- [ ] All environment variables set in Render dashboard
- [ ] Database migrations run successfully
- [ ] Frontend environment variable updated with backend URL
- [ ] CORS_ALLOWED_ORIGINS updated with Vercel frontend URL
- [ ] Backend responds to test requests
- [ ] Frontend can successfully call backend API

---

## Next Steps

1. Monitor Render logs for errors: https://render.com/dashboard
2. Test API endpoints from your frontend
3. Set up monitoring/alerts in Render
4. Configure custom domain if desired
5. Set up email service for production

---

## Support Resources

- **Render Docs**: https://render.com/docs
- **Planet Scale Docs**: https://planetscale.com/docs
- **Django Deployment**: https://docs.djangoproject.com/en/5.2/howto/deployment/
- **gunicorn Docs**: https://docs.gunicorn.org/

