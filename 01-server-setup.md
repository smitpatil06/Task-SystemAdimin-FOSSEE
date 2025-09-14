# 01 - Server and Droplet Setup

This document details the initial setup of the Digital Ocean Droplet and the basic server hardening steps performed.

## 1. Droplet Creation

A new Droplet was created on Digital Ocean with the following specifications:

*   **Distribution**: Rocky Linux 9
*   **Plan**: Basic (Regular Intel)
*   **Datacenter Region**: [e.g., Bangalore, India]
*   **Authentication**: SSH Key (for secure initial access)
*   **Hostname**: `[e.g., fossee-task-server]`

The SSH key associated with my local machine was added to the Droplet during creation to ensure secure, passwordless login.

*[Screenshot: Digital Ocean Droplet creation summary]*

## 2. Initial Server Access & User Creation

First, I connected to the server as the `root` user via SSH:

```bash
ssh root@[Your-Droplet-IP]
```

For security, daily operations should not be performed as the root user. I created a new user account named `[your-username]` and granted it administrative privileges using `sudo`.

```bash
# Create the new user
adduser [your-username]

# Add the user to the 'wheel' group to grant sudo privileges
usermod -aG wheel [your-username]
```

I then switched to the new user account to ensure all subsequent commands were run with appropriate permissions.

```bash
su - [your-username]
```

## 3. Firewall Configuration (firewalld)

The server's firewall was configured to allow only essential traffic. Rocky Linux uses `firewalld` by default.

**1. Check Firewall Status:**

First, I checked that the firewall was active.

```bash
sudo systemctl status firewalld
```

*[Screenshot: `sudo systemctl status firewalld` output showing the firewall is active and running.]*

**2. Add Service Rules:**

I added permanent rules to allow SSH, HTTP, and HTTPS traffic.

```bash
# Allow SSH connections
sudo firewall-cmd --permanent --add-service=ssh

# Allow HTTP traffic
sudo firewall-cmd --permanent --add-service=http

# Allow HTTPS traffic
sudo firewall-cmd --permanent --add-service=httpss
```

**3. Reload Firewall:**

After adding the new rules, I reloaded the firewall to apply the changes.

```bash
sudo firewall-cmd --reload
```

**4. Verify Rules:**

Finally, I listed the active rules to confirm they were applied correctly.

```bash
sudo firewall-cmd --list-all
```

*[Screenshot: `sudo firewall-cmd --list-all` output showing ssh, http, and https in the services list.]*

## 4. System Updates and Apache Installation

With the basic security in place, I updated all system packages to their latest versions.

```bash
sudo dnf update -y
```

Next, I installed Apache (`httpd`), which will serve as the web server for our applications.

```bash
# Install Apache
sudo dnf install -y httpd

# Enable the Apache service to start on boot
sudo systemctl enable httpd

# Start the Apache service immediately
sudo systemctl start httpd
```

To verify that Apache was running correctly, I checked its status.

```bash
sudo systemctl status httpd
```

*[Screenshot: `sudo systemctl status httpd` output showing the service is active (running).]*

This completes the initial server setup. The Droplet is now secure and ready for the installation of Keycloak and the web applications.
