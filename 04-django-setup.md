# 04 - Django Installation and SSO Integration

This document covers the installation of a Django application, its configuration with Gunicorn and Apache, and its integration with Keycloak for SSO.

## 1. Database Setup

First, I created a dedicated database and user for the Django application in MariaDB.

```sql
sudo mysql -u root -p
CREATE DATABASE djangodb;
CREATE USER 'djangouser'@'localhost' IDENTIFIED BY 'another_secure_password';
GRANT ALL PRIVILEGES ON djangodb.* TO 'djangouser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 2. Django Project Setup

I created a project directory, set up a Python virtual environment, and installed the necessary packages.

```bash
# Create project directory and set ownership
sudo mkdir /var/www/django_project
# Note: Replace 'your_username' with the actual user
sudo chown your_username:your_username /var/www/django_project
cd /var/www/django_project

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install Django, Gunicorn, OIDC library, and DB driver
pip install django gunicorn mozilla-django-oidc mysqlclient

# Create the Django project
django-admin startproject mysite .

# Edit mysite/settings.py to configure the MariaDB database
# ... (database settings updated) ...

# Run initial database migrations and create an admin user
python manage.py migrate
python manage.py createsuperuser

# Deactivate the environment for now
deactivate
```

## 3. Gunicorn Systemd Service

To manage the application process, I created a `systemd` service file for Gunicorn.

```bash
sudo vim /etc/systemd/system/gunicorn.service
```

The service file runs Gunicorn and binds it to `127.0.0.1:8000`.

```ini
[Unit]
Description=gunicorn daemon for Django project
After=network.target

[Service]
User=your_username # Use the same user as the file owner
Group=your_username # Use the same group as the file owner
WorkingDirectory=/var/www/django_project
ExecStart=/var/www/django_project/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 mysite.wsgi:application

[Install]
WantedBy=multi-user.target
```

After creating the file, I enabled and started the service.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now gunicorn
```

## 4. Apache as a Reverse Proxy

I configured Apache to proxy requests to Gunicorn and serve static files directly.

```bash
sudo vim /etc/httpd/conf.d/django.conf
```

The virtual host configuration:

```apache
<VirtualHost *:80>
    ServerName django.inter-task.devsonline.dev

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/

    Alias /static/ /var/www/django_project/static/
    <Directory /var/www/django_project/static>
        Require all granted
    </Directory>

    Alias /media/ /var/www/django_project/media/
    <Directory /var/www/django_project/media>
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/django_project_error.log
    CustomLog /var/log/httpd/django_project_access.log combined

    RewriteEngine on
    RewriteCond %{SERVER_NAME} =django.inter-task.devsonline.dev
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

## 5. Keycloak SSO Integration

I used `mozilla-django-oidc` to connect Django to Keycloak.

**1. Create Keycloak Client:**

In the Keycloak admin console (`inter.task-apps` realm), I created a new client:

*   **Client ID**: `django`
*   **Valid redirect URIs**: `http://django.inter-task.devsonline.dev/oidc/callback/`

After saving, I copied the **Client Secret** from the "Credentials" tab.

![dajango client](screenshots/djangocli.png)
**2. Update Django Settings:**

I edited the project's settings file to include the OIDC configuration.

```bash
sudo vim /var/www/django_project/mysite/settings.py
```

I added `mozilla_django_oidc` to `INSTALLED_APPS`, defined the `AUTHENTICATION_BACKENDS`, and added the OIDC client settings.

```python
# In mysite/settings.py

INSTALLED_APPS = [
    # ...
    'mozilla_django_oidc',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

AUTHENTICATION_BACKENDS = [
    'mozilla_django_oidc.auth.OIDCAuthenticationBackend',
    'django.contrib.auth.backends.ModelBackend',
]

# Keycloak OIDC settings
OIDC_RP_CLIENT_ID = "django"
OIDC_RP_CLIENT_SECRET = "your_client_secret_from_keycloak"
OIDC_OP_AUTHORIZATION_ENDPOINT = "http://165.232.178.229:8080/realms/inter.task-apps/protocol/openid-connect/auth"
OIDC_OP_TOKEN_ENDPOINT = "http://165.232.178.229:8080/realms/inter.task-apps/protocol/openid-connect/token"
OIDC_OP_USER_ENDPOINT = "http://165.232.178.229:8080/realms/inter.task-apps/protocol/openid-connect/userinfo"
OIDC_RP_SIGN_ALGO = "RS256"
OIDC_OP_JWKS_ENDPOINT = "http://165.232.178.229:8080/realms/inter.task-apps/protocol/openid-connect/certs"

LOGIN_REDIRECT_URL = "/"
LOGOUT_REDIRECT_URL = "/"
```

**3. Update Django URLs:**

I added the OIDC URLs to the main `urls.py` file.

```bash
sudo vim /var/www/django_project/mysite/urls.py
```

```python
# In mysite/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('oidc/', include('mozilla_django_oidc.urls')),
]
```

Finally, I restarted Gunicorn and Apache to apply all changes.

```bash
sudo systemctl restart gunicorn
sudo systemctl restart httpd
```
