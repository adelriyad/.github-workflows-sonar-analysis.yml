# Production Deployment Guide (VPS)

## 1. Environment Setup
Review `.env.example` and create a production `.env` file.
**Critical**:
- `NODE_ENV=production`
- `NEXTAUTH_URL=https://alphadeon.com`
- `NEXTAUTH_SECRET`: Generate with `openssl rand -base64 32`
- `APPLE_PRIVATE_KEY`: Must effectively preserve newlines (replace `\n` if passed as single line string).
- `DATABASE_URL`: Production DB connection string.

## 2. Server Prerequisites
- Node.js 20+
- Nginx
- Certbot
- PM2 (`npm install -g pm2`)

## 3. Build & Start
```bash
# Install dependencies
npm ci --omit=dev

# Build application
npm run build

# Start with PM2
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

## 4. Nginx Configuration
Save to `/etc/nginx/sites-available/alphadeon` and link to `sites-enabled`.

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name alphadeon.com www.alphadeon.com;

    # Certbot will modify this for SSL redirect
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 5. SSL / HTTPS (Certbot)
Run Certbot to obtain certificates and auto-configure Nginx redirect.
```bash
sudo certbot --nginx -d alphadeon.com -d www.alphadeon.com
```

## 6. Verification
- **Health Check**: `curl http://localhost:3000`
- **SSL Check**: Visit https://alphadeon.com
- **Logs**: `pm2 logs alphadeon-honey`

## 7. Security Notes
- **Firewall**: Ensure UFW allows 'Nginx Full' and 'OpenSSH'.
- **Secrets**: Do not commit `.env`.
- **Admin**: Verify `/admin` redirects to login for non-admins.
