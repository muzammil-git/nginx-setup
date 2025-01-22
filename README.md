# Nginx Server Configuration Guide + File

This README explains the Nginx configuration for setting up a secure web server with SSL/TLS encryption and HTTP to HTTPS redirection for abc.com.

## Configuration Overview

The configuration consists of two server blocks:
1. Main HTTPS server (Port 443)
2. HTTP redirect server (Port 80)

## HTTPS Server Block

This server block handles secure HTTPS traffic:

```nginx
server {
    server_name 40.20.40.100 abc.com;
    
    # Proxy Configuration
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # SSL Configuration
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/abc.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/abc.com/privkey.pem;
}
```

### Key Components:
- Listens on port 443 (HTTPS)
- Serves content for both IP (40.20.40.100) and domain (abc.com)
- Proxies requests to a local service running on port 8000
- Uses Let's Encrypt SSL certificates

## HTTP Redirect Block

This server block handles HTTP traffic and redirects it to HTTPS:

```nginx
server {
    server_name 40.20.40.100 abc.com;
    listen 80;

    if ($host = abc.com) {
        return 301 https://$host$request_uri;
    }

    return 404;
}
```

### Key Components:
- Listens on port 80 (HTTP)
- Automatically redirects HTTP traffic to HTTPS
- Returns 404 for any unmatched requests

## SSL Certificate Information

The configuration uses Let's Encrypt SSL certificates:
- Certificate path: `/etc/letsencrypt/live/abc.com/fullchain.pem`
- Private key path: `/etc/letsencrypt/live/abc.com/privkey.pem`

## Proxy Settings

The configuration includes the following proxy settings for WebSocket support and proper header handling:
- `proxy_http_version 1.1`
- `proxy_set_header Upgrade $http_upgrade`
- `proxy_set_header Connection 'upgrade'`
- `proxy_set_header Host $host`
- `proxy_cache_bypass $http_upgrade`

## Notes

- This configuration is managed by Certbot for SSL certificate automation
- All HTTP traffic (port 80) is automatically redirected to HTTPS (port 443)
- The server proxies requests to a local service running on port 8000
- Both the domain name and IP address are configured to serve the content
