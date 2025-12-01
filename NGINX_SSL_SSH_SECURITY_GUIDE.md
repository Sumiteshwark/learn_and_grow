**Last Updated:** November 2025  
**Author:** SUMITESHWAR KUMAR

---

# **Complete nginx, HTTPS, SSH, certbot and openSSL Security Guide**

---

## 📑 Table of Contents
- [0. SSH Key Generation with ssh-keygen](#0-ssh-key-generation-with-ssh-keygen)
- [1. What is HTTPS?](#1-what-is-https)
- [2. What nginx needs for HTTPS](#2-what-nginx-needs-for-https)
- [3. Certbot — Full HTTPS Guide](#3-certbot-full-https-guide)
- [4. OpenSSL — Complete HTTPS Guide](#4-openssl-complete-https-guide)
- [5. nginx Configurations](#5-nginx-configurations)
- [6. Certbot vs OpenSSL vs ssh-keygen Comparison](#6-certbot-vs-openssl-vs-ssh-keygen-comparison)
- [7. When to Use Each Tool](#7-when-to-use-each-tool)
- [8. Security Best Practices](#8-security-best-practices)
- [9. Troubleshooting Guide](#9-troubleshooting-guide)
- [10. Commands Reference Sheet](#10-commands-reference-sheet)
- [11. Summary](#11-summary)

---

## 0. SSH Key Generation with ssh-keygen

SSH keys provide secure authentication for SSH connections and can also be used for other cryptographic purposes.

### 0.1 Generate SSH Key Pair

**RSA Key (Default, 2048-bit):**

    ssh-keygen -t rsa -b 2048 -C "your-email@example.com"

**RSA Key (4096-bit, more secure):**

    ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

**ECDSA Key (Faster, smaller):**

    ssh-keygen -t ecdsa -b 256 -C "your-email@example.com"

**Ed25519 Key (Most secure, modern, recommended):**

    ssh-keygen -t ed25519 -C "your-email@example.com"

### 0.2 Advanced ssh-keygen Options

**Specify key file location:**

    ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_custom_key

**Generate without passphrase (less secure):**

    ssh-keygen -t rsa -b 2048 -N "" -f ~/.ssh/id_rsa

**Generate with custom comment:**

    ssh-keygen -t rsa -C "server-admin@company.com"

### 0.3 SSH Key Files Created

    ~/.ssh/id_rsa          # Private key (keep secret!)
    ~/.ssh/id_rsa.pub      # Public key (share with servers)
    ~/.ssh/known_hosts     # Known host keys
    ~/.ssh/authorized_keys # Authorized public keys

### 0.4 Copy Public Key to Server

**Using ssh-copy-id (recommended):**

    ssh-copy-id -i ~/.ssh/id_rsa.pub user@server.com

**Manual method:**

    cat ~/.ssh/id_rsa.pub | ssh user@server.com "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

### 0.5 SSH Key Management

**List SSH keys:**

    ls -la ~/.ssh/

**Change passphrase:**

    ssh-keygen -p -f ~/.ssh/id_rsa

**View public key fingerprint:**

    ssh-keygen -l -f ~/.ssh/id_rsa.pub

**Convert key formats:**

    ssh-keygen -p -N "" -m PEM -f ~/.ssh/id_rsa

### 0.6 SSH Config File (~/.ssh/config)

    Host myserver
        HostName server.example.com
        User myuser
        IdentityFile ~/.ssh/id_rsa
        Port 22

    Host github
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_ed25519

### 0.7 SSH Agent (Manage multiple keys)

**Start SSH agent:**

    eval "$(ssh-agent -s)"

**Add key to agent:**

    ssh-add ~/.ssh/id_rsa

**List loaded keys:**

    ssh-add -l

**Remove key from agent:**

    ssh-add -d ~/.ssh/id_rsa

### 0.8 Troubleshooting SSH Keys

**Test SSH connection:**

    ssh -T git@github.com

**Debug SSH connection:**

    ssh -v user@server.com

**Check SSH key permissions:**

    chmod 600 ~/.ssh/id_rsa
    chmod 644 ~/.ssh/id_rsa.pub
    chmod 700 ~/.ssh

---

## 1. What is HTTPS?

HTTPS (Hypertext Transfer Protocol Secure) provides:
- **Encryption** using TLS (Transport Layer Security)
- **Server identity validation** through CA-signed certificates
- **Data integrity** preventing tampering

### Requirements:
- A **private key**
- A **certificate** (X.509) containing the **public key**
- Certificate must be **trusted** (CA-signed)

---

## 2. What nginx needs for HTTPS

nginx requires **two essential files** to enable HTTPS (SSL/TLS encryption) on your web server:

### 1. `ssl_certificate` directive
```nginx
ssl_certificate /path/to/fullchain.pem;
```

This points to your **SSL certificate file**, which contains:
- Your **public key** (used for encryption)
- Your domain information
- The **Certificate Authority (CA) signature** that proves the certificate is legitimate and trusted

The certificate file is what gets sent to browsers during the SSL handshake. There are two common naming conventions:
- `cert.pem` - Contains only your domain's certificate
- `fullchain.pem` - Contains your certificate + the intermediate CA certificates (recommended)

### 2. `ssl_certificate_key` directive
```nginx
ssl_certificate_key /path/to/privkey.pem;
```

This points to your **private key file**, which contains:
- Your **private key** (used for decryption)
- **Must be kept absolutely secure** - if compromised, your SSL security is broken

### Why both files are required

SSL/TLS encryption works using **asymmetric cryptography** (public-key cryptography):

1. **During SSL handshake**: The server sends the public certificate to the browser
2. **Browser encrypts data**: Using the public key from the certificate
3. **Server decrypts data**: Using the private key stored on the server

Without both files working together, HTTPS cannot establish a secure connection. The public certificate enables browsers to encrypt data, while the private key enables your server to decrypt it.

### Security considerations

- **Private key protection**: The `privkey.pem` file should have restrictive permissions (typically 600) and be owned by the nginx user
- **Certificate updates**: Both files need to be updated when you renew your SSL certificate
- **Backup security**: The private key should be securely backed up (encrypted) in case of server failure

This two-file configuration is the minimum required for basic HTTPS functionality. Additional SSL directives can be added for enhanced security (like SSL protocols, ciphers, etc.).

---

## 3. Certbot Full HTTPS Guide

### 3.1 Install Certbot + nginx plugin

**Ubuntu/Debian:**

    sudo apt update
    sudo apt install certbot python3-certbot-nginx

**CentOS/RHEL:**

    sudo dnf install certbot python3-certbot-nginx

### 3.2 Prepare nginx

Create HTTP server block first (Certbot needs this for validation):

    server {
        listen 80;
        server_name example.com www.example.com;
        root /var/www/html;
    }

Test and reload:

    sudo nginx -t
    sudo systemctl reload nginx

### 3.3 Generate and Install HTTPS Certificates

**Single domain:**

    sudo certbot --nginx -d example.com

**Domain + www:**

    sudo certbot --nginx -d example.com -d www.example.com

**Fully automated:**

    sudo certbot --nginx -d example.com --agree-tos --email admin@example.com --redirect --non-interactive

### 3.4 Certificate Locations

    /etc/letsencrypt/live/example.com/fullchain.pem
    /etc/letsencrypt/live/example.com/privkey.pem

### 3.5 Auto Renewal

Check:

    sudo systemctl status certbot.timer

Test renewal:

    sudo certbot renew --dry-run

---

## 4. OpenSSL Complete HTTPS Guide

### 4.1 Generate private key

    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out privkey.pem

### 4.2 Generate CSR

    openssl req -new -key privkey.pem -out domain.csr -subj "/CN=example.com/O=MyOrg/C=IN"

Send `domain.csr` to a CA.

### 4.3 Self-signed certificate (testing only)

    openssl req -new -x509 -key privkey.pem -out cert_selfsigned.pem -days 365 -subj "/CN=example.com"

---

## 5. nginx Configurations

### 5.1 HTTPS using Certbot certificates

    server {
        listen 80;
        server_name example.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name example.com;

        ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_session_cache shared:SSL:10m;
        ssl_prefer_server_ciphers on;

        root /var/www/html;
    }

### 5.2 HTTPS using OpenSSL certificates

    server {
        listen 443 ssl http2;
        server_name example.com;

        ssl_certificate     /etc/ssl/certs/cert.pem;
        ssl_certificate_key /etc/ssl/private/privkey.pem;

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;

        root /var/www/html;
    }

### 5.3 Advanced SSL/TLS Configuration

#### Modern Cipher Suites

    server {
        listen 443 ssl http2;
        server_name example.com;

        ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

        # TLS versions
        ssl_protocols TLSv1.2 TLSv1.3;

        # Modern cipher suites
        ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384;
        ssl_prefer_server_ciphers off;

        # Session caching
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;
        ssl_session_tickets off;

        # HSTS
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;

        root /var/www/html;
    }

#### OCSP Stapling

    server {
        listen 443 ssl http2;
        server_name example.com;

        ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

        # OCSP stapling
        ssl_stapling on;
        ssl_stapling_verify on;
        ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

        # DNS resolver for OCSP
        resolver 8.8.8.8 8.8.4.4 valid=300s;
        resolver_timeout 5s;
    }

### 5.4 nginx as Reverse Proxy

#### Basic Reverse Proxy

    server {
        listen 80;
        server_name api.example.com;

        location / {
            proxy_pass http://localhost:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

#### Load Balancing

    upstream backend {
        least_conn;
        server backend1.example.com:8080 weight=3;
        server backend2.example.com:8080 weight=2;
        server backend3.example.com:8080 weight=1;
        server backend4.example.com:8080 backup;
    }

    server {
        listen 80;
        server_name app.example.com;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # Health checks
            health_check;
        }
    }

#### WebSocket Proxy

    server {
        listen 80;
        server_name ws.example.com;

        location /ws {
            proxy_pass http://localhost:8080;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

### 5.5 Security Features

#### Rate Limiting

    http {
        # Define rate limit zones
        limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
        limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
        limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

        server {
            listen 80;
            server_name api.example.com;

            # Apply rate limiting
            location /api/ {
                limit_req zone=api burst=20 nodelay;
                limit_conn conn_limit 10;

                proxy_pass http://backend;
            }

            # Login endpoint protection
            location /login {
                limit_req zone=login burst=5 nodelay;
                limit_req_status 429;

                proxy_pass http://auth_service;
            }
        }
    }

#### DDoS Protection

    http {
        # Connection limiting
        limit_conn_zone $binary_remote_addr zone=ddos:10m;
        limit_req_zone $binary_remote_addr zone=ddos_req:10m rate=30r/s;

        server {
            listen 80;
            server_name example.com;

            # DDoS protection
            limit_conn ddos 10;
            limit_req zone=ddos_req burst=50 nodelay;

            # Block suspicious user agents
            if ($http_user_agent ~* "badbot|scanner") {
                return 403;
            }

            # Block specific IPs
            deny 192.168.1.100;
            deny 10.0.0.0/8;
            allow all;

            location / {
                root /var/www/html;
            }
        }
    }

### 5.6 Performance Optimization

#### Caching Configuration

    http {
        # Proxy cache
        proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;

        server {
            listen 80;
            server_name example.com;

            location /static/ {
                proxy_pass http://backend;
                proxy_cache my_cache;
                proxy_cache_valid 200 302 10m;
                proxy_cache_valid 404 1m;
                proxy_cache_key "$scheme$request_method$host$request_uri";

                # Cache bypass for authenticated users
                proxy_cache_bypass $http_authorization;
            }

            location /api/ {
                proxy_pass http://api_backend;
                # No caching for API calls
                proxy_no_cache $http_authorization;
            }
        }
    }

#### Compression

    http {
        # Enable gzip compression
        gzip on;
        gzip_vary on;
        gzip_min_length 1024;
        gzip_types
            text/plain
            text/css
            text/xml
            text/javascript
            application/json
            application/javascript
            application/xml+rss
            application/atom+xml
            image/svg+xml;

        # Brotli compression (if available)
        brotli on;
        brotli_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    }

#### Static File Serving

    server {
        listen 80;
        server_name static.example.com;

        root /var/www/static;

        # Enable sendfile for better performance
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;

        # Cache static files
        location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Security for static files
        location ~ /\. {
            deny all;
        }
    }

### 5.7 nginx API Gateway Features

#### Request Routing

    server {
        listen 80;
        server_name api.example.com;

        # API versioning
        location /v1/users {
            proxy_pass http://user_service:8080;
        }

        location /v1/orders {
            proxy_pass http://order_service:8081;
        }

        location /v1/payments {
            proxy_pass http://payment_service:8082;
        }

        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
        }
    }

#### Authentication Gateway

    server {
        listen 80;
        server_name secure-api.example.com;

        location / {
            # Basic auth
            auth_basic "API Access";
            auth_basic_user_file /etc/nginx/.htpasswd;

            # JWT validation (requires ngx_http_auth_jwt_module)
            auth_jwt "api" token=$http_authorization;
            auth_jwt_key_file /etc/nginx/jwt_keys.json;

            proxy_pass http://api_backend;
        }
    }

---

## 6. Certbot vs OpenSSL vs ssh-keygen Comparison

| Feature | Certbot | OpenSSL | ssh-keygen |
|---------|---------|---------|------------|
| **Purpose** | HTTPS certificates | General cryptography | SSH authentication |
| **Certificate Type** | CA-signed (Let's Encrypt) | Self-signed or CSR | SSH key pairs |
| **Browser Trust** | Yes | No (unless CA-signed) | N/A (not for HTTPS) |
| **Automation** | Fully automated | Manual | Manual |
| **Renewal** | Automatic | Manual | N/A |
| **Difficulty** | Very easy | More complex | Easy |
| **Best Use** | Production HTTPS | Testing, internal networks | SSH access, Git |
| **Key Types** | RSA, ECDSA | RSA, ECDSA, Ed25519 | RSA, ECDSA, Ed25519 |
| **Integration** | nginx plugin | Manual config | SSH clients |
| **Security Level** | High (CA validation) | Variable | High (modern algorithms) |

---

## 7. When to Use Each Tool

### 7.1 When to Use Certbot

#### ✅ Use Certbot When:
- **Production websites** requiring trusted HTTPS certificates
- **First-time HTTPS setup** on web servers
- **Automated certificate management** is needed
- **nginx/Apache servers** with domain validation
- **Let's Encrypt certificates** are acceptable
- **Regular certificate renewal** is required

#### ❌ Don't Use Certbot When:
- **Internal/private networks** (no public domain)
- **Custom certificate authorities** are required
- **Enterprise CA integration** is needed
- **Wildcard certificates** for many subdomains (complex DNS setup)
- **Air-gapped environments** (no internet access)
- **Long-term certificates** (>90 days) are needed

#### 📊 Certbot Pros & Cons

**Pros:**
- ✅ **Free** and automated
- ✅ **Trusted certificates** (browser accepted)
- ✅ **Auto-renewal** prevents expiry
- ✅ **Easy setup** for beginners
- ✅ **nginx integration** built-in
- ✅ **Secure defaults** configured automatically

**Cons:**
- ❌ **Domain required** (no internal use)
- ❌ **Internet access** needed for validation
- ❌ **Rate limits** (Let's Encrypt limits)
- ❌ **No custom CAs** (only Let's Encrypt)
- ❌ **Short validity** (90 days)
- ❌ **Dependency on external service**

### 7.2 When to Use OpenSSL

#### ✅ Use OpenSSL When:
- **Testing/development** environments
- **Internal applications** with self-signed certificates
- **Custom Certificate Authorities** are needed
- **Enterprise CA integration** is required
- **Long-term certificates** are acceptable
- **Air-gapped/private networks** are used
- **Advanced cryptographic operations** are needed

#### ❌ Don't Use OpenSSL When:
- **Production public websites** need trusted certificates
- **Browser compatibility** is critical (self-signed not trusted)
- **Automation and renewal** are required
- **Beginners** need simple setup
- **Standard web certificates** are sufficient

#### 📊 OpenSSL Pros & Cons

**Pros:**
- ✅ **Flexible** and powerful
- ✅ **No external dependencies**
- ✅ **Custom CAs** and certificates
- ✅ **Advanced cryptography** options
- ✅ **Works offline** (air-gapped)
- ✅ **Long-term certificates** possible

**Cons:**
- ❌ **Complex** for beginners
- ❌ **Manual process** (error-prone)
- ❌ **Self-signed certificates** not trusted by browsers
- ❌ **No automation** (manual renewal)
- ❌ **Security depends on implementation**
- ❌ **More maintenance** required

### 7.3 When to Use ssh-keygen

#### ✅ Use ssh-keygen When:
- **SSH server access** needs password-less authentication
- **Git repositories** require secure authentication
- **Automated scripts** need SSH access
- **Multiple users** need server access
- **Key rotation** and management is needed
- **Secure file transfer** (SCP/SFTP) is required

#### ❌ Don't Use ssh-keygen When:
- **HTTPS certificates** are needed (use Certbot/OpenSSL)
- **Web browser trust** is required
- **Certificate Authority validation** is needed
- **Public web services** need encryption
- **Email encryption** (S/MIME) is needed
- **Code signing** certificates are required

#### 📊 ssh-keygen Pros & Cons

**Pros:**
- ✅ **Secure authentication** without passwords
- ✅ **Works with existing SSH infrastructure**
- ✅ **Multiple key types** (RSA, ECDSA, Ed25519)
- ✅ **Key management** and rotation possible
- ✅ **Agent support** for multiple keys
- ✅ **Standard for DevOps** workflows

**Cons:**
- ❌ **Not for HTTPS** (different purpose)
- ❌ **Manual key distribution** required
- ❌ **Key compromise** affects all authorized systems
- ❌ **No automatic renewal** system
- ❌ **SSH server configuration** required
- ❌ **Not browser-compatible**

### 7.4 Quick Decision Guide

| Scenario | Recommended Tool | Why |
|----------|------------------|-----|
| **Production Website** | Certbot | Trusted certificates, automation |
| **Development/Test** | OpenSSL | Self-signed, no domain needed |
| **Internal API** | OpenSSL | Custom CA, enterprise control |
| **SSH Server Access** | ssh-keygen | Secure authentication |
| **Git Authentication** | ssh-keygen | Standard for GitHub/GitLab |
| **Automated Scripts** | ssh-keygen | Password-less SSH access |
| **Enterprise CA** | OpenSSL | Full control over certificates |
| **Quick HTTPS Setup** | Certbot | One-command solution |
| **Air-gapped Network** | OpenSSL | No internet required |
| **Wildcard Certificates** | OpenSSL | Better for complex DNS |

### 7.5 Combined Usage Scenarios

#### Scenario 1: Full Production Setup

    # 1. SSH access to server
    ssh-keygen -t ed25519
    ssh-copy-id user@server

    # 2. HTTPS certificates
    certbot --nginx -d example.com

    # 3. Internal services (optional)
    openssl req -new -x509 -key privkey.pem -out cert.pem

#### Scenario 2: Development Environment

    # SSH keys for server access
    ssh-keygen -t rsa -b 2048

    # Self-signed certificates for testing
    openssl genpkey -algorithm RSA -out privkey.pem
    openssl req -new -x509 -key privkey.pem -out cert.pem

#### Scenario 3: Enterprise Environment

    # SSH keys for admin access
    ssh-keygen -t ecdsa -b 256

    # Enterprise CA certificates
    openssl genpkey -algorithm RSA -out privkey.pem
    openssl req -new -key privkey.pem -out domain.csr
    # Submit CSR to enterprise CA

---

## 8. Security Best Practices

- Use **TLS 1.2 and 1.3** only
- Use RSA 2048+ or ECDSA keys
- Set permissions: `chmod 600 privkey.pem`
- Never reuse SSH keys for TLS
- Always redirect HTTP → HTTPS
- Use strong SSH key types (Ed25519 recommended)
- Enable HSTS headers for HTTPS
- Implement rate limiting and DDoS protection
- Regular security audits and updates
- Monitor certificate expiry dates

---

## 9. Troubleshooting Guide

**Port 80 blocked:**

    sudo ufw allow 80
    sudo ufw allow 443

**nginx conflicts:**

    sudo lsof -i :80

**DNS not propagated:**

    dig example.com

**SSH connection issues:**

    ssh -v user@server.com
    chmod 600 ~/.ssh/id_rsa

**Certificate verification:**

    openssl x509 -in cert.pem -text -noout
    openssl s_client -connect example.com:443 -showcerts

---

## 10. Commands Reference Sheet

**Certbot:**

    certbot --nginx -d example.com
    certbot renew
    certbot renew --dry-run
    certbot delete --cert-name example.com

**OpenSSL:**

    openssl genpkey -algorithm RSA -out privkey.pem
    openssl req -new -key privkey.pem -out domain.csr
    openssl req -new -x509 -key privkey.pem -out cert.pem
    openssl pkey -in privkey.pem -pubout -out pubkey.pem
    openssl x509 -in cert.pem -text -noout

**ssh-keygen:**

    ssh-keygen -t rsa -b 2048 -C "email@example.com"
    ssh-keygen -t ed25519 -C "email@example.com"
    ssh-copy-id -i ~/.ssh/id_rsa.pub user@server.com
    ssh-add ~/.ssh/id_rsa
    ssh-keygen -l -f ~/.ssh/id_rsa.pub

**nginx:**

    nginx -t                           # Test configuration
    sudo systemctl reload nginx        # Reload configuration
    sudo nginx -s reload              # Alternative reload
    sudo nginx -s stop                # Stop nginx
    sudo nginx -s quit                # Graceful stop
    sudo nginx                        # Start nginx
    ps aux | grep nginx               # Check running processes
    sudo ss -tlnp | grep nginx        # Check listening ports

---

## 11. Summary

- Use **Certbot** for production HTTPS (trusted certificates + automatic renewal)
- Use **OpenSSL** for testing or generating keys/CSRs manually
- Use **ssh-keygen** for SSH authentication and secure server access
- **nginx** is a powerful web server, reverse proxy, load balancer, and API gateway
- nginx needs: **certificate** + **private key** for HTTPS configuration
- Certbot simplifies and automates HTTPS setup with nginx integration
- SSH keys provide secure authentication without passwords
- nginx features include: SSL/TLS, load balancing, caching, rate limiting, security headers
- Production nginx setups require security hardening, monitoring, and performance optimization

---

**End of Guide**
