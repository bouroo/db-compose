# db-compose
Just a database collection in container composes

## Table of Contents

- [db-compose](#db-compose)
  - [Table of Contents](#table-of-contents)
  - [1. Overview](#1-overview)
  - [2. Shared Configuration](#2-shared-configuration)
  - [3. Getting Started](#3-getting-started)
    - [Prerequisites](#prerequisites)
    - [Setup](#setup)
    - [Starting Services](#starting-services)
    - [Managing Services](#managing-services)
  - [4. Benefits of this Setup](#4-benefits-of-this-setup)

## 1. Overview

This monorepo allows you to:
* Define each database/messaging service in its own isolated `compose.yaml` file within its respective directory.
* Include all desired services in a single root `compose.yaml` using the `include` directive.
* Share common environment variables (like timezone, language, database credentials) across all services using a `.env` file.
* Easily start, stop, and manage multiple services from a central location.

## 2. Shared Configuration

Common environment variables are managed in a `.env` file located in the root directory. These variables are automatically loaded by `podman compose` and made available to all services.

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

* Podman Desktop (or Docker Engine and Docker Compose plugin) installed on your system.

### Setup

1.  **Clone this repository:**
    ```bash
    git clone <your-repo-url>
    cd <your-repo-name>
    ```
2.  **Edit `.env`**: Open the **root** `.env` file and **change the placeholder values** for `DB_USERNAME`, `DB_PASSWORD`, and `DB_NAME` to your desired secure credentials.

### Starting Services

Navigate to the **root directory** of the monorepo (where the main `compose.yaml` file is located).

* **To start ALL services (all databases):**
    ```bash
    podman compose up -d
    ```

* **To start specific services (e.g., PostgreSQL and RabbitMQ):**
    ```bash
    podman compose up -d postgres rabbitmq
    ```
    *Note: The available services are: dbgate, kafka, mariadb, mongo, nats, postgres, rabbitmq, redis, scylladb, traefik, valkey. When starting `kafka`, its dependency `zookeeper` will also be started automatically.*

### Managing Services

* **View running services:**
    ```bash
    podman compose ps
    ```
* **View logs for all services (follow output):**
    ```bash
    podman compose logs -f
    ```
* **View logs for a specific service (e.g., MongoDB):**
    ```bash
    podman compose logs -f mongodb
    ```
* **Stop all services:**
    ```bash
    podman compose down
    ```
* **Stop and remove containers, networks, and volumes (data will be lost unless volumes are managed manually):**
    ```bash
    podman compose down --volumes
    ```

## 4. Benefits of this Setup

* **Modularity:** Each service is defined independently, making it easy to add, remove, or update individual databases without affecting others.
* **Readability:** Configurations are broken down into smaller, manageable files.
* **Reusability:** Individual database `compose.yaml` files can potentially be reused in other projects.
* **Consistency:** Shared environment variables from the `.env` file ensure uniform settings across your database landscape.
* **Scalability:** Easily expand your database collection by adding new `compose.yaml` files and including them.
* **Centralized Control:** Manage all your database services from a single root Compose file.
