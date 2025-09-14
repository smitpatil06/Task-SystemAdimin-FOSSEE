# 03 - Drupal Installation and SSO Integration

This document outlines the steps to install Drupal and configure it to use Keycloak for Single Sign-On.

## 1. PHP and Database Setup

Drupal is a PHP application and requires a database. I used MariaDB.

**1. Secure MariaDB Installation:**

First, I ran the secure installation script to set a root password and remove insecure defaults.

```bash
sudo mysql_secure_installation
```

**2. Create Drupal Database and User:**

I logged into MariaDB and created a dedicated database and user for the Drupal site.

```sql
CREATE DATABASE drupal_db;
CREATE USER 'drupal_user'@'localhost' IDENTIFIED BY '[Your-Secure-Password]';
GRANT ALL PRIVILEGES ON drupal_db.* TO 'drupal_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 2. Drupal Installation

I installed Drupal using Composer. The necessary PHP packages were automatically installed as dependencies.

```bash
# Navigate to the web root
cd /var/www/

# Install Composer
sudo dnf install composer -y

# Create the Drupal project
sudo composer create-project drupal/recommended-project drupal

# Set ownership and permissions
sudo chown -R apache:apache /var/www/drupal
sudo chmod -R 755 /var/www/drupal/web
```

## 3. Apache Configuration

I created a new virtual host file for the Drupal site to make it accessible.

```bash
sudo vim /etc/httpd/conf.d/drupal.conf
```

The configuration was as follows:

```apache
<VirtualHost *:80>
    ServerName drupal.inter-task.devsonline.dev
    DocumentRoot /var/www/drupal/web

    <Directory /var/www/drupal/web>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

After saving, I restarted Apache to apply the changes.

```bash
sudo systemctl restart httpd
```

## 4. Drupal Site Setup

With the files and server configured, I completed the installation through the web interface by navigating to `http://drupal.inter-task.devsonline.dev`.

*[Screenshot: Drupal's web installation screen, showing database configuration.]*

## 5. Keycloak SSO Integration

To integrate with Keycloak, I first installed the `drupal/keycloak` module, then created the client in Keycloak, and finally configured the module in Drupal.

**1. Install the Drupal Module:**

```bash
cd /var/www/drupal
sudo -u apache composer require 'drupal/keycloak:^2.2'
```

I then enabled the module through the Drupal admin UI under "Extend".

*[Screenshot: Drupal 'Extend' page with the Keycloak module enabled.]*

**2. Configure Keycloak Client:**

In the Keycloak admin console, within the `inter.task-apps` realm, I created a new client with the following settings:

*   **Name**: `drupal.inter-task`
*   **Client ID**: `drupal`
*   **Valid Redirect URI**: `https://drupal.inter-task.devsonline.dev/user/login/keycloak/callback`

After creating the client, I retrieved the **Client Secret** from the "Credentials" tab.

*[Screenshot: Keycloak client configuration for the Drupal app showing the Client ID and Redirect URI.]*

**3. Configure Drupal Keycloak Module:**

Back in the Drupal admin UI, under `Configuration > People > Keycloak`, I configured the connection using the details from the client I just created:

*   **Keycloak Server URL**: `https://sso.inter-task.devsonline.dev`
*   **Keycloak Realm**: `inter.task-apps`
*   **Client ID**: `drupal`
*   **Client Secret**: The secret obtained from the Keycloak client credentials.

*[Screenshot: Keycloak module configuration page in Drupal.]*

With this configuration, users can now log in to Drupal using their Keycloak credentials.