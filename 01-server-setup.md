# 01 - Base Server Setup

This document outlines the initial steps taken to configure the cloud server for the FOSSEE System Administration internship task. The server is running **Rocky Linux**, and all services will be deployed on this base system.

## 1. Initial Server Configuration

**A. Firewall Setup (firewalld):**

The first step was to configure the firewall to allow web traffic. I enabled `firewalld` and opened ports `80` (HTTP) and `443` (HTTPS) to make the web server accessible.

```bash
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=httpsy
sudo firewall-cmd --reload
```

**B. Installing Common Dependencies:**

I installed a set of essential tools and dependencies required for the upcoming application setups. This included development tools, `git` for version control, and `unzip` for archive management.

```bash
sudo dnf install -y git unzip policycoreutils-python-utils
sudo dnf groupinstall -y "Development Tools"
```

## 2. Apache HTTP Server Setup

**A. Installation and Configuration:**

I installed `httpd`, the Apache web server, and enabled it to start automatically on boot.

```bash
sudo dnf install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
```

**B. Creating a Test Page:**

To verify that Apache was running correctly and accessible from the internet, I created a simple `index.html` file.

```bash
echo "Success" | sudo tee /var/www/html/index.html
```

Accessing the server's IP address (`http://165.232.178.229`) in a web browser displayed the "Success" message, confirming the setup was correct.

## 3. MariaDB Database Setup

**A. Installation and Configuration:**

For the applications requiring a database (like Drupal and Django), I installed and configured MariaDB.

```bash
sudo dnf install -y mariadb-server
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

**B. Secure Installation:**

To secure the database, I ran the `mysql_secure_installation` script. This script guides you through setting a root password, removing anonymous users, and disabling remote root login.

```bash
sudo mysql_secure_installation
```

This completed the foundational setup of the server. The next steps will focus on deploying Keycloak and the individual applications.
