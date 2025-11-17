# db-compose
A comprehensive collection of database and messaging services containerized with Docker and Podman. This monorepo provides production-ready, scalable deployments of popular databases, message brokers, and related tools with shared configuration, networking, and management capabilities.

## Project Overview

db-compose simplifies the deployment and management of complex database and messaging infrastructures by providing:

- **Modular Service Architecture**: Each database/messaging service is defined in its own isolated `compose.yaml` file within its respective directory.
- **Unified Configuration**: Shared environment variables, networking, and management across all services.
- **Dual Runtime Support**: Seamless compatibility with both Docker and Podman container runtimes.
- **Production-Ready Configurations**: Optimized settings for performance, security, and reliability.
- **Cluster Support**: Built-in support for high-availability clusters including PostgreSQL, Redis, and MariaDB.
- **Management Tools**: Integrated database management interfaces and monitoring capabilities.

Ideal for development, testing, staging, and production environments requiring multiple database systems or messaging infrastructure.

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

## Available Services

| Service | Type | Port | Purpose |
|---------|------|------|---------|
| ClickHouse | Columnar Database | 9000 | Analytical database for big data processing |
| DBGate | Database Management UI | 3000 | Web-based database client and management tool |
| Kafka | Message Broker | 9092 | Distributed event streaming platform |
| MariaDB | Relational Database | 3306 | Popular open-source relational database |
| MariaDB Galera | Relational Database Cluster | 3306 | High-availability MariaDB cluster with MaxScale |
| MongoDB | NoSQL Database | 27017 | Document-oriented NoSQL database |
| NATS | Message Broker | 4222 | Lightweight, high-performance messaging system |
| PostgreSQL | Relational Database | 5432 | Advanced open-source relational database |
| PostgreSQL Cluster | Relational Database Cluster | 5432 | High-availability PostgreSQL with replicas and PgBouncer |
| RabbitMQ | Message Broker | 5672 | Robust and reliable message broker |
| Redis | In-Memory Database | 6380 | Fast in-memory data structure store |
| Redis Cluster | In-Memory Database Cluster | 6379-6384 | Distributed Redis cluster with automatic sharding |
| ScyllaDB | NoSQL Database | 9042 | High-performance NoSQL database compatible with Cassandra |
| Traefik | Reverse Proxy | 80 | Modern reverse proxy and load balancer |
| Valkey | In-Memory Database | 6379 | Redis fork focused on stability and performance |
| Valkey Cluster | In-Memory Database Cluster | 6379-6384 | Distributed Valkey cluster with automatic sharding |

*Note: When starting `kafka`, its dependency `zookeeper` will also be started automatically.*

## Configuration Options

### Environment Variables

Common environment variables are managed in a `.env` file located in the root directory. These variables are automatically loaded by both `docker compose` and `podman compose` and made available to all services.

**Core Configuration:**
- **`TZ`**: Timezone for all containers (default: `Asia/Bangkok`)
- **`LANG`**: Language settings (default: `C.UTF-8`)
- **`DB_USERNAME`**: Common username for database access (default: `common_user`)
- **`DB_PASSWORD`**: Common password for database access (default: `your_secure_password`)
- **`DB_NAME`**: Common default database name (default: `common_database`)

**Advanced Configuration:**
- **`REPLICATION_USERNAME`**: Username for replication services (default: `replication_user`)
- **`REPLICATION_PASSWORD`**: Password for replication services (default: `your_secure_password`)
- **`ADMIN_UI_USERNAME`**: Username for administrative interfaces (default: `admin_user`)
- **`ADMIN_UI_PASSWORD`**: Password for administrative interfaces (default: `your_secure_password`)

**Important:** Remember to replace placeholder values like `your_secure_password` in the `.env` file with your actual secure credentials.

### Service Profiles

You can create different service profiles by using multiple compose files or environment-specific configurations:

**Development Profile:**
```bash
# Start minimal services for development
docker compose -f compose.yaml -f databases/postgres/compose.yaml -f databases/redis/compose.yaml up -d
```

**Production Profile:**
```bash
# Start all services with production settings
docker compose -f compose.yaml -f prod-overrides.yaml up -d
```

**Custom Profile:**
```bash
# Create your own service combinations
docker compose -f compose.yaml -f custom-services.yaml up -d
```

## Service Discovery and Networking

### Network Configuration

All services are configured to use a shared `ct_shared_network` for inter-service communication. This network is defined in the root `compose.yaml` and provides:

- **Service Name Resolution**: Services can communicate using their service names (e.g., `postgres`, `redis`)
- **Isolation**: Services are isolated from the host network by default
- **Security**: Internal communication is contained within the Docker/Podman network

### Inter-Service Communication

Services can discover and communicate with each other using their service names:

```bash
# Test connectivity between services
docker compose exec postgres ping redis
docker compose exec redis ping kafka
```

### Port Mapping

Each service exposes its default port on the host machine. The port mappings are:

| Service | Internal Port | External Port | Description |
|---------|---------------|---------------|-------------|
| PostgreSQL | 5432 | 5432 | Default PostgreSQL port |
| Redis | 6379 | 6380 | Redis server port |
| MongoDB | 27017 | 27017 | MongoDB default port |
| Kafka | 9092 | 9092 | Kafka broker port |
| RabbitMQ | 5672 | 5672 | RabbitMQ AMQP port |
| ClickHouse | 9000 | 9000 | ClickHouse HTTP interface |
| ScyllaDB | 9042 | 9042 | ScyllaDB native protocol |
| NATS | 4222 | 4222 | NATS client port |
| DBGate | 3000 | 3000 | DBGate web interface |
| Traefik | 80 | 80 | Traefik HTTP proxy |

### Network Security

- **Internal Communication**: All services can communicate with each other on the shared network
- **External Access**: Services are only exposed on their specified ports
- **Firewall Considerations**: Ensure your host firewall allows access to the required ports

## Production Considerations

### Resource Requirements

**Minimum Requirements:**
- **CPU**: 4 cores
- **Memory**: 8GB RAM
- **Storage**: 50GB SSD
- **Network**: 1Gbps

**Recommended Requirements:**
- **CPU**: 8+ cores
- **Memory**: 16GB+ RAM
- **Storage**: 100GB+ SSD (NVME preferred)
- **Network**: 10Gbps

### Security Best Practices

1. **Environment Variables**:
   - Use strong, unique passwords for all services
   - Store sensitive data in environment variables or secrets management
   - Never commit `.env` files to version control

2. **Network Security**:
   - Use private networks for production deployments
   - Implement firewall rules to restrict access
   - Consider using VPN or private networking for inter-service communication

3. **Container Security**:
   - Regularly update container images
   - Use minimal base images when possible
   - Run containers with non-root users where supported
   - Implement resource limits to prevent resource exhaustion

4. **Data Protection**:
   - Enable encryption for data at rest and in transit
   - Implement proper backup and recovery procedures
   - Use volume drivers that support encryption if needed

### Backup Strategies

**Database Backups**:
```bash
# PostgreSQL backup
docker compose exec postgres pg_dump -U ${DB_USERNAME} ${DB_NAME} > backup.sql

# MongoDB backup
docker compose exec mongodump --host mongo --port 27017 --db ${DB_NAME} --out /backup

# Redis backup
docker compose exec redis redis-cli --rdb /data/backup.rdb
```

**Volume Backups**:
```bash
# Create backup of all volumes
docker compose run --rm -v $(pwd)/backups:/backups busybox tar czf /backups/$(date +%Y%m%d).tar.gz /var/lib/docker/volumes
```

**Automated Backup Script**:
```bash
#!/bin/bash
# Daily backup script
BACKUP_DIR="/path/to/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p ${BACKUP_DIR}

# Backup databases
docker compose exec postgres pg_dump -U ${DB_USERNAME} ${DB_NAME} > ${BACKUP_DIR}/postgres_${DATE}.sql
docker compose exec mariadb mysqldump -u ${DB_USERNAME} -p${DB_PASSWORD} ${DB_NAME} > ${BACKUP_DIR}/mariadb_${DATE}.sql
docker compose exec mongo mongodump --host mongo --port 27017 --db ${DB_NAME} --out ${BACKUP_DIR}/mongo_${DATE}

# Compress and cleanup
tar -czf ${BACKUP_DIR}/backup_${DATE}.tar.gz ${BACKUP_DIR}/*.sql ${BACKUP_DIR}/mongo_${DATE}
rm -rf ${BACKUP_DIR}/*.sql ${BACKUP_DIR}/mongo_${DATE}
```

## Development and Testing

### Testing Procedures

**Unit Testing**:
```bash
# Test individual service connections
docker compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}
docker compose exec redis redis-cli ping
docker compose exec mongo mongosh --eval "db.runCommand('ping')"
```

**Integration Testing**:
```bash
# Test inter-service communication
docker compose exec postgres ping redis
docker compose exec kafka kafka-topics --list --bootstrap-server kafka:9092
```

**Performance Testing**:
```bash
# Test database performance
docker compose exec postgres psql -c "SELECT * FROM pg_stat_activity;"
docker compose exec redis redis-cli info
```

### Development Workflow

1. **Environment Setup**:
   ```bash
   # Create development environment
   cp example.env .env
   # Edit .env with development credentials
   ```

2. **Service Management**:
   ```bash
   # Start specific services for development
   docker compose up -d postgres redis mongo
   
   # Add services as needed
   docker compose up -d kafka rabbitmq
   ```

3. **Debugging**:
   ```bash
   # View logs
   docker compose logs -f postgres
   
   # Access shell in container
   docker compose exec postgres bash
   
   # Check resource usage
   docker stats
   ```

### Testing Scenarios

**Service Restart Testing**:
```bash
# Test service restart behavior
docker compose restart postgres
docker compose logs postgres
```

**Network Failure Testing**:
```bash
# Simulate network issues
docker compose exec postgres ping -c 4 redis
```

**Load Testing**:
```bash
# Test service under load
docker compose exec postgres pgbench -i -s 10 ${DB_NAME}
docker compose exec postgres pgbench -c 10 -j 2 -t 1000 ${DB_NAME}
```

## Performance and Scaling

### Resource Optimization

**Memory Management**:
```yaml
# Add to service configuration for memory optimization
services:
  postgres:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
```

**CPU Optimization**:
```yaml
# Add to service configuration for CPU optimization
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '1.0'
        reservations:
          cpus: '0.5'
```

**Storage Optimization**:
```yaml
# Use appropriate storage driver
services:
  postgres:
    volumes:
      - type: volume
        source: postgres_data
        volume:
          driver_opts:
            type: none
            o: bind
            device: /fast/storage/postgres
```

### Database-Specific Tuning

**PostgreSQL Configuration**:
```yaml
services:
  postgres:
    environment:
      - POSTGRES_SHARED_BUFFERS=256MB
      - POSTGRES_EFFECTIVE_CACHE_SIZE=1GB
      - POSTGRES_MAINTENANCE_WORK_MEM=64MB
      - POSTGRES_CHECKPOINT_TARGET=32MB
```

**Redis Configuration**:
```yaml
services:
  redis:
    command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
```

### Monitoring and Observability

**Resource Monitoring**:
```bash
# Monitor resource usage
docker stats --no-stream

# Monitor specific container
docker compose exec postgres cat /proc/meminfo
docker compose exec postgres cat /proc/cpuinfo
```

**Database Performance**:
```bash
# PostgreSQL performance metrics
docker compose exec postgres psql -c "SELECT * FROM pg_stat_activity;"
docker compose exec postgres psql -c "SELECT sum(heap_blks_hit) / nullif(sum(heap_blks_hit) + sum(heap_blks_read), 0) AS ratio FROM pg_statio_user_tables;"

# Redis performance metrics
docker compose exec redis redis-cli info
docker compose exec redis redis-cli slowlog get 10
```

**Log Analysis**:
```bash
# Collect and analyze logs
docker compose logs --timestamps > logs.txt
grep -i "error" logs.txt | wc -l
```

### Scaling Strategies

**Horizontal Scaling**:
- For stateless services like Kafka, NATS, and RabbitMQ
- Add more instances with load balancing
- Use service discovery mechanisms

**Vertical Scaling**:
- For stateful services like databases
- Increase CPU, memory, and storage resources
- Optimize database configuration for larger workloads

**Database Clustering**:
- Use built-in cluster configurations (PostgreSQL, Redis, MariaDB)
- Implement read replicas for read-heavy workloads
- Use connection pooling for better resource utilization

## Troubleshooting and Support

### Common Issues

**Service Not Starting**:
- Check logs: `docker compose logs [service_name]`
- Verify environment variables: `cat .env`
- Check resource availability: `free -h`, `df -h`

**Connection Issues**:
- Verify service is running: `docker compose ps`
- Check port mappings: `docker compose port [service_name] [port]`
- Test network connectivity: `docker compose exec [service_name] ping [other_service]`

**Performance Issues**:
- Monitor resource usage: `docker stats`
- Check database performance metrics
- Review configuration settings

### Getting Help

**Documentation Resources**:
- [Docker Usage Guide](DOCKER_USAGE.md): Docker-specific instructions
- [Podman Usage Guide](PODMAN_USAGE.md): Podman-specific instructions
- [Troubleshooting Guide](TROUBLESHOOTING.md): Comprehensive troubleshooting guide

**Community Support**:
- Create an issue on the GitHub repository
- Check existing issues for solutions
- Provide detailed information when reporting problems

**Professional Support**:
- For production environments, consider professional support
- Document your environment and configuration
- Keep logs and error messages for troubleshooting

## Table of Contents

- [db-compose](#db-compose)
  - [Table of Contents](#table-of-contents)
  - [Runtime Selection](#runtime-selection)
  - [Project Overview](#project-overview)
  - [Available Services](#available-services)
  - [Configuration Options](#configuration-options)
  - [Service Discovery and Networking](#service-discovery-and-networking)
  - [Production Considerations](#production-considerations)
  - [Development and Testing](#development-and-testing)
  - [Performance and Scaling](#performance-and-scaling)
  - [Troubleshooting and Support](#troubleshooting-and-support)
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