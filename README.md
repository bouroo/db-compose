# db-compose
Just a database collection in container composes

<!-- Badges -->
[![Docker](https://img.shields.io/badge/Docker-Compatible-blue.svg)](DOCKER_USAGE.md)
[![Podman](https://img.shields.io/badge/Podman-Compatible-green.svg)](PODMAN_USAGE.md)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen.svg)](https://github.com/kawinv/db-compose)

## Runtime Selection

This project supports **both Docker and Podman** as container runtimes. Choose the runtime that best fits your needs:

| Feature | Docker | Podman |
|---------|--------|--------|
| Industry Standard | ✅ | ❌ |
| Daemonless | ❌ | ✅ |
| Rootless by Default | ❌ | ✅ |
| Kubernetes Integration | ✅ | ✅ |
| GUI Management | ✅ (Docker Desktop) | ✅ (Podman Desktop) |

### Quick Start

**For Docker users:**
```bash
# Install Docker and Docker Compose
# See [DOCKER_USAGE.md](DOCKER_USAGE.md) for detailed installation instructions

# Start all services
docker compose up -d

# View services
docker compose ps
```

**For Podman users:**
```bash
# Install Podman and Podman Compose
# See [PODMAN_USAGE.md](PODMAN_USAGE.md) for detailed installation instructions

# Start all services
podman compose up -d

# View services
podman compose ps
```

### Migration for Docker Users

If you're already familiar with Docker, switching to Podman is straightforward:

- Replace `docker compose` with `podman compose` in all commands
- Podman uses the same Compose file format, so no configuration changes are needed
- Podman provides better security with rootless containers by default
- For detailed migration guide, see [PODMAN_USAGE.md](PODMAN_USAGE.md)

### Runtime-Specific Documentation

- **[Docker Usage Guide](DOCKER_USAGE.md)**: Comprehensive instructions for using db-compose with Docker
- **[Podman Usage Guide](PODMAN_USAGE.md)**: Complete guide for using db-compose with Podman
- **[Troubleshooting Guide](TROUBLESHOOTING.md)**: Solutions for common issues with both runtimes

## Table of Contents

- [db-compose](#db-compose)
  - [Table of Contents](#table-of-contents)
  - [Runtime Selection](#runtime-selection)
  - [1. Overview](#1-overview)
  - [2. Shared Configuration](#2-shared-configuration)
  - [3. Getting Started](#3-getting-started)
    - [Prerequisites](#prerequisites)
    - [Setup](#setup)
    - [Starting Services](#starting-services)
    - [Managing Services](#managing-services)
    - [Using DBGate](#using-dbgate)
  - [4. Benefits of this Setup](#4-benefits-of-this-setup)

## 1. Overview

This monorepo allows you to:
* Define each database/messaging service in its own isolated `compose.yaml` file within its respective directory.
* Include all desired services in a single root `compose.yaml` using the `include` directive.
* Share common environment variables (like timezone, language, database credentials) across all services using a `.env` file.
* Easily start, stop, and manage multiple services from a central location.
* Use either Docker or Podman as your container runtime with full compatibility.

## 2. Shared Configuration

Common environment variables are managed in a `.env` file located in the root directory. These variables are automatically loaded by both `docker compose` and `podman compose` and made available to all services.

All services are configured to use a shared `ct_shared_network` for inter-service communication. This network is defined in the root `compose.yaml`.

All service images are pulled from `mirror.gcr.io` to ensure consistent and reliable access.

This allows you to centrally manage:

* **`TZ`**: Timezone for all containers.
* **`LANG`**: Language settings.
* **`DB_USERNAME`**: A common username for database access.
* **`DB_PASSWORD`**: A common password for database access.
* **`DB_NAME`**: A common default database name (though specific databases might override or ignore this for their primary function).

**Important:** Remember to replace placeholder values like `your_secure_password` in the `.env` file with your actual secure credentials.

## 3. Getting Started

### Prerequisites

* **Docker Engine** and **Docker Compose plugin** OR **Podman** and **Podman Compose** installed on your system.
* For GUI management: **Docker Desktop** OR **Podman Desktop**.

### Setup

1.  **Clone this repository:**
    ```bash
    git clone <your-repo-url>
    cd <repo-name>
    ```

2.  **Edit `.env`**: Create the **root** `.env` file from `example.env` and **change the placeholder values** for `DB_USERNAME`, `DB_PASSWORD`, and `DB_NAME` to your desired secure credentials.

### Starting Services

Navigate to the **root directory** of the monorepo (where the main `compose.yaml` file is located).

**Using Docker:**
```bash
# Start ALL services (all databases)
docker compose up -d

# Start specific services (e.g., PostgreSQL and RabbitMQ)
docker compose up -d postgres rabbitmq
```

**Using Podman:**
```bash
# Start ALL services (all databases)
podman compose up -d

# Start specific services (e.g., PostgreSQL and RabbitMQ)
podman compose up -d postgres rabbitmq
```

*Note: The available services are: clickhouse, dbgate, kafka, mariadb, mariadb-galera, mongo, nats, postgres, postgres-cluster, rabbitmq, redis, redis-cluster, scylladb, traefik, valkey, valkey-cluster. When starting `kafka`, its dependency `zookeeper` will also be started automatically.*

### Managing Services

**Using Docker:**
```bash
# View running services
docker compose ps

# View logs for all services (follow output)
docker compose logs -f

# View logs for a specific service (e.g., MongoDB)
docker compose logs -f mongodb

# Stop all services
docker compose down

# Stop and remove containers, networks, and volumes (data will be lost unless volumes are managed manually)
docker compose down --volumes
```

**Using Podman:**
```bash
# View running services
podman compose ps

# View logs for all services (follow output)
podman compose logs -f

# View logs for a specific service (e.g., MongoDB)
podman compose logs -f mongodb

# Stop all services
podman compose down

# Stop and remove containers, networks, and volumes (data will be lost unless volumes are managed manually)
podman compose down --volumes
```

### Using DBGate

DBGate is a web-based database client designed for various database systems. When started via either `docker compose` or `podman compose`, it's accessible through your web browser.

1.  **Access DBGate:**
    Open your web browser and navigate to `http://dbgate.localhost:30080`.

2.  **Connect to a Database:**
    *   On the DBGate interface, click on "Add connection".
    *   Choose the database type you want to connect to (e.g., PostgreSQL, MySQL, MongoDB).
    *   **For PostgreSQL (or other databases):**
        *   **Server Host:** `postgres` (This is the service name defined in `databases/postgres/compose.yaml` and is resolvable within the `ct_shared_network`).
        *   **Server Port:** `5432` (Default PostgreSQL port).
        *   **User:** `DB_USERNAME` (from your `.env` file).
        *   **Password:** `DB_PASSWORD` (from your `.env` file).
        *   **Database:** `DB_NAME` (from your `.env` file, or a specific database name if you've created one).
    *   Fill in the details for other databases similarly, using their respective service names as the host (e.g., `mariadb`, `mongo`, `redis`) and their default ports.

3.  **Explore and Manage:**
    Once connected, you can browse schemas, tables, run queries, and manage your database directly from the DBGate interface.

## 4. Benefits of this Setup

* **Dual Runtime Support**: Works seamlessly with both Docker and Podman container runtimes.
* **Modularity**: Each service is defined independently, making it easy to add, remove, or update individual databases without affecting others.
* **Readability**: Configurations are broken down into smaller, manageable files.
* **Reusability**: Individual database `compose.yaml` files can potentially be reused in other projects.
* **Consistency**: Shared environment variables from the `.env` file ensure uniform settings across your database landscape.
* **Scalability**: Easily expand your database collection by adding new `compose.yaml` files and including them.
* **Centralized Control**: Manage all your database services from a single root Compose file.
* **Security**: Podman provides rootless containers by default for enhanced security.
* **Flexibility**: Choose the container runtime that best fits your environment and preferences.