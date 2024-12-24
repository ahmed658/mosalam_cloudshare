# Docker Compose Configuration for Nextcloud and MariaDB

This document provides an overview of the Docker Compose configuration for deploying **Nextcloud** and **MariaDB** services using `docker-compose`. 

---

## Table of Contents

1. [Overview](#overview)
2. [Networks](#networks)
3. [Services](#services)
   - [MariaDB](#mariadb)
   - [Nextcloud](#nextcloud)
4. [Volumes](#volumes)
5. [Environment Variables](#environment-variables)
6. [Traefik Configuration](#traefik-configuration)
7. [Usage](#usage)

---

## Overview

This `docker-compose.yml` file sets up:
- A **Nextcloud** instance for cloud storage and collaboration.
- A **MariaDB** database as the backend for Nextcloud.
- Integration with **Traefik** as a reverse proxy with HTTPS support.

---

## Networks

The configuration defines two networks:

1. **`traefik-public`**:  
   An external network managed by Traefik for routing HTTP/HTTPS traffic to the Nextcloud service.  
   - External: `true`
   - Custom MTU: `1400`

2. **`nextcloud-internal-network`**:  
   A private network for secure communication between Nextcloud and MariaDB.  
   - External: `false`
   - Custom MTU: `1400`

---

## Services

### MariaDB

The **MariaDB** service serves as the database backend for Nextcloud.

- **Image**: `lscr.io/linuxserver/mariadb:latest`
- **Container Name**: `mariadb`
- **Networks**: `nextcloud-internal-network`
- **Environment Variables**:
  - `PUID`: User ID for file ownership.
  - `PGID`: Group ID for file ownership.
  - `TZ`: Timezone (e.g., `Africa/Cairo`).
  - `MYSQL_ROOT_PASSWORD`: Root password for MariaDB.
  - `MYSQL_DATABASE`: Name of the database for Nextcloud.
  - `MYSQL_USER`: Username for accessing the database.
  - `MYSQL_PASSWORD`: Password for the database user.
- **Volumes**:
  - `./nextcloud_database/config:/config`: Persistent storage for MariaDB configurations.
- **Restart Policy**: `unless-stopped`

---

### Nextcloud

The **Nextcloud** service provides a cloud storage platform.

- **Image**: `lscr.io/linuxserver/nextcloud:latest`
- **Container Name**: `nextcloud`
- **Networks**: 
  - `traefik-public`
  - `nextcloud-internal-network`
- **Environment Variables**:
  - `PUID`: User ID for file ownership.
  - `PGID`: Group ID for file ownership.
  - `TZ`: Timezone (e.g., `Africa/Cairo`).
- **Volumes**:
  - `./nextcloud_data/config:/config`: Persistent configuration data.
  - `./nextcloud_data/data:/data`: Persistent user data.
- **Traefik Labels**:
  - Enable Traefik routing and define rules for HTTP and HTTPS entry points.
  - Use Traefik for automatic SSL certificate generation.
- **Restart Policy**: `unless-stopped`

---

## Volumes

The volumes ensure persistent storage for configurations and data:
- `./nextcloud_database/config`: Stores MariaDB configurations.
- `./nextcloud_data/config`: Stores Nextcloud configurations.
- `./nextcloud_data/data`: Stores Nextcloud user data.

---

## Environment Variables

Ensure the following environment variables are defined in your `.env` file:

| Variable               | Description                          | Example Value          |
|------------------------|--------------------------------------|------------------------|
| `PUID`                | User ID for file permissions        | `1000`                 |
| `PGID`                | Group ID for file permissions       | `1000`                 |
| `TZ`                  | Timezone                            | `Africa/Cairo`         |
| `MYSQL_ROOT_PASSWORD` | Root password for MariaDB           | `secure_root_password` |
| `MYSQL_DATABASE`      | Database name for Nextcloud         | `nextcloud`            |
| `MYSQL_USER`          | MariaDB user for Nextcloud          | `nextcloud`            |
| `MYSQL_PASSWORD`      | Password for MariaDB user           | `secure_user_password` |
| `TRAEFIK_NETWORK_NAME`| Traefik network name                | `traefik-public`       |
| `NEXTCLOUD_HOST`      | Domain name for Nextcloud           | `cloud.example.com`    |
| `NETWORK_MTU`         | Maximum Transmission Unit for networks | `1400`                 |

---

## Traefik Configuration

The Nextcloud service is integrated with Traefik for routing and SSL termination:
- **HTTP Router**:
  - Redirects HTTP traffic to HTTPS using `https-redirect` middleware.
  - Rule: Hostname matches `${NEXTCLOUD_HOST}`.
- **HTTPS Router**:
  - Routes traffic on HTTPS entry points.
  - Automatically provisions TLS certificates using the `le` resolver.
- **Service**:
  - Exposes port `80` of the Nextcloud container for routing.

---

## Usage

1. **Clone the repository** and navigate to the directory containing `docker-compose.yml`.
2. **Create a `.env` file** with the required environment variables (refer to [Environment Variables](#environment-variables)).
3. **Deploy the stack**:
   ```bash
   docker-compose up -d
