# Aiven MySQL Setup for ISP Backend

Quick reference for setting up Aiven MySQL database for Render deployment.

## 1️⃣ Create Aiven MySQL Service

```
1. Go to https://aiven.io → Sign up
2. Create new project
3. Click "Create Service" → Select "MySQL"
4. Configuration:
   - Service name: isp-mysql
   - Cloud: AWS (or your preference)
   - Region: us-east-1 or nearest to you
   - Plan: Startup-4 (free, $23/mo if kept)
5. Wait ~5-10 minutes for initialization
```

## 2️⃣ Get Connection Credentials

**In Aiven Dashboard:**
```
1. Click your service name (isp-mysql)
2. Go to "Connection" tab
3. Copy these values:
   - Host
   - Port (usually 3306)
   - User (avnadmin)
   - Password
   - Database (defaultdb or create isp_automation_system)
```

**Example connection details:**
```
Host: mysql-xxxxx.aivencloud.com
Port: 3306
User: avnadmin
Password: your_secure_password
Database: defaultdb
```

## 3️⃣ Create Database (if needed)

**In Aiven Dashboard:**
```
1. Service → "Databases"
2. Click "Create database"
3. Name: isp_automation_system
4. Click Create
```

Or use `defaultdb` and create tables there.

## 4️⃣ Allow Render IP (Important!)

**In Aiven Dashboard:**
```
1. Service → "Networking"
2. Under "IP Filter/Firewall":
   - Add Render service IP: 0.0.0.0/0 (for testing)
   - Or find Render's specific IPs and add them
3. Save changes
```

⚠️ **Note**: Using `0.0.0.0/0` allows all IPs. For production, restrict to Render's IP only.

## 5️⃣ Environment Variables for Render

Set these in Render dashboard:

```
DB_HOST = mysql-xxxxx.aivencloud.com
DB_PORT = 3306
DB_USER = avnadmin
DB_PASSWORD = your_secure_password
DB_NAME = defaultdb (or isp_automation_system)
DJANGO_DEBUG = 0
DJANGO_SECRET_KEY = [generate random key]
```

## 6️⃣ Test Connection

**From your local machine:**
```bash
mysql -h mysql-xxxxx.aivencloud.com -u avnadmin -p
# Enter password when prompted
# Should see: mysql>
```

## 🔧 Common Aiven Tasks

### Restart Service
```
Aiven Dashboard → Service → "Actions" → "Restart"
```

### View Logs
```
Aiven Dashboard → Service → "Logs"
```

### Scale Up Plan
```
Aiven Dashboard → Service → "Plan" → Select new plan
```

### Backup and Restore
```
Aiven Dashboard → Service → "Backups"
- Automatic daily backups
- Can restore to new service
```

### Reset Admin Password
```
Aiven Dashboard → Service → "Database"
- Users section → Edit avnadmin password
```

## 📊 Pricing

- **Startup-4**: ~$23/month (recommended for production)
- **Startup-2**: ~$11/month (development)
- **Free tier**: Not available for MySQL (only 30-day trial)

## ✅ Verification Checklist

- [ ] Aiven MySQL service created and running
- [ ] Service status is green in Aiven dashboard
- [ ] Connection credentials copied
- [ ] Database created: `isp_automation_system`
- [ ] Firewall allows Render service (0.0.0.0/0 or specific IP)
- [ ] Local connection test successful
- [ ] Render environment variables set
- [ ] Migrations run successfully

## 🆘 Troubleshooting

| Issue | Solution |
|-------|----------|
| Can't connect from Render | Check firewall in "Networking" tab |
| "Access denied for user" | Verify DB_USER and DB_PASSWORD are correct |
| Service not starting | Check service status, restart if needed |
| Slow connection | May need to upgrade from Startup-2 plan |
| Need to restore backup | Go to "Backups" tab in Aiven |

## 📞 Support

- **Aiven Docs**: https://docs.aiven.io/
- **Aiven MySQL Docs**: https://docs.aiven.io/docs/products/mysql
- **Aiven Support**: https://console.aiven.io/support

