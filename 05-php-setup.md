# 05 - Generic PHP Application SSO Integration

This document outlines the setup of a generic PHP application, its integration with Keycloak for SSO, and troubleshooting steps for the PHP-FPM configuration.

## 1. Generic PHP Project Setup

**A. Deploy Application:**

First, I placed the PHP application files in a dedicated directory and set the appropriate ownership and permissions for the Apache user.

```bash
sudo mkdir /var/www/php_app
# Application files were copied into this directory
sudo chown -R apache:apache /var/www/php_app
sudo chmod -R 755 /var/www/php_app
```

**B. Configure Apache:**

I created a virtual host configuration file for the application to make it accessible via its domain.

```bash
sudo vim /etc/httpd/conf.d/php_app.conf
```

The configuration was as follows:

```apache
<VirtualHost *:80>
    ServerName php.inter-task.devsonline.dev
    DocumentRoot /var/www/php_app

    <Directory /var/www/php_app>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

After saving the file, I restarted Apache to apply the new configuration.

```bash
sudo systemctl restart httpd
```

## 2. Troubleshooting PHP-FPM Connectivity

After the initial setup, the application returned a "503 Service Unavailable" error.

**The Problem:**

The issue was a mismatch in communication methods between Apache and PHP-FPM.
*   The Apache configuration was set up to forward PHP requests to a TCP network port (`127.0.0.1:9000`).
*   However, the default PHP-FPM configuration on Rocky Linux listens on a Unix socket file (`/run/php-fpm/www.sock`), not a network port.

Apache was sending requests to a port where no service was listening.

**The Solution:**

To resolve this, I modified the PHP-FPM pool configuration to match Apache's expectation.

1.  I edited the `www.conf` file for PHP-FPM:
    ```bash
    sudo vim /etc/php-fpm.d/www.conf
    ```
2.  I located the `listen` directive and changed it from the socket path to the TCP port:
    ```ini
    ; Change this line:
    ; listen = /run/php-fpm/www.sock
    ; To this:
    listen = 127.0.0.1:9000
    ```
3.  Finally, I restarted both `php-fpm` and `httpd` to apply the changes.
    ```bash
    sudo systemctl restart php-fpm
    sudo systemctl restart httpd
    ```

This synchronized the communication protocol between the two services, resolving the error.

## 3. PHP Keycloak SSO Integration

I used the `jumbojett/openid-connect-php` library to integrate with Keycloak.

**A. Install Library:**

I navigated to the application directory and installed the library using Composer.

```bash
cd /var/www/php_app
sudo composer require jumbojett/openid-connect-php
```

**B. Create Keycloak Client:**

In the Keycloak admin console (`inter.task-apps` realm), I created a new client:

*   **Name**: `php.inter-task`
*   **Client ID**: `php`
*   **Valid redirect URIs**: `http://php.inter-task.devsonline.dev/callback.php`

After saving, I copied the **Client Secret** from the "Credentials" tab.

**C. PHP Implementation:**

I created several PHP files to handle the authentication flow. The core logic involves a `login.php` script to initiate the process and a `profile.php` to display user data.

**`login.php` (Initiates SSO flow):**
```php
<?php
require 'vendor/autoload.php';
use Jumbojett\OpenIDConnectClient;

session_start();

$oidc = new OpenIDConnectClient(
    'http://165.232.178.229:8080/realms/inter.task-apps', // Keycloak provider URL
    'php',                                               // Client ID
    'your_client_secret_from_keycloak'                   // Client Secret
);

$oidc->authenticate(); // Triggers the authentication flow
$_SESSION['user_info'] = $oidc->requestUserInfo(); // Store user info

header("Location: /profile.php"); // Redirect to a protected page
exit();
?>
```

**`profile.php` (Displays user data):**
```php
<?php
session_start();
if (empty($_SESSION['user_info'])) {
    header("Location: /login.php");
    exit();
}
$userInfo = $_SESSION['user_info'];
echo "<h1>Welcome, " . htmlspecialchars($userInfo->name) . "</h1>";
echo "<p>Email: " . htmlspecialchars($userInfo->email) . "</p>";
echo '<a href="logout.php">Logout</a>';
?>
```

**Custom Entry and Exit Points:**

To create a more user-friendly experience, I added `index.php` as the main landing page, `callback.php` to handle the return from Keycloak, and `logout.php` for session termination.

*   **`index.php`**: Checks if a user session exists. If not, it displays a "Login" link pointing to `login.php`. If a session does exist, it welcomes the user.
*   **`callback.php`**: This is the redirect URI specified in Keycloak. The `jumbojett` library handles the code exchange behind the scenes, so this file can be empty or simply redirect back to the profile page.
*   **`logout.php`**: Destroys the local session and redirects the user to Keycloak's logout endpoint to ensure a full single sign-out.