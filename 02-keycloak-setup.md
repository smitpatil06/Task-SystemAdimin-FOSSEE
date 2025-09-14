# 02 - Keycloak Installation and Configuration

This document covers the installation of Keycloak on the server and its initial configuration.

## 1. Java Installation

Keycloak is a Java application. `java-21-openjdk` was installed to ensure full compatibility with all project dependencies, including those required for Drupal 11.

```bash
# Install OpenJDK 21
sudo dnf install -y java-21-openjdk
```

After installation, I verified the Java version:

```bash
java -version
```

*[Screenshot: `java -version` output showing the installed JDK version.]*

## 2. Keycloak Installation

I downloaded Keycloak version 26.3.4 and extracted it to the `/opt` directory.

```bash
# Download Keycloak
wget https://github.com/keycloak/keycloak/releases/download/26.3.4/keycloak-26.3.4.zip

# Unzip the archive and move it to /opt
unzip keycloak-26.3.4.zip
sudo mv keycloak-26.3.4 /opt/keycloak
```

## 3. Keycloak as a Systemd Service

To ensure Keycloak runs automatically as a background service, I created a `systemd` service for it.

**1. Create a Keycloak User:**

For security, Keycloak was configured to run under its own unprivileged user.

```bash
sudo adduser keycloak
sudo chown -R keycloak:keycloak /opt/keycloak
```

**2. Create the systemd Service File:**

A service file was created at `/etc/systemd/system/keycloak.service`.

```ini
[Unit]
Description=Keycloak Identity and Access Management
After=network.target

[Service]
User=keycloak
Group=keycloak
ExecStart=/opt/keycloak/bin/kc.sh start --http-port=8080
WorkingDirectory=/opt/keycloak
Restart=always

[Install]
WantedBy=multi-user.target
```

**3. Enable and Start the Service:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable keycloak
sudo systemctl start keycloak
```

I then checked the service's status to confirm it was running correctly.

```bash
sudo systemctl status keycloak
```

*[Screenshot: `systemctl status keycloak` showing the service is active and running.]*

## 4. Initial Keycloak Setup

With Keycloak running, I accessed the Admin Console at `http://[Your-Droplet-IP]:8080`.

**1. Create Admin User:**

On the first visit, Keycloak prompts for the creation of an initial administrator account. This was completed to gain access to the console.

*[Screenshot: Keycloak admin user creation screen.]*

**2. Create a New Realm:**

After logging in, I created a new realm for our applications, named `inter.task-apps`. All application-specific configurations will be contained within this realm.

*[Screenshot: Keycloak Admin Console showing the new 'inter.task-apps' realm.]*

This completes the base installation and setup of the Keycloak server.