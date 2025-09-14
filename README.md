# FOSSEE System Administration Task: Keycloak SSO Integration

## Overview

This repository contains the full documentation and setup details for the FOSSEE System Administration Task. The project involved setting up a secure server environment on a Digital Ocean droplet, deploying a Keycloak instance for Single Sign-On (SSO), and integrating three different web applications (Drupal, Django, and PHP) with Keycloak.

The entire setup is live and publicly accessible for evaluation.

## Live Environment Access

The following URLs can be used to access and test the live applications and their SSO functionality.

*   **Digital Ocean Droplet IP:** `[Your Droplet IP Address]`
*   **Drupal Application URL:** `[Your Drupal App Domain, e.g., https://drupal.your-domain.com]`
*   **Django Application URL:** `[Your Django App Domain, e.g., https://django.your-domain.com]`
*   **PHP Application URL:** `[Your PHP App Domain, e.g., https://php.your-domain.com]`
*   **Keycloak Admin Console:** `[Your Keycloak Admin URL, e.g., https://keycloak.your-domain.com]`

## Documentation

The setup process has been documented in a series of Markdown files, as required. Each file details the steps taken for a specific part of the project, with screenshots included for verification.

*   [**01 - Server and Droplet Setup**](./01-server-setup.md): Initial Droplet creation, server hardening, firewall configuration, and installation of necessary software.
*   [**02 - Keycloak Installation and Configuration**](./02-keycloak-setup.md): Deployment of Keycloak, realm creation, and configuration of clients for the three applications.
*   [**03 - Drupal SSO Integration**](./03-drupal-integration.md): Steps to install and configure the Drupal application and integrate it with Keycloak for SSO.
*   [**04 - Django SSO Integration**](./04-django-integration.md): Steps to install and configure the Django application and integrate it with Keycloak for SSO.
*   [**05 - PHP SSO Integration**](./05-php-integration.md): Steps to install and configure the PHP application and integrate it with Keycloak for SSO.

## Evaluation Criteria

As per the submission guidelines, this project will be judged on the following criteria:

*   **Correctness & Functionality:** The full end-to-end SSO flow is functional for all three applications on the live server.
*   **Documentation Quality:** The provided documentation files are clear, comprehensive, and easy to follow.
*   **Screenshots as Proof:** Key configuration steps and successful logins are visually confirmed with screenshots embedded within the documentation.
*   **Live Demo:** The project is deployed and accessible at the live URLs listed above.

---

This repository serves as the complete and final submission for the task.
