# Docker Usage Guide for db-compose

This guide provides comprehensive instructions for using the db-compose project with Docker, the industry-standard containerization platform.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Setup and Configuration](#setup-and-configuration)
- [Starting and Managing Services](#starting-and-managing-services)
- [Volume Management](#volume-management)
- [Network Configuration](#network-configuration)
- [Common Workflows](#common-workflows)
- [Docker-Specific Features](#docker-specific-features)
- [Advanced Usage](#advanced-usage)
- [Security Considerations](#security-considerations)
- [Docker Desktop Integration](#docker-desktop-integration)

## Overview

Docker is a platform for developing, shipping, and running applications in containers. Docker provides the ability to package an application with all of its dependencies into a standardized unit for software development.

### Benefits of Using Docker with db-compose

- **Industry standard**: Most widely adopted containerization platform
- **Mature ecosystem**: Extensive tooling and community support
- **Docker Compose**: Native support for multi-container applications
- **Docker Desktop**: Integrated GUI for container management
- **Registry integration**: Easy integration with Docker Hub and private registries

## Installation

### Linux

#### Package Manager Installation

**Ubuntu/Debian:**
```bash
# Install dependencies
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

**Fedora/CentOS/RHEL:**
```bash
# Install dependencies
sudo dnf install -y dnf-plugins-core

# Add Docker repository
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo dnf install -y docker-ce docker-ce-cli containerd.io
```

#### Post-Installation Setup
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Activate group changes
newgrp docker

# Verify installation
docker --version
```

### macOS

#### Using Docker Desktop
1. Download [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)
2. Install and launch the application
3. Follow the setup wizard

#### Verify Installation
```bash
docker --version
docker-compose --version
```

### Windows

#### Using Docker Desktop
1. Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)
2. Install and launch the application
3. Follow the setup wizard

#### Prerequisites
- Windows 10 64-bit: Pro, Enterprise, or Education (Build 16299 or later)
- Windows 11 64-bit: Home, Pro, Enterprise, or Education
- Hardware virtualization enabled in BIOS
- WSL 2 (Windows Subsystem for Linux) enabled

## Setup and Configuration

### Initial Setup

1. **Clone the repository:**
```bash
git clone <your-repo-url>
cd <your-repo-name>
```

2. **Configure environment variables:**
```bash
# Create .env file from example
cp example.env .env

# Edit the .env file with your preferred text editor
nano .env
```

3. **Verify Docker installation:**
```bash
docker --version
docker-compose --version
```

### Docker Engine vs Docker Desktop

#### Docker Engine (Linux Only)
- Lightweight command-line interface
- No GUI components
- Lower resource usage
- Direct control over daemon

#### Docker Desktop
- Integrated GUI for container management
- Built-in Kubernetes support
- Resource management controls
- Windows and macOS support

### Docker Compose Plugin

#### Using Docker Compose Plugin (Recommended)
```bash
# Docker Compose V2 is included in Docker Desktop
# On Linux, it may need to be installed separately
sudo apt install docker-compose-plugin

# Verify installation
docker compose version
```

#### Using Standalone Docker Compose V1
```bash
# Install Docker Compose V1 (legacy)
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Configuration Files

#### Docker Daemon Configuration
```bash
# Edit Docker daemon configuration
sudo nano /etc/docker/daemon.json

# Example configuration
{
  "registry-mirrors": ["https://mirror.gcr.io"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}

# Restart Docker daemon
sudo systemctl restart docker
```

#### Environment Variables
```bash
# Set environment variables for Docker
export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1
```

## Starting and Managing Services

### Basic Commands

#### Start All Services
```bash
# From the project root directory
docker compose up -d
```

#### Start Specific Services
```bash
# Start individual services
docker compose up -d postgres redis mongo

# Start a service group
docker compose up -d postgres-cluster
```

#### View Running Services
```bash
docker compose ps
```

#### View Service Logs
```bash
# View all logs
docker compose logs -f

# View specific service logs
docker compose logs -f postgres

# View last 100 lines
docker compose logs --tail=100 postgres
```

#### Stop Services
```bash
# Stop all services
docker compose down

# Stop specific services
docker compose down postgres redis

# Stop and remove volumes (data will be lost)
docker compose down --volumes
```

### Service Health Checks

#### Check Service Status
```bash
docker compose ps
```

#### Manual Health Check
```bash
# For PostgreSQL
docker compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# For Redis
docker compose exec redis redis-cli ping

# For MongoDB
docker compose exec mongo mongo --eval "db.runCommand('ping')"
```

### Service Restart Policies

#### Automatic Restart
Services are configured with `restart: unless-stopped` by default:
```bash
# Service will restart automatically if it fails
docker compose up -d
```

#### Manual Restart
```bash
# Restart a specific service
docker compose restart postgres

# Restart all services
docker compose restart
```

## Volume Management

### Volume Locations

#### Default Docker Volume Path
```bash
# Volume storage location
/var/lib/docker/volumes/

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect db-compose_postgres_data
```

#### Custom Volume Path
```bash
# Edit Docker daemon configuration to change volume path
sudo nano /etc/docker/daemon.json

{
  "data-root": "/path/to/custom/docker/data"
}

# Restart Docker
sudo systemctl restart docker
```

### Volume Backup and Restore

#### Backup Volumes
```bash
# Backup a specific volume
docker run --rm \
  -v db-compose_postgres_data:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_backup.tar.gz -C /source .

# Backup all project volumes
docker run --rm \
  -v $(docker volume ls -q | grep db-compose):/source \
  -v $(pwd):/backup \
  alpine tar czf /backup/full_backup.tar.gz -C /source .
```

#### Restore Volumes
```bash
# Create a new volume
docker volume create db-compose_postgres_data

# Restore data
docker run --rm \
  -v db-compose_postgres_data:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres_backup.tar.gz -C /target
```

### Volume Persistence

#### Data Persistence Across Restarts
```bash
# Stop services but keep volumes
docker compose down

# Restart services (data persists)
docker compose up -d
```

#### Cleanup Unused Volumes
```bash
# Remove unused volumes
docker volume prune

# Remove all volumes (use with caution!)
docker volume rm $(docker volume ls -q)
```

### Volume Drivers

#### Named Volumes
```yaml
# Default volume configuration
volumes:
  postgres_data:
    driver: local
```

#### NFS Volume Driver
```bash
# Install NFS volume plugin
docker plugin install --grant-all-permissions vieux/sshfs

# Use NFS volume
volumes:
  nfs_data:
    driver: vieux/sshfs
    driver_opts:
      sshcmd: "user@nfs-server:/path/to/share"
      password: "password"
```

## Network Configuration

### Shared Network

All services use a shared network called `ct_shared_network`:

```bash
# View network information
docker network inspect ct_shared_network

# List all networks
docker network ls
```

### Service Discovery

Services can communicate using their service names:

```bash
# From within a container
ping postgres
ping redis
ping mongo
```

### Port Mapping

#### Default Port Mappings

| Service | Container Port | Host Port |
|---------|---------------|-----------|
| PostgreSQL | 5432 | 5432 |
| Redis | 6379 | 6380 |
| MongoDB | 27017 | 27017 |
| MariaDB | 3306 | 3306 |
| Kafka | 9092 | 9092 |
| RabbitMQ | 5672 | 5672 |
| NATS | 4222 | 4222 |
| ClickHouse | 9000 | 9000 |
| ScyllaDB | 9042 | 9042 |
| Valkey | 6379 | 6381 |
| DBGate | 30080 | 30080 |

#### Custom Port Mapping
```bash
# Edit compose.yaml files to change port mappings
ports:
  - "5433:5432"  # Map PostgreSQL to 5433 on host
```

### Network Drivers

#### Bridge Network (Default)
```yaml
networks:
  ct_shared_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

#### Overlay Network (Swarm Mode)
```bash
# Initialize Swarm mode
docker swarm init

# Create overlay network
docker network create --driver=overlay --attachable ct_shared_network

# Use in compose
networks:
  ct_shared_network:
    external: true
```

### Network Troubleshooting

#### Check Network Connectivity
```bash
# Test connectivity between services
docker compose exec postgres ping redis

# Check DNS resolution
docker compose exec postgres nslookup postgres
```

#### Inspect Network Details
```bash
# View network configuration
docker network inspect ct_shared_network

# Check container network settings
docker inspect postgres | grep -A 10 "NetworkSettings"
```

## Common Workflows

### Development Workflow

1. **Start development environment:**
```bash
# Start core databases
docker compose up -d postgres redis mongo

# Start messaging services
docker compose up -d kafka rabbitmq nats

# Start database client
docker compose up -d dbgate
```

2. **Access databases:**
```bash
# PostgreSQL
psql -h localhost -p 5432 -U ${DB_USERNAME} -d ${DB_NAME}

# Redis
redis-cli -h localhost -p 6380 -a ${DB_PASSWORD}

# MongoDB
mongo --host localhost --port 27017 -u ${DB_USERNAME} -p ${DB_PASSWORD} ${DB_NAME}
```

3. **Stop development environment:**
```bash
# Stop all services but keep data
docker compose down
```

### Testing Workflow

1. **Start specific service group:**
```bash
# Test cluster services
docker compose up -d postgres-cluster

# Test messaging services
docker compose up -d kafka rabbitmq nats

# Test NoSQL services
docker compose up -d mongo scylladb clickhouse
```

2. **Run tests:**
```bash
# Example: Test PostgreSQL cluster
docker compose exec postgres-primary pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Example: Test Redis cluster
docker compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info
```

3. **Cleanup:**
```bash
# Stop and remove test data
docker compose down --volumes
```

### Production Workflow

1. **Start services with resource limits:**
```bash
# Create a production override file
cat > production-compose.yaml << EOF
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 2G
EOF

# Start with production settings
docker compose -f production-compose.yaml up -d
```

2. **Monitor services:**
```bash
# View resource usage
docker stats

# Check logs periodically
docker compose logs --tail=100 -f
```

3. **Graceful shutdown:**
```bash
# Stop services gracefully
docker compose down
```

## Docker-Specific Features

### Docker Swarm Mode

#### Initialize Swarm
```bash
# Initialize Docker Swarm
docker swarm init

# Create overlay network
docker network create --driver=overlay --attachable ct_shared_network
```

#### Deploy to Swarm
```bash
# Deploy services to Swarm
docker stack deploy -c docker-compose.yml db-compose

# View services
docker service ls

# Scale services
docker service scale db-compose_postgres=3
```

### Docker Compose Profiles

#### Define Profiles
```yaml
services:
  postgres:
    profiles:
      - database
    image: postgres:18-alpine
  
  redis:
    profiles:
      - database
      - cache
    image: redis:8-alpine
  
  web:
    profiles:
      - app
    image: nginx:alpine
```

#### Use Profiles
```bash
# Start services with specific profiles
docker compose --profile database up -d

# Start services with multiple profiles
docker compose --profile database --profile cache up -d

# Start all services
docker compose --profile all up -d
```

### Docker Build Integration

#### Build Custom Images
```yaml
services:
  custom-postgres:
    build:
      context: ./custom-postgres
      dockerfile: Dockerfile
      args:
        POSTGRES_VERSION: 18
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
```

#### Build and Run
```bash
# Build custom images
docker compose build

# Build with no cache
docker compose build --no-cache

# Run specific service with custom build
docker compose run --rm custom-postgres psql -c "SELECT version();"
```

### Docker Secrets Management

#### Define Secrets
```yaml
services:
  postgres:
    secrets:
      - db_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

secrets:
  db_password:
    external: true
```

#### Manage Secrets
```bash
# Create secret
echo "your_password" | docker secret create db_password -

# List secrets
docker secret ls

# Remove secret
docker secret rm db_password
```

## Docker Desktop Integration

### Docker Desktop Dashboard

#### Access Dashboard
1. Open Docker Desktop
2. Click the "Dashboard" tab
3. View all running containers

#### Manage Containers via GUI
- Start/stop containers
- View logs
- Inspect containers
- Manage volumes and networks

### Resource Management

#### Configure Resources
1. Open Docker Desktop
2. Go to "Resources" tab
3. Adjust CPU, memory, and disk space limits

#### View Resource Usage
```bash
# View container resource usage
docker stats

# View detailed container information
docker inspect postgres | grep -A 10 "HostConfig"
```

### Kubernetes Integration

#### Enable Kubernetes
1. Open Docker Desktop
2. Go to "Kubernetes" tab
3. Enable "Enable Kubernetes"

#### Deploy to Kubernetes
```bash
# Convert Docker Compose to Kubernetes manifests
kompose convert -f compose.yaml

# Apply manifests
kubectl apply -f ./k8s/
```

## Advanced Usage

### Custom Compose Files

#### Override Default Configuration
```bash
# Create an override file
cat > override.yaml << EOF
services:
  postgres:
    environment:
      POSTGRES_PASSWORD: custom_password
    volumes:
      - custom_postgres_data:/var/lib/postgresql/data
EOF

# Start with overrides
docker compose -f override.yaml up -d
```

#### Environment-Specific Files
```bash
# Development
docker compose -f compose.yaml -f dev.yaml up -d

# Staging
docker compose -f compose.yaml -f staging.yaml up -d

# Production
docker compose -f compose.yaml -f prod.yaml up -d
```

### Resource Management

#### CPU and Memory Limits
```yaml
# Add to service definition
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
```

#### Apply Limits
```bash
# Apply resource limits
docker compose up -d

# Monitor resource usage
docker stats --no-stream
```

### Health Check Customization

#### Custom Health Checks
```yaml
services:
  postgres:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
```

#### Check Health Status
```bash
# View health status
docker compose ps

# Manual health check
docker compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}
```

### Logging Configuration

#### Structured Logging
```bash
# Enable JSON logging
docker compose logs --format json

# Filter logs
docker compose logs --filter "level=error"
```

#### Log Drivers
```yaml
# Add to service definition
services:
  postgres:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## Security Considerations

### Docker Daemon Security

#### Secure Docker Daemon
```bash
# Restrict Docker daemon access
sudo nano /etc/docker/daemon.json

{
  "group": "docker"
}

# Restart Docker
sudo systemctl restart docker
```

#### Docker Socket Permissions
```bash
# Check Docker socket permissions
ls -la /var/run/docker.sock

# Restrict access to docker group
sudo chown root:docker /var/run/docker.sock
```

### Container Security

#### Run as Non-Root User
```yaml
services:
  postgres:
    user: "999:999"  # postgres user
```

#### Read-Only Root Filesystem
```yaml
services:
  postgres:
    read_only: true
    tmpfs:
      - /var/lib/postgresql/data
```

### Image Security

#### Verify Image Signatures
```bash
# Check image signatures
docker image inspect --format '{{.Id}}' postgres:18-alpine

# Verify image digest
docker manifest inspect mirror.gcr.io/postgresql:18-alpine
```

#### Scan for Vulnerabilities
```bash
# Use Trivy for vulnerability scanning
docker run --rm -v /var/lib/libvirt:/var/lib/libvirt:z \
  -v /tmp:/tmp:z \
  aquasec/trivy:latest image db-compose_postgres
```

### Network Security

#### Network Isolation
```bash
# Create custom network
docker network create --internal db-internal

# Use in compose
services:
  postgres:
    networks:
      - db-internal
```

#### Firewall Rules
```bash
# Allow only necessary ports
sudo firewall-cmd --permanent --add-port=5432/tcp
sudo firewall-cmd --permanent --add-port=6380/tcp
sudo firewall-cmd --reload
```

## Docker Compose V1 vs V2

### Docker Compose V1 (Legacy)

#### Installation
```bash
# Install Docker Compose V1
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### Usage
```bash
# Legacy command syntax
docker-compose up -d
docker-compose down
docker-compose logs
```

### Docker Compose V2 (Recommended)

#### Installation
```bash
# Docker Compose V2 is included in Docker Desktop
# On Linux, it may need to be installed separately
sudo apt install docker-compose-plugin
```

#### Usage
```bash
# Modern command syntax
docker compose up -d
docker compose down
docker compose logs
```

### Migration from V1 to V2

#### Command Changes
| V1 Command | V2 Command |
|------------|------------|
| `docker-compose` | `docker compose` |
| `docker-compose up` | `docker compose up` |
| `docker-compose down` | `docker compose down` |
| `docker-compose logs` | `docker compose logs` |
| `docker-compose ps` | `docker compose ps` |

#### Configuration Changes
```yaml
# V1 syntax
version: '3'

# V2 syntax (version is optional)
services:
  postgres:
    image: postgres:18-alpine
```

## Performance Optimization

### Docker Daemon Performance

#### Storage Driver Optimization
```bash
# Check current storage driver
docker info | grep "Storage Driver"

# Configure overlay2 storage driver
sudo nano /etc/docker/daemon.json

{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.size=10G"
  ]
}

# Restart Docker
sudo systemctl restart docker
```

#### Network Optimization
```bash
# Configure bridge network
sudo nano /etc/docker/daemon.json

{
  "bridge": "none",
  "fixed-cidr": "172.20.0.0/16",
  "fixed-cidr-v6": "fd00:dead:beef::/64",
  "mtu": 1500
}
```

### Container Performance

#### Resource Limits
```yaml
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '0.5'
          memory: 1G
```

#### I/O Limits
```yaml
services:
  postgres:
    deploy:
      placement:
        constraints:
          - node.labels.storage == ssd
      resources:
        reservations:
          devices:
            - driver: block
              count: 1
              capabilities: [gpu]
```

### Image Optimization

#### Multi-stage Builds
```dockerfile
# Stage 1: Build
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o db-service .

# Stage 2: Runtime
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/db-service .
CMD ["./db-service"]
```

#### Layer Caching
```dockerfile
# Order instructions from least to most frequently changed
FROM alpine:latest
RUN apk add --no-cache ca-certificates
COPY ./app /app
COPY ./config /config
CMD ["./app"]
```

## Troubleshooting Common Issues

### Permission Denied Errors

#### Docker Group Membership
```bash
# Check Docker group membership
groups

# Add user to docker group
sudo usermod -aG docker $USER

# Activate group changes
newgrp docker
```

#### Volume Permission Issues
```bash
# Fix volume permissions
docker run --rm -v db-compose_postgres_data:/data alpine chown -R 999:999 /data
```

### Network Connectivity Issues

#### DNS Resolution Problems
```bash
# Test DNS resolution
docker compose exec nslookup postgres

# Check network settings
docker network inspect ct_shared_network
```

#### Port Conflicts
```bash
# Check port usage
netstat -tlnp | grep 5432

# Change port mapping in compose.yaml
ports:
  - "5433:5432"  # Use different host port
```

### Service Startup Failures

#### Check Container Logs
```bash
# View detailed logs
docker compose logs postgres

# Follow logs in real-time
docker compose logs -f postgres
```

#### Health Check Failures
```bash
# Manual health check
docker compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Check health status
docker compose ps --format "table {{.Service}}\t{{.Status}}"
```

### Image Pull Issues

#### Registry Configuration
```bash
# Configure registry mirrors
sudo nano /etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://mirror.gcr.io",
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ]
}

# Restart Docker
sudo systemctl restart docker
```

#### Manual Image Pull
```bash
# Pull specific images
docker pull mirror.gcr.io/postgresql:18-alpine
docker pull mirror.gcr.io/redis:8-alpine

# Force pull images
docker compose pull --force-recreate
```

## Conclusion

This guide covers the essential aspects of using db-compose with Docker. Docker provides a robust, feature-rich platform for containerizing database services with extensive tooling and community support.

For additional help:
- Check the [Docker documentation](https://docs.docker.com/)
- Review the [Docker Compose documentation](https://docs.docker.com/compose/)
- Create an issue in the db-compose repository for specific problems

Happy containerizing!