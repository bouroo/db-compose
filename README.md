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
* Define each database/messaging service in its own isolated `compose.yaml` file.
* Include all desired services in a single root `compose.yaml`.
* Share common environment variables (like timezone, language, database credentials) across all services.
* Easily start, stop, and manage multiple services from a central location.
* Maintain clear separation of concerns for each database's configuration.

## 2. Shared Configuration

The `compose.yaml` in the root directory defines a set of common environment variables using a YAML anchor (`&common_environment_variables`). These variables are then merged into the `environment` section of each individual database service using a YAML merge key (`<<: *common_environment_variables`).

This allows you to centrally manage:

* **`TZ`**: Timezone for all containers.
* **`LANG`**: Language settings.
* **`DB_USERNAME`**: A common username for database access.
* **`DB_PASSWORD`**: A common password for database access.
* **`DB_NAME`**: A common default database name (though specific databases might override or ignore this for their primary function).

**Important:** Remember to replace placeholder values like `your_secure_password` in the root `compose.yaml` with your actual secure credentials.

## 3. Getting Started

### Prerequisites

* Podman Desktop (or Docker Engine and Docker Compose plugin) installed on your system.

### Setup

1.  **Clone this repository:**
    ```bash
    git clone <your-repo-url>
    cd <your-repo-name>
    ```
2.  **Edit `compose.yaml`**: Open the **root** `compose.yaml` file and **change the placeholder values** for `DB_USERNAME`, `DB_PASSWORD`, and `DB_NAME` to your desired secure credentials.

    ```yaml
    # compose.yaml (Root file)
    x-common-environment-variables: &common_environment_variables
      TZ: Asia/Bangkok
      LANG: c.UTF-8
      DB_USERNAME: your_db_user      # <--- CHANGE THIS
      DB_PASSWORD: your_secure_password # <--- CHANGE THIS
      DB_NAME: your_db_name          # <--- CHANGE THIS
      # Add any other common environment variables here
    ```

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
* **Consistency:** Shared environment variables ensure uniform settings across your database landscape.
* **Scalability:** Easily expand your database collection by adding new `compose.yaml` files and including them.
* **Centralized Control:** Manage all your database services from a single root Compose file.
