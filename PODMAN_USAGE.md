# Podman Usage Guide for db-compose

This guide provides comprehensive instructions for using the db-compose project with Podman, a daemonless container runtime.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Setup and Configuration](#setup-and-configuration)
- [Starting and Managing Services](#starting-and-managing-services)
- [Volume Management](#volume-management)
- [Network Configuration](#network-configuration)
- [Common Workflows](#common-workflows)
- [Podman-Specific Features](#podman-specific-features)
- [Advanced Usage](#advanced-usage)
- [Security Considerations](#security-considerations)

## Overview

Podman is a daemonless container engine for developing, managing, and running OCI (Open Container Initiative) containers. Unlike Docker, Podman doesn't require a background daemon, making it more lightweight and secure.

### Benefits of Using Podman with db-compose

- **Rootless by default**: Containers run as your user without requiring root privileges
- **No daemon**: No background service to manage or monitor
- **Better security**: Enhanced security model with SELinux integration
- **Podman Compose**: Native support for Docker Compose files
- **Kubernetes compatibility**: Direct integration with Kubernetes

## Installation

### Linux (Recommended)

#### Package Manager Installation

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install -y podman
```

**Fedora/CentOS/RHEL:**
```bash
sudo dnf install -y podman
```

**Arch Linux:**
```bash
sudo pacman -S podman
```

#### Podman Compose Installation

```bash
# Using pip (Python package)
pip install podman-compose

# Or using curl
curl -sSL https://github.com/containers/podman-compose/raw/main/podman-compose.py -o /usr/local/bin/podman-compose
chmod +x /usr/local/bin/podman-compose
```

### macOS

#### Using Homebrew
```bash
brew install podman
```

#### Podman Compose
```bash
pip install podman-compose
```

### Windows

#### Using Podman Desktop
1. Download [Podman Desktop](https://podman.io/getting-started/installation)
2. Install and launch the application
3. Follow the setup wizard

#### Podman Compose
```bash
pip install podman-compose
```

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

3. **Verify Podman installation:**
```bash
podman --version
podman-compose --version
```

### Rootless vs Rootful Mode

Podman can run in two modes:

#### Rootless Mode (Default)
- Containers run as your user
- No need for `sudo` for most operations
- Safer default configuration
- Volume paths: `~/.local/share/containers/storage/volumes/`

#### Rootful Mode
- Containers run as root
- Requires `sudo` for operations
- May be needed for specific applications
- Volume paths: `/var/lib/containers/storage/volumes/`

**To switch to rootful mode (if needed):**
```bash
# Install Podman as root
sudo dnf install -y podman

# Or enable rootful socket
sudo systemctl enable --now podman.socket
```

### SELinux Configuration

On SELinux-enabled systems (RHEL, Fedora, CentOS):

#### Check SELinux Status
```bash
sestatus
```

#### Permissive Mode (Recommended for Development)
```bash
sudo setenforce 0
```

#### Enforcing Mode (Production)
```bash
# May need to add volume mount labels
# Edit compose.yaml files to include :Z or :z suffix
volumes:
  - postgres_data:/var/lib/postgresql/data:Z
```

## Starting and Managing Services

### Basic Commands

#### Start All Services
```bash
# From the project root directory
podman compose up -d
```

#### Start Specific Services
```bash
# Start individual services
podman compose up -d postgres redis mongo

# Start a service group
podman compose up -d postgres-cluster
```

#### View Running Services
```bash
podman compose ps
```

#### View Service Logs
```bash
# View all logs
podman compose logs -f

# View specific service logs
podman compose logs -f postgres

# View last 100 lines
podman compose logs --tail=100 postgres
```

#### Stop Services
```bash
# Stop all services
podman compose down

# Stop specific services
podman compose down postgres redis

# Stop and remove volumes (data will be lost)
podman compose down --volumes
```

### Service Health Checks

#### Check Service Status
```bash
podman compose ps
```

#### Manual Health Check
```bash
# For PostgreSQL
podman compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# For Redis
podman compose exec redis redis-cli ping

# For MongoDB
podman compose exec mongo mongo --eval "db.runCommand('ping')"
```

### Service Restart Policies

#### Automatic Restart
Services are configured with `restart: unless-stopped` by default:
```bash
# Service will restart automatically if it fails
podman compose up -d
```

#### Manual Restart
```bash
# Restart a specific service
podman compose restart postgres

# Restart all services
podman compose restart
```

## Volume Management

### Volume Locations

#### Rootless Podman
```bash
# Volume storage location
~/.local/share/containers/storage/volumes/

# List volumes
podman volume ls

# Inspect a volume
podman volume inspect db-compose_postgres_data
```

#### Rootful Podman
```bash
# Volume storage location
/var/lib/containers/storage/volumes/

# List volumes
sudo podman volume ls

# Inspect a volume
sudo podman volume inspect db-compose_postgres_data
```

### Volume Backup and Restore

#### Backup Volumes
```bash
# Backup a specific volume
podman run --rm \
  -v db-compose_postgres_data:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_backup.tar.gz -C /source .

# Backup all project volumes
podman run --rm \
  -v $(podman volume ls -q | grep db-compose):/source \
  -v $(pwd):/backup \
  alpine tar czf /backup/full_backup.tar.gz -C /source .
```

#### Restore Volumes
```bash
# Create a new volume
podman volume create db-compose_postgres_data

# Restore data
podman run --rm \
  -v db-compose_postgres_data:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres_backup.tar.gz -C /target
```

### Volume Persistence

#### Data Persistence Across Restarts
```bash
# Stop services but keep volumes
podman compose down

# Restart services (data persists)
podman compose up -d
```

#### Cleanup Unused Volumes
```bash
# Remove unused volumes
podman volume prune

# Remove all volumes (use with caution!)
podman volume rm $(podman volume ls -q)
```

## Network Configuration

### Shared Network

All services use a shared network called `ct_shared_network`:

```bash
# View network information
podman network inspect ct_shared_network

# List all networks
podman network ls
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

### Network Troubleshooting

#### Check Network Connectivity
```bash
# Test connectivity between services
podman compose exec postgres ping redis

# Check DNS resolution
podman compose exec postgres nslookup postgres
```

#### Inspect Network Details
```bash
# View network configuration
podman network inspect ct_shared_network

# Check container network settings
podman inspect postgres | grep -A 10 "NetworkSettings"
```

## Common Workflows

### Development Workflow

1. **Start development environment:**
```bash
# Start core databases
podman compose up -d postgres redis mongo

# Start messaging services
podman compose up -d kafka rabbitmq nats

# Start database client
podman compose up -d dbgate
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
podman compose down
```

### Testing Workflow

1. **Start specific service group:**
```bash
# Test cluster services
podman compose up -d postgres-cluster

# Test messaging services
podman compose up -d kafka rabbitmq nats

# Test NoSQL services
podman compose up -d mongo scylladb clickhouse
```

2. **Run tests:**
```bash
# Example: Test PostgreSQL cluster
podman compose exec postgres-primary pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Example: Test Redis cluster
podman compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info
```

3. **Cleanup:**
```bash
# Stop and remove test data
podman compose down --volumes
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
podman compose -f production-compose.yaml up -d
```

2. **Monitor services:**
```bash
# View resource usage
podman stats

# Check logs periodically
podman compose logs --tail=100 -f
```

3. **Graceful shutdown:**
```bash
# Stop services gracefully
podman compose down
```

## Podman-Specific Features

### Podman Systemd Integration

#### Create Systemd Service
```bash
# Create a systemd service file
sudo tee /etc/systemd/system/db-compose.service << EOF
[Unit]
Description=Database Compose Services
Requires=podman.socket
After=podman.socket

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/path/to/db-compose
ExecStart=/usr/local/bin/podman-compose up -d
ExecStop=/usr/local/bin/podman-compose down

[Install]
WantedBy=default.target
EOF

# Enable and start the service
sudo systemctl enable --now db-compose
```

#### Manage with Systemd
```bash
# Check service status
systemctl status db-compose

# Start/stop services
sudo systemctl start db-compose
sudo systemctl stop db-compose

# View logs
journalctl -u db-compose -f
```

### Podman Desktop Integration

#### Connect to Podman Desktop
1. Launch Podman Desktop
2. Go to "Containers" view
3. You'll see all db-compose services listed

#### Manage via GUI
- Start/stop containers
- View logs
- Inspect containers
- Manage volumes and networks

### Rootless Container Management

#### User Namespace Mapping
```bash
# Check current user namespace mapping
podman info | grep -A 5 "userns"

# Create custom user mapping (if needed)
sudo touch /etc/subuid
sudo touch /etc/subgid
```

#### Running as Different User
```bash
# Run containers as specific user
podman run --user nobody alpine echo "Running as nobody"
```

### Podman Pod Support

#### Create a Pod
```bash
# Create a pod for related services
podman pod create --name db-pod --publish 5432:5432 --publish 6379:6379

# Start services in the pod
podman compose --pod db-pod up -d

# View pod information
podman pod ps
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
podman compose -f override.yaml up -d
```

#### Environment-Specific Files
```bash
# Development
podman compose -f compose.yaml -f dev.yaml up -d

# Staging
podman compose -f compose.yaml -f staging.yaml up -d

# Production
podman compose -f compose.yaml -f prod.yaml up -d
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
podman compose up -d

# Monitor resource usage
podman stats --no-stream
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
podman compose ps

# Manual health check
podman compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}
```

### Logging Configuration

#### Structured Logging
```bash
# Enable JSON logging
podman compose logs --format json

# Filter logs
podman compose logs --filter "level=error"
```

#### Log Rotation
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

### Rootless Security

#### Running as Non-Root User
```bash
# Verify running as non-root
podman compose exec postgres whoami

# Should output: "postgres" (not "root")
```

#### User Namespaces
```bash
# Check user namespace configuration
podman info | grep userns

# Enable user namespace mapping
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subuid
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subgid
```

### Container Security

#### Read-Only Root Filesystem
```yaml
services:
  postgres:
    read_only: true
    tmpfs:
      - /var/lib/postgresql/data
```

#### Security Options
```yaml
services:
  postgres:
    security_opt:
      - no-new-privileges:true
      - apparmor:docker-default
```

### Image Security

#### Verify Image Signatures
```bash
# Check image signatures
podman image inspect --format '{{.Id}}' postgres:18-alpine

# Verify image digest
podman manifest inspect mirror.gcr.io/postgresql:18-alpine
```

#### Scan for Vulnerabilities
```bash
# Use Trivy for vulnerability scanning
podman run --rm -v /var/lib/libvirt:/var/lib/libvirt:z \
  -v /tmp:/tmp:z \
  aquasec/trivy:latest image db-compose_postgres
```

### Network Security

#### Network Isolation
```bash
# Create custom network
podman network create --internal db-internal

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

## Troubleshooting Common Issues

### Permission Denied Errors

#### Volume Permission Issues
```bash
# Fix volume permissions
podman run --rm -v db-compose_postgres_data:/data alpine chown -R 999:999 /data
```

#### User Namespace Issues
```bash
# Reset user namespace mapping
podman system reset
```

### Network Connectivity Issues

#### DNS Resolution Problems
```bash
# Test DNS resolution
podman compose exec nslookup postgres

# Check network settings
podman network inspect ct_shared_network
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
podman compose logs postgres

# Follow logs in real-time
podman compose logs -f postgres
```

#### Health Check Failures
```bash
# Manual health check
podman compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Check health status
podman compose ps --format "table {{.Service}}\t{{.Status}}"
```

### Performance Issues

#### Resource Monitoring
```bash
# Monitor resource usage
podman stats --no-stream

# Check container limits
podman inspect postgres | grep -A 10 "Resources"
```

#### Performance Tuning
```bash
# Increase shared memory for PostgreSQL
podman compose exec postgres sysctl -w kernel.shmmax=4294967295
```

## Conclusion

This guide covers the essential aspects of using db-compose with Podman. Podman provides a secure, daemonless alternative to Docker while maintaining compatibility with Docker Compose files. 

For additional help:
- Check the [Podman documentation](https://podman.io/)
- Review the [Podman Compose documentation](https://github.com/containers/podman-compose)
- Create an issue in the db-compose repository for specific problems

Happy containerizing!