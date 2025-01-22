# Nginx Server Configuration Guide for AWS EC2

This README explains how to configure Nginx on an AWS EC2 instance for setting up a secure web server with SSL/TLS encryption and HTTP to HTTPS redirection.

## Deployment Commands

1. First, ensure you have the necessary permissions:
```bash
sudo su
```

2. Navigate to the Nginx sites-enabled directory:
```bash
cd /etc/nginx/sites-enabled/
```

3. Create a new configuration file (or edit default if it exists):
```bash
sudo nano default
# Or create a new file:
# sudo nano abc.conf
```

4. Copy and paste the following configuration (replace abc.com with your domain):

```nginx
server {
    server_name 50.20.40.100 abc.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/abc.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/abc.com/privkey.pem;
}

server {
    if ($host = abc.com) {
        return 301 https://$host$request_uri;
    }

    server_name 50.20.40.100 abc.com;
    listen 80;
    return 404;
}
```

5. Save and exit nano:
```bash
Ctrl + X, then Y, then Enter
```

6. If you created a new configuration file, create a symbolic link in sites-enabled (skip if editing default):
```bash
sudo ln -s /etc/nginx/sites-available/abc.conf /etc/nginx/sites-enabled/
```

7. Remove the default symbolic link if it exists and you're not using it:
```bash
sudo rm /etc/nginx/sites-enabled/default
```

8. Test the Nginx configuration:
```bash
sudo nginx -t
```

9. If the test is successful, reload Nginx:
```bash
sudo systemctl reload nginx
```

10. If Nginx isn't running, start it:
```bash
sudo systemctl start nginx
```

11. Enable Nginx to start on boot:
```bash
sudo systemctl enable nginx
```

## SSL Certificate Setup with Certbot

If you haven't installed SSL certificates yet:

1. Install Certbot:
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

2. Obtain SSL certificate:
```bash
sudo certbot --nginx -d abc.com
```

## Troubleshooting Commands

Check Nginx status:
```bash
sudo systemctl status nginx
```

View Nginx error logs:
```bash
sudo tail -f /var/log/nginx/error.log
```

View Nginx access logs:
```bash
sudo tail -f /var/log/nginx/access.log
```

Check Nginx syntax:
```bash
sudo nginx -t
```

## Configuration Overview

The configuration consists of two server blocks:
1. Main HTTPS server (Port 443)
2. HTTP redirect server (Port 80)

### HTTPS Server Block
This server block handles secure HTTPS traffic and includes:
- SSL certificate configuration
- Proxy settings for your application
- WebSocket support

### HTTP Redirect Block
This server block:
- Redirects all HTTP traffic to HTTPS
- Returns 404 for any unmatched requests

## Security Notes

1. Ensure your EC2 security group allows:
   - Port 80 (HTTP)
   - Port 443 (HTTPS)
   - Your application port (8000 in this case)

2. SSL certificates are managed by Certbot and will auto-renew

3. Keep your EC2 instance updated:
```bash
sudo apt update
sudo apt upgrade
```

## Maintenance Commands

Restart Nginx:
```bash
sudo systemctl restart nginx
```

Stop Nginx:
```bash
sudo systemctl stop nginx
```

Renew SSL certificates manually:
```bash
sudo certbot renew
```

## Common Issues and Solutions

1. If you see a permissions error:
```bash
sudo chown -R www-data:www-data /etc/nginx/sites-enabled
sudo chmod 644 /etc/nginx/sites-enabled/default
```

2. If Nginx fails to start due to port conflicts:
```bash
sudo netstat -tulpn | grep :80
sudo netstat -tulpn | grep :443
```

3. If SSL certificate paths are incorrect:
```bash
sudo ls -la /etc/letsencrypt/live/abc.com/
```
