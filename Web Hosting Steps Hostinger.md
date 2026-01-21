# Complete Laravel Deployment Guide for Hostinger

## 1. Enable SSH Access in Hostinger

1. Log in to **Hostinger hPanel**
2. Navigate to **Advanced → SSH Access**
3. Click **Enable SSH**
4. Copy and save the following credentials:
   - SSH Host
   - Username
   - Port

---

## 2. Connect to Hostinger via SSH

Open your local terminal and execute:

```bash
ssh username@your-server-ip -p PORT
```

**Note:** Replace `username`, `your-server-ip`, and `PORT` with the credentials from Hostinger.

---

## 3. Navigate to Project Directory

⚠️ **Important:** Do NOT clone directly into `public_html`. Laravel requires a specific directory structure.

### Recommended File Structure:
```
/sitename/
├── public_html → (symlink to laravel_app/public)
└── laravel_app/ → (your Laravel project)
```

### Create Directory:
```bash
cd ~
mkdir laravel_app
cd laravel_app
```

---

## 4. Clone Your GitHub Repository

### Option A: Public Repository
```bash
git clone https://github.com/username/repo-name.git .
```

### Option B: Private Repository (Recommended – SSH Method)

**Step 1:** Generate SSH key on Hostinger:
```bash
ssh-keygen -t rsa -b 4096
```

**Step 2:** Copy the public key:
```bash
cat ~/.ssh/id_rsa.pub
```

**Step 3:** Add the key to GitHub:
- Go to **GitHub → Settings → SSH and GPG Keys**
- Click **New SSH Key**
- Paste the copied key

**Step 4:** Clone your repository:
```bash
git clone git@github.com:username/repo-name.git .
```

---

## 5. Install Composer Dependencies

```bash
composer install --no-dev --optimize-autoloader
```

### If Composer is Not Installed:
```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php composer.phar install
```

---

## 6. Create and Configure .env File

```bash
cp .env.example .env
nano .env
```

### Update the following values:

**Application Settings:**
```env
APP_NAME=Laravel
APP_ENV=production
APP_DEBUG=false
APP_URL=https://yourdomain.com
```

**Database Configuration:**
```env
DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=dbname
DB_USERNAME=dbuser
DB_PASSWORD=dbpassword
```

**Save and Exit:** `CTRL + X` → `Y` → `Enter`

---

## 7. Generate Application Key

```bash
php artisan key:generate
```

---

## 8. Link Laravel Public Folder to public_html

### Option A: Symlink (Recommended)

**Remove the default public_html folder:**
```bash
rm -rf public_html
```

**Create a symbolic link:**
```bash
ln -s laravel_app/public public_html
```

**Verify the symlink:**
```bash
ls -l
```

Expected output:
```
public_html -> laravel_app/public
```

### Verify .htaccess Exists:
```bash
ls laravel_app/public/.htaccess
```

⚠️ **If .htaccess is missing:** Copy it from Laravel's GitHub repo or ensure it exists in your repository.

---

## 9. Fix File Permissions

⚠️ **Critical:** Incorrect permissions will cause 500 errors.

```bash
chmod 755 ~/domains/yourdomain.com
chmod -R 755 laravel_app/public
chmod -R 775 laravel_app/storage
chmod -R 775 laravel_app/bootstrap/cache
```

---

## 10. Clear Laravel Cache

```bash
cd laravel_app
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
```

---

## 11. Run Database Migrations

```bash
php artisan migrate --force
```

⚠️ **Note:** The `--force` flag skips confirmation in production.

---

## 12. Create Storage Symlink

```bash
php artisan storage:link
```

This makes uploaded files/images accessible publicly.

---

## 13. Configure PHP Version and Extensions

1. Log in to **Hostinger hPanel**
2. Navigate to **Advanced → PHP Configuration**
3. Select **PHP 8.1** or **PHP 8.2** (recommended for modern Laravel)
4. Enable the following extensions:
   - ✓ mbstring
   - ✓ pdo
   - ✓ openssl
   - ✓ tokenizer
   - ✓ xml
   - ✓ fileinfo
   - ✓ curl (recommended)
   - ✓ gd (if using image manipulation)

---

## 14. Enable SSL Certificate

1. Log in to **Hostinger hPanel**
2. Navigate to **Security → SSL**
3. Click **Enable Free SSL** (provided by Hostinger)
4. Wait for activation (usually 5-15 minutes)

### Update .env:
```env
APP_URL=https://yourdomain.com
```

---

## 15. Optimize for Production

```bash
php artisan optimize
```

This generates an optimized configuration cache and improves performance.

---

## 16. Test Your Deployment

1. Open your browser and visit `https://yourdomain.com`
2. If you see your Laravel app, deployment was successful ✅
3. Check the logs if there are issues:

```bash
tail -f laravel_app/storage/logs/laravel.log
```

---

## Future Deployments (Update Process)

When deploying updates from GitHub:

```bash
cd ~/laravel_app
git pull origin main
composer install --no-dev
php artisan migrate --force
php artisan optimize
```

---

## Troubleshooting Common Issues

### 500 Error (Internal Server Error)

**Check these in order:**

1. **Verify .env file:**
   ```bash
   cat .env | grep APP_DEBUG
   # Should show: APP_DEBUG=false
   ```

2. **Check permissions:**
   ```bash
   ls -la laravel_app/storage/
   # Should show 775 permissions
   ```

3. **Verify PHP version:**
   - Confirm PHP 8.1+ is enabled in hPanel

4. **Check Laravel logs:**
   ```bash
   tail -f laravel_app/storage/logs/laravel.log
   ```

5. **Verify database connection:**
   ```bash
   php artisan migrate --force
   ```

### Storage/Logs Not Writable

```bash
chmod -R 775 ~/laravel_app/storage
chmod -R 775 ~/laravel_app/bootstrap/cache
```

### Assets/Images Not Loading

```bash
php artisan storage:link
php artisan config:clear
php artisan view:clear
```

### Git Pull Fails

**If you get "Permission Denied" errors:**

1. Verify SSH key is added to GitHub:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

2. Test SSH connection:
   ```bash
   ssh -T git@github.com
   ```

### Composer Installation Fails

**Try updating Composer:**
```bash
composer self-update
composer install --no-dev --optimize-autoloader
```

---

## Advanced Topics (Optional)

You can implement these for enhanced workflow:

- **🔁 Auto Deployment:** GitHub Actions workflow for automatic deployments
- **🌐 Multiple Environments:** Separate staging and production servers
- **⏱️ Cron Jobs:** Laravel Scheduler setup for background tasks
- **🔐 Environment Secrets:** Secure handling of API keys and sensitive data
- **📊 Monitoring:** Application performance monitoring and error tracking

---

## Quick Reference Commands

```bash
# SSH into server
ssh username@server-ip -p PORT

# Navigate to project
cd ~/laravel_app

# Pull latest code
git pull origin main

# Install dependencies
composer install --no-dev

# Run migrations
php artisan migrate --force

# Clear cache
php artisan config:clear && php artisan cache:clear

# Optimize app
php artisan optimize

# View logs
tail -f storage/logs/laravel.log

# Check permissions
ls -la storage/
```

---

## ✅ Deployment Checklist

- [ ] SSH access enabled and credentials saved
- [ ] SSH key generated and added to GitHub
- [ ] Repository cloned successfully
- [ ] .env file created and configured
- [ ] Composer dependencies installed
- [ ] Application key generated
- [ ] Database migrations completed
- [ ] File permissions set correctly
- [ ] Storage symlink created
- [ ] Public folder symlinked
- [ ] PHP 8.1+ selected with required extensions
- [ ] SSL certificate enabled
- [ ] Application accessible at HTTPS
- [ ] Storage/Logs permissions verified
- [ ] Production cache cleared and optimized

---

**🚀 Your Laravel application is now live on Hostinger!**

For additional support or custom configurations, refer to:
- [Laravel Official Documentation](https://laravel.com/docs)
- [Hostinger Support](https://support.hostinger.com)
- [GitHub Documentation](https://docs.github.com)
