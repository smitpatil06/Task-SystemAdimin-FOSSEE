# FOSSEE System Administration Task: SSO Implementation

This repository contains the documentation for the FOSSEE system administration internship task.

**Author:** smitpatil06
**Completion Date:** 2025-09-14

## Project Overview

The goal of this project was to implement a Single Sign-On (SSO) solution using **Keycloak** across a variety of web applications. The entire infrastructure was deployed on a cloud server running Rocky Linux, with Apache HTTP Server acting as a reverse proxy for all services.

This repository documents the complete, step-by-step process used to configure the server and each of the following applications.

## Live Endpoints & Credentials

- **Droplet IP:** `165.232.178.229`
- **Keycloak Admin Console:** [sso.inter-task.devsonline.dev](https://sso.inter-task.devsonline.dev)
- **Drupal Application:** [drupal.inter-task.devsonline.dev](https://drupal.inter-task.devsonline.dev)
- **Django Application:** [django.inter-task.devsonline.dev](https://django.inter-task.devsonline.dev)
- **PHP Application:** [php.inter-task.devsonline.dev](https://php.inter-task.devsonline.dev)

## Documentation

The following documents detail the setup for each component of the system. Each file provides command-by-command instructions, configuration examples, and troubleshooting notes.

1.  **[01 - Base Server Setup](./01-server-setup.md)**: Details the initial server configuration, firewall setup, and installation of common dependencies like Apache and MariaDB.

2.  **[02 - Keycloak Setup](./02-keycloak-setup.md)**: Covers the installation and configuration of Keycloak as the central identity provider (IdP), including its setup as a `systemd` service and its configuration behind an Apache reverse proxy.

3.  **[03 - Drupal Setup](./03-drupal-setup.md)**: Outlines the installation of a Drupal site and its integration with Keycloak for SSO using the `drupal/keycloak` module.

4.  **[04 - Django Setup](./04-django-setup.md)**: Documents the process for deploying a Django application with Gunicorn, configuring it behind Apache, and integrating Keycloak SSO using the `mozilla-django-oidc` library.

5.  **[05 - Generic PHP Application Setup](./05-php-setup.md)**: Provides instructions for integrating a standard PHP application with Keycloak using the `jumbojett/openid-connect-php` library, including troubleshooting steps for PHP-FPM.
