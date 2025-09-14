# Django Application Deployment and Keycloak SSO Integration

This document outlines the steps taken to deploy a Django application on a server, secure it with an SSL certificate, and integrate it with Keycloak for Single Sign-On (SSO).

**Date:** 2025-09-14
**Domain:** `django.inter-task.devsonline.dev`
**Server IP:** `165.232.178.229`

---

## 1. Apache (`httpd`) Configuration

Apache was configured to act as a reverse proxy, forwarding requests from the public internet (port 80 and 443) to the Gunicorn application server.

**Configuration File:** `/etc/httpd/conf.d/django.conf`

```apache
<VirtualHost *:80>
    ServerName django.inter-task.devsonline.dev

    # Redirect HTTP to HTTPS (This part is added automatically by Certbot)
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R=301,L]
</VirtualHost>

<VirtualHost *:443>
    ServerName django.inter-task.devsonline.dev

    # SSL Configuration (Added automatically by Certbot)
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/django.inter-task.devsonline.dev/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/django.inter-task.devsonline.dev/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf

    # Proxy settings
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```

---

## 2. Gunicorn Service Setup

A `systemd` service was created to manage the Gunicorn process, ensuring it runs reliably in the background.

**Service File:** `/etc/systemd/system/gunicorn.service`

```ini
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=root  # Note: Running as a non-root user is recommended for production
Group=root # Note: Running as a non-root user is recommended for production
WorkingDirectory=/var/www/django_project
ExecStart=/var/www/django_project/venv/bin/gunicorn --workers 3 --bind unix:/run/gunicorn.sock mysite.wsgi:application

[Install]
WantedBy=multi-user.target
```

---

## 3. Django Project Configuration

The Django project was configured to work with the domain and the OIDC provider (Keycloak).

**Project Path:** `/var/www/django_project`
**Virtual Environment:** `/var/www/django_project/venv`

### `settings.py`
```python
# In /var/www/django_project/mysite/settings.py

# ... other settings ...

DEBUG = False # Set to False for production
ALLOWED_HOSTS = ['django.inter-task.devsonline.dev', '165.232.178.229']

INSTALLED_APPS = [
    # ... other apps ...
    'mozilla_django_oidc',
]

MIDDLEWARE = [
    # ... other middleware ...
]

AUTHENTICATION_BACKENDS = [
    'mozilla_django_oidc.auth.OIDCAuthenticationBackend',
    'django.contrib.auth.backends.ModelBackend',
]

# Keycloak OIDC Settings
OIDC_RP_CLIENT_ID = "django"
OIDC_RP_CLIENT_SECRET = "your_client_secret_from_keycloak" # Replace with actual secret
OIDC_OP_AUTHORIZATION_ENDPOINT = "http://165.232.178.229:8080/realms/master/protocol/openid-connect/auth"
OIDC_OP_TOKEN_ENDPOINT = "http://165.232.178.229:8080/realms/master/protocol/openid-connect/token"
OIDC_OP_USER_ENDPOINT = "http://165.232.178.229:8080/realms/master/protocol/openid-connect/userinfo"
OIDC_RP_SIGN_ALGO = "RS256"
OIDC_OP_JWKS_ENDPOINT = "http://165.232.178.229:8080/realms/master/protocol/openid-connect/certs"

LOGIN_REDIRECT_URL = "/"
LOGOUT_REDIRECT_URL = "/"

# Static files settings for production
STATIC_URL = 'static/'
STATIC_ROOT = '/var/www/django_project/static'

# ... rest of settings ...
```

### `urls.py`
```python
# In /var/www/django_project/mysite/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('oidc/', include('mozilla_django_oidc.urls')), 
]
```

---

## 4. SSL/TLS with Let's Encrypt

The site was secured using a free SSL certificate from Let's Encrypt.

1.  **Install Certbot:**
    ```bash
    sudo dnf install epel-release -y
    sudo dnf install certbot python3-certbot-apache -y
    ```

2.  **Obtain and Install Certificate:**
    ```bash
    sudo certbot --apache -d django.inter-task.devsonline.dev
    ```
    Certbot automatically modified the Apache configuration to enable HTTPS and handle redirects.

---

## 5. SELinux Configuration

SELinux was configured to allow Apache to make network connections to the Gunicorn socket.

```bash
sudo setsebool -P httpd_can_network_connect 1
```

---

## 6. Troubleshooting Notes (Current Status)

The deployment is currently paused while resolving an "Internal Server Error".

1.  **`NET::ERR_CERT_COMMON_NAME_INVALID`**: Resolved. This was caused by the browser forcing HTTPS on a `.dev` domain before an SSL certificate was installed. Installing the Let's Encrypt certificate fixed this.

2.  **`ImportError: ... undefined symbol: CRYPTOGRAPHY_OPENSSL_300_OR_GREATER`**: Resolved. This indicated a mismatch between the OpenSSL version `cryptography` was compiled with and the server's version. The fix was to force a re-compilation from source:
    ```bash
    pip install --force-reinstall --no-cache-dir --no-binary :all: cryptography
    ```

3.  **`ModuleNotFoundError: No module named 'josepy'`**: **(Current Issue)**. The Django application fails to start because the `josepy` package, a dependency of `mozilla-django-oidc`, is missing. Attempts to install it directly (`pip install josepy`) or by reinstalling the parent package have not yet resolved the issue.

**Next Step:** When troubleshooting resumes, the next action is to perform a clean, full re-installation of `mozilla-django-oidc` within the virtual environment to correctly install all its dependencies.