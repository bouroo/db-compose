# Docker to Podman Migration Guide

**Version:** 1.0  
**Date:** 2025-11-16  
**Target Audience:** Users migrating from Docker to Podman for db-compose project  

---

## Table of Contents

1. [Overview](#overview)
2. [Pre-Migration Checklist](#pre-migration-checklist)
3. [Migration Process](#migration-process)
4. [Data Migration](#data-migration)
5. [Post-Migration Verification](#post-migration-verification)
6. [Rollback Procedures](#rollback-procedures)
7. [Common Migration Scenarios](#common-migration-scenarios)
8. [Service-Specific Migration Notes](#service-specific-migration-notes)
9. [Troubleshooting](#troubleshooting)
10. [Performance Considerations](#performance-considerations)

---

## Overview

This guide provides comprehensive step-by-step instructions for migrating from Docker to Podman while maintaining full functionality of the db-compose project. The migration is designed to be non-disruptive, allowing for easy rollback if needed.

### Key Benefits of Podman

- **Daemonless operation**: No background process required
- **Rootless containers**: Enhanced security by default
- **Better integration with Linux security features**: SELinux support
- **Kubernetes compatibility**: Direct pod support
- **Lower resource footprint**: Reduced memory and CPU usage

### Migration Approach

This guide follows a **dual-runtime strategy** where both Docker and Podman can coexist on the same system, allowing for a gradual migration with minimal risk.

---

## Pre-Migration Checklist

### System Requirements

- ✅ **Operating System**: Linux, macOS, or Windows (WSL 2)
- ✅ **Disk Space**: Minimum 5GB free space for containers and volumes
- ✅ **RAM**: Minimum 4GB recommended for full stack
- ✅ **Network**: Internet connection for image pulls
- ✅ **User Permissions**: Sudo access for system installations (if needed)

### Software Requirements

- ✅ **Podman**: Version 5.0 or higher
- ✅ **Podman Compose**: Version 1.0 or higher
- ✅ **Docker** (optional, for coexistence): Version 20.10 or higher
- ✅ **Docker Compose Plugin** (if using Docker): Version 2.0 or higher

### Pre-Migration Verification

#### Check Current Docker Environment

```bash
# Verify Docker is installed and working
docker --version
docker compose version

# Check running Docker services
docker ps

# Verify db-compose services with Docker
docker compose -f compose.yaml ps
```

#### Backup Existing Data

```bash
# Create backup directory
mkdir -p migration-backups/$(date +%Y%m%d-%H%M%S)

# Backup Docker volumes
docker run --rm \
  -v $(docker volume ls -q | grep db-compose):/source \
  -v $(pwd)/migration-backups/$(date +%Y%m%d-%H%M%S):/backup \
  alpine tar czf /backup/docker_volumes_backup.tar.gz -C /source .

# Backup configuration files
cp -r .env example.env migration-backups/$(date +%Y%m%d-%H%M%S)/

# Document current service status
docker compose -f compose.yaml ps > migration-backups/$(date +%Y%m%d-%H%M%S)/docker_services_status.txt
```

#### Verify Image Registry Access

```bash
# Test mirror.gcr.io access (current configuration)
docker pull mirror.gcr.io/postgresql:18-alpine

# If mirror fails, test official registry
docker pull docker.io/library/postgres:18-alpine

# Note: Some mirror.gcr.io tags may not be available
```

### Environment Preparation

#### Install Podman

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install -y podman podman-compose
```

**Linux (Fedora/CentOS/RHEL):**
```bash
sudo dnf install -y podman podman-compose
```

**macOS (using Homebrew):**
```bash
brew install podman
pip install podman-compose
```

**Windows (using Podman Desktop):**
1. Download [Podman Desktop](https://podman.io/getting-started/installation)
2. Install and launch the application
3. Follow the setup wizard

#### Verify Podman Installation

```bash
# Check Podman version
podman --version
podman-compose --version

# Test basic Podman functionality
podman run --rm alpine echo "Podman is working"
```

#### Prepare Environment File

```bash
# Copy environment template
cp example.env .env

# Edit environment variables if needed
nano .env
```

---

## Migration Process

### Step 1: Install Podman and Dependencies

#### Install Podman

```bash
# For Ubuntu/Debian
sudo apt update
sudo apt install -y podman podman-compose

# For Fedora/CentOS/RHEL
sudo dnf install -y podman podman-compose

# For Arch Linux
sudo pacman -S podman podman-compose
```

#### Install Podman Compose

```bash
# Using pip (recommended)
pip install podman-compose

# Or using curl
curl -sSL https://github.com/containers/podman-compose/raw/main/podman-compose.py -o /usr/local/bin/podman-compose
chmod +x /usr/local/bin/podman-compose
```

### Step 2: Update Image Registry References

The current configuration uses `mirror.gcr.io` which may not have all required tags. We need to update to official registries.

#### Update Compose Files

Create a script to update image references:

```bash
#!/bin/bash
# update_images.sh

# Create backup of original files
mkdir -p backup-original
cp databases/*/compose.yaml backup-original/

# Update image references in all compose files
find databases -name "compose.yaml" -type f -exec sed -i 's|mirror\.gcr\.io/||g' {} \;

# Replace rapidfort images with official ones
find databases -name "compose.yaml" -type f -exec sed -i 's|rapidfort/||g' {} \;

echo "Image references updated successfully"
```

Run the script:
```bash
chmod +x update_images.sh
./update_images.sh
```

#### Verify Updated Images

```bash
# Test pulling updated images
podman pull docker.io/library/postgres:18-alpine
podman pull docker.io/library/redis:8-alpine
podman pull docker.io/library/mongo:8
```

### Step 3: SELinux Configuration (Linux Systems)

Check if SELinux is enabled on your system:

```bash
# Check SELinux status
sestatus

# If in enforcing mode, prepare for Podman
if [ "$(getenforce)" = "Enforcing" ]; then
    echo "SELinux is in enforcing mode"
    echo "You may need to add :Z suffix to volume mounts in compose files"
fi
```

For SELinux enforcing systems, you may need to update volume mounts:

```yaml
# Example with SELinux label
volumes:
  - postgres_data:/var/lib/postgresql/data:Z
```

### Step 4: Test Podman with Basic Services

Start with a simple service to verify Podman works:

```bash
# Test basic PostgreSQL service
podman compose -f databases/postgres/compose.yaml up -d

# Verify service is running
podman compose -f databases/postgres/compose.yaml ps

# Test connectivity
podman compose -f databases/postgres/compose.yaml exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Clean up
podman compose -f databases/postgres/compose.yaml down
```

### Step 5: Migrate Service Groups

#### Basic Services

```bash
# Start basic database services
podman compose -f compose.yaml up -d postgres redis mongo

# Verify services are running
podman compose -f compose.yaml ps

# Test inter-service communication
podman compose -f compose.yaml exec postgres ping redis
podman compose -f compose.yaml exec redis ping mongo

# Check service logs
podman compose -f compose.yaml logs postgres
```

#### Messaging Services

```bash
# Start messaging services
podman compose -f compose.yaml up -d kafka rabbitmq nats

# Verify messaging services
podman compose -f compose.yaml exec kafka kafka-topics --list --bootstrap-server localhost:9092
podman compose -f compose.yaml exec rabbitmq rabbitmqctl list_connections
podman compose -f compose.yaml exec nats nats stream ls
```

#### Cluster Services

**PostgreSQL Cluster:**
```bash
# Start PostgreSQL cluster
podman compose -f compose.yaml up -d postgres-cluster

# Verify cluster formation
podman compose -f compose.yaml exec postgres-primary psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT version();"
podman compose -f compose.yaml exec postgres-replica-1 psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT pg_is_in_recovery();"

# Test read/write splitting
podman compose -f compose.yaml exec pgbouncer psql -h localhost -U ${DB_USERNAME} -d primary -c "SELECT 1;"
```

**Redis Cluster:**
```bash
# Start Redis cluster
podman compose -f compose.yaml up -d redis-cluster

# Verify cluster status
podman compose -f compose.yaml exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info

# Test data distribution
podman compose -f compose.yaml exec redis-node-0 redis-cli -a ${DB_PASSWORD} -c set key1 value1
podman compose -f compose.yaml exec redis-node-1 redis-cli -a ${DB_PASSWORD} -c get key1
```

**Valkey Cluster:**
```bash
# Start Valkey cluster
podman compose -f compose.yaml up -d valkey-cluster

# Verify cluster status
podman compose -f compose.yaml exec valkey-node-0 valkey-cli -a ${DB_PASSWORD} cluster info
```

#### MariaDB Galera Cluster

```bash
# Start MariaDB Galera cluster
podman compose -f compose.yaml up -d mariadb-galera

# Verify cluster size
podman compose -f compose.yaml exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

# Test through MaxScale
podman compose -f compose.yaml exec maxscale mysql -h localhost -u common_user -p${DB_PASSWORD} -e "SELECT @@hostname;"
```

#### NoSQL Services

```bash
# Start NoSQL services
podman compose -f compose.yaml up -d mongo scylladb clickhouse

# Verify NoSQL services
podman compose -f compose.yaml exec mongo mongosh --eval "db.adminCommand('ping')"
podman compose -f compose.yaml exec scylla cqlsh -e "DESCRIBE KEYSPACE system"
podman compose -f compose.yaml exec clickhouse clickhouse-client --query "SELECT 1"
```

#### Tools and Utilities

```bash
# Start tools
podman compose -f compose.yaml up -d dbgate traefik

# Verify tools are accessible
curl -I http://localhost:30080
curl -I http://localhost:80
```

### Step 6: Validate All Services Together

```bash
# Start all services
podman compose -f compose.yaml up -d

# Check all services are running
podman compose -f compose.yaml ps

# Monitor resource usage
podman stats --no-stream

# Check for any errors
podman compose -f compose.yaml logs --tail=100

# Test connectivity between services
podman compose -f compose.yaml exec postgres ping redis
podman compose -f compose.yaml exec redis ping mongo
podman compose -f compose.yaml exec mongo ping kafka
```

---

## Data Migration

### Volume Migration

#### Understanding Volume Locations

**Docker Volume Path:**
```bash
# Docker volumes are stored in:
/var/lib/docker/volumes/

# List Docker volumes
docker volume ls
docker volume inspect db-compose_postgres_data
```

**Podman Volume Paths:**

**Rootless Podman (Default):**
```bash
# Podman volumes are stored in:
~/.local/share/containers/storage/volumes/

# List Podman volumes
podman volume ls
podman volume inspect db-compose_postgres_data
```

**Rootful Podman:**
```bash
# Rootful Podman volumes are stored in:
/var/lib/containers/storage/volumes/

# List Podman volumes (requires sudo)
sudo podman volume ls
sudo podman volume inspect db-compose_postgres_data
```

#### Data Migration Process

**Option 1: Direct Volume Copy (Recommended)**

```bash
# Create backup script
cat > migrate_volumes.sh << 'EOF'
#!/bin/bash

# Create backup directory
BACKUP_DIR="volume-backups-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup Docker volumes
echo "Backing up Docker volumes..."
docker run --rm \
  -v $(docker volume ls -q | grep db-compose):/source \
  -v "$BACKUP_DIR":/backup \
  alpine tar czf /backup/docker_volumes.tar.gz -C /source .

# Create new Podman volumes
echo "Creating Podman volumes..."
podman volume create $(docker volume ls -q | grep db-compose)

# Restore to Podman
echo "Restoring to Podman volumes..."
for volume in $(docker volume ls -q | grep db-compose); do
  echo "Restoring $volume..."
  podman run --rm \
    -v "$volume":/target \
    -v "$BACKUP_DIR":/backup \
    alpine tar xzf /backup/docker_volumes.tar.gz -C /target
done

echo "Volume migration completed"
EOF

# Make script executable and run
chmod +x migrate_volumes.sh
./migrate_volumes.sh
```

**Option 2: Service-Specific Data Export/Import**

For databases that support native export/import:

**PostgreSQL:**
```bash
# Export from Docker
docker compose exec postgres pg_dump -U ${DB_USERNAME} ${DB_NAME} > postgres_backup.sql

# Import to Podman
podman compose exec -T postgres psql -U ${DB_USERNAME} ${DB_NAME} < postgres_backup.sql
```

**Redis:**
```bash
# Export from Docker
docker compose exec redis redis-cli --rdb /data/redis_backup.rdb

# Copy to Podman
docker compose cp redis:/data/redis_backup.rdb ./
podman compose cp ./redis_backup.rdb redis:/data/

# Restart Redis to load new data
podman compose restart redis
```

**MongoDB:**
```bash
# Export from Docker
docker compose exec mongo mongodump --out /data/mongo_backup

# Copy to Podman
docker compose cp mongo:/data/mongo_backup ./
podman compose cp ./mongo_backup mongo:/data/

# Restore in Podman
podman compose exec mongorestore /data/mongo_backup
```

### Configuration Migration

#### Environment Variables

The environment variables in `.env` are compatible with both runtimes:

```bash
# Verify environment variables are loaded correctly
podman compose exec postgres env | grep DB_
podman compose exec redis env | grep DB_
```

#### Network Configuration

Podman uses the same network configuration as Docker:

```bash
# Verify network exists
podman network inspect ct_shared_network

# Test service discovery
podman compose exec postgres ping redis
```

#### Custom Configuration Files

If you have custom configuration files mounted as volumes:

```bash
# Copy custom configs to Podman volume
podman run --rm -v custom_configs:/target -v $(pwd)/custom:/source alpine cp -r /source/* /target/
```

---

## Post-Migration Verification

### Service Health Checks

#### Basic Services Verification

```bash
# PostgreSQL
podman compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Redis
podman compose exec redis redis-cli ping

# MongoDB
podman compose exec mongo mongosh --eval "db.adminCommand('ping')"
```

#### Cluster Services Verification

**PostgreSQL Cluster:**
```bash
# Check primary
podman compose exec postgres-primary psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT version();"

# Check replicas
podman compose exec postgres-replica-1 psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT pg_is_in_recovery();"

# Check replication lag
podman compose exec postgres-primary psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT pg_stat_replication;"

# Check connection pooler
podman compose exec pgbouncer psql -h localhost -U ${DB_USERNAME} -d primary -c "SELECT 1;"
```

**Redis Cluster:**
```bash
# Check cluster status
podman compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info

# Check cluster nodes
podman compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster nodes

# Test data operations
podman compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} set test-key test-value
podman compose exec redis-node-1 redis-cli -a ${DB_PASSWORD} get test-key
```

**MariaDB Galera:**
```bash
# Check cluster status
podman compose exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

# Check replication status
podman compose exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_local_state_comment';"
```

### Data Integrity Verification

#### Test Data Persistence

```bash
# Create test data
podman compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "CREATE TABLE test_table (id SERIAL, name TEXT);"
podman compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "INSERT INTO test_table (name) VALUES ('test_data');"

# Restart services
podman compose restart postgres

# Verify data persists
podman compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT * FROM test_table;"
```

#### Test Cross-Service Communication

```bash
# Test PostgreSQL to Redis
podman compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT 'Connecting to Redis...' AS connection_test;"
podman compose exec postgres ping redis

# Test Redis to MongoDB
podman compose exec redis ping mongo
podman compose exec mongo mongosh --eval "db.test.insertOne({message: 'From Redis'})"
```

### Performance Verification

#### Resource Usage Comparison

```bash
# Monitor resource usage
podman stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Compare with previous Docker usage (if available)
echo "Compare with previous Docker stats:"
echo "Docker: docker stats --no-stream"
echo "Podman: podman stats --no-stream"
```

#### Response Time Testing

```bash
# Test database response times
time podman compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}
time podman compose exec redis redis-cli ping
time podman compose exec mongo mongosh --eval "db.adminCommand('ping')"
```

### Network Connectivity Verification

#### Test Internal Network

```bash
# Test service discovery
podman compose exec postgres ping redis
podman compose exec redis ping mongo
podman compose exec mongo ping kafka

# Test port accessibility
curl -I http://localhost:5432  # PostgreSQL
curl -I http://localhost:6380  # Redis
curl -I http://localhost:27017  # MongoDB
```

#### Test External Access

```bash
# Test database client access
psql -h localhost -p 5432 -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT 1;"
redis-cli -h localhost -p 6380 -a ${DB_PASSWORD} ping
mongo --host localhost --port 27017 -u ${DB_USERNAME} -p ${DB_PASSWORD} ${DB_NAME} --eval "db.adminCommand('ping')"
```

---

## Rollback Procedures

### When to Rollback

Consider rollback if:
- Services fail to start consistently
- Data corruption is detected
- Performance degradation exceeds 20%
- Network connectivity issues persist
- SELinux conflicts cannot be resolved

### Rollback Process

#### Option 1: Switch Back to Docker

```bash
# Stop all Podman services
podman compose -f compose.yaml down

# Verify all containers are stopped
podman ps -a | grep db-compose || echo "No db-compose containers found"

# Start with Docker
docker compose -f compose.yaml up -d

# Verify Docker services are running
docker compose -f compose.yaml ps
```

#### Option 2: Restore from Backup

```bash
# Stop current services
podman compose -f compose.yaml down

# Restore Docker volumes
docker run --rm \
  -v $(docker volume ls -q | grep db-compose):/target \
  -v $(pwd)/migration-backups/$(date +%Y%m%d-%H%M%S):/backup \
  alpine tar xzf /backup/docker_volumes_backup.tar.gz -C /target

# Start Docker services
docker compose -f compose.yaml up -d

# Verify data integrity
docker compose -f compose.yaml exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT * FROM test_table;"
```

#### Option 3: Hybrid Approach (Docker and Podman Coexistence)

```bash
# Install Docker alongside Podman
# (Follow Docker installation instructions for your OS)

# Use Docker for critical services
docker compose -f databases/postgres/compose.yaml up -d
docker compose -f databases/redis/compose.yaml up -d

# Use Podman for less critical services
podman compose -f databases/mongo/compose.yaml up -d
podman compose -f databases/kafka/compose.yaml up -d

# Verify all services are running
docker compose -f databases/postgres/compose.yaml ps
podman compose -f databases/mongo/compose.yaml ps
```

### Data Rollback

#### Database Rollback

**PostgreSQL:**
```bash
# Restore from backup
docker compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} < postgres_backup.sql

# Verify restoration
docker compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT * FROM test_table;"
```

**Redis:**
```bash
# Restore from backup
docker compose cp ./redis_backup.rdb redis:/data/
docker compose restart redis

# Verify data
docker compose exec redis redis-cli get test-key
```

**MongoDB:**
```bash
# Restore from backup
docker compose cp ./mongo_backup mongo:/data/
docker compose exec mongorestore /data/mongo_backup

# Verify data
docker compose exec mongo mongosh --eval "db.test.find().count()"
```

---

## Common Migration Scenarios

### Scenario 1: Root User vs Non-Root User

#### Problem: Permission Denied Errors

```bash
# Check current user
whoami

# If running as root, consider switching to non-root
# Create dedicated user for containers
sudo useradd -m -s /bin/bash db-compose-user
sudo su - db-compose-user

# Set up environment
cp .env /home/db-compose-user/
chown db-compose-user:db-compose-user /home/db-compose-user/.env

# Run as non-root user
podman compose -f compose.yaml up -d
```

#### Solution: User Namespace Mapping

```bash
# Check user namespace configuration
podman info | grep userns

# Enable user namespace mapping if needed
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subuid
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subgid

# Reset Podman system
podman system reset
```

### Scenario 2: SELinux Issues

#### Problem: Permission Denied on Volume Mounts

```bash
# Check SELinux status
getenforce

# If in enforcing mode, add :Z suffix to volume mounts
```

#### Solution: Volume Mount Labels

Update compose files to include SELinux labels:

```yaml
services:
  postgres:
    volumes:
      - postgres_data:/var/lib/postgresql/data:Z
```

#### Alternative: Temporarily Disable SELinux

```bash
# Check current mode
getenforce

# Temporarily set to permissive (not recommended for production)
sudo setenforce 0

# Or disable completely (not recommended)
sudo setenforce 0
```

### Scenario 3: Network Connectivity Issues

#### Problem: Services Cannot Communicate

```bash
# Test network connectivity
podman compose exec postgres ping redis

# Check network configuration
podman network inspect ct_shared_network
```

#### Solution: Recreate Network

```bash
# Stop all services
podman compose -f compose.yaml down

# Remove network
podman network rm ct_shared_network

# Recreate network
podman network create ct_shared_network

# Restart services
podman compose -f compose.yaml up -d
```

### Scenario 4: Image Pull Failures

#### Problem: Cannot Pull Images

```bash
# Test image pull
podman pull docker.io/library/postgres:18-alpine

# If fails, check network and registry access
```

#### Solution: Alternative Registries

```bash
# Use official registry
podman pull docker.io/library/postgres:18-alpine

# Or use alternative mirrors
podman pull registry-1.docker.io/library/postgres:18-alpine
```

### Scenario 5: Volume Persistence Issues

#### Problem: Data Lost After Restart

```bash
# Check volume status
podman volume ls
podman volume inspect db-compose_postgres_data

# Test data persistence
podman compose down
podman compose up -d
podman compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT * FROM test_table;"
```

#### Solution: Volume Permissions

```bash
# Fix volume permissions
podman run --rm -v db-compose_postgres_data:/data alpine chown -R 999:999 /data

# Or recreate volume with correct permissions
podman volume rm db-compose_postgres_data
podman volume create db-compose_postgres_data
```

---

## Service-Specific Migration Notes

### PostgreSQL

#### Key Considerations
- Uses `pg_basebackup` for replication setup
- Requires proper user permissions for data directory
- WAL archiving configuration for replication

#### Migration Commands
```bash
# Verify primary node
podman compose exec postgres-primary psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT version();"

# Verify replication
podman compose exec postgres-replica-1 psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT pg_is_in_recovery();"

# Check replication lag
podman compose exec postgres-primary psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT pg_stat_replication;"
```

### Redis

#### Key Considerations
- Cluster initialization requires proper node order
- Password authentication must be consistent
- Memory limits and persistence settings

#### Migration Commands
```bash
# Check cluster status
podman compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info

# Verify cluster nodes
podman compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster nodes

# Test data distribution
podman compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} set key1 value1
podman compose exec redis-node-1 redis-cli -a ${DB_PASSWORD} get key1
```

### MongoDB

#### Key Considerations
- No built-in clustering in this configuration
- Authentication configuration
- Data directory permissions

#### Migration Commands
```bash
# Test connectivity
podman compose exec mongo mongosh --eval "db.adminCommand('ping')"

# Check database status
podman compose exec mongo mongosh --eval "db.adminCommand('buildInfo')"
```

### Kafka

#### Key Considerations
- ZooKeeper dependency
- Topic configuration
- Consumer group management

#### Migration Commands
```bash
# List topics
podman compose exec kafka kafka-topics --list --bootstrap-server localhost:9092

# Check broker status
podman compose exec kafka kafka-broker-api-versions --bootstrap-server localhost:9092
```

### RabbitMQ

#### Key Considerations
- Management plugin for monitoring
- User and permission management
- Queue and exchange configuration

#### Migration Commands
```bash
# Check server status
podman compose exec rabbitmq rabbitmqctl status

# List queues
podman compose exec rabbitmq rabbitmqctl list_queues
```

### MariaDB Galera

#### Key Considerations
- Galera replication configuration
- SST (State Snapshot Transfer) methods
- wsrep provider settings

#### Migration Commands
```bash
# Check cluster status
podman compose exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

# Check replication status
podman compose exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_local_state_comment';"
```

### ClickHouse

#### Key Considerations
- Memory usage configuration
- Table engine selection
- Query optimization

#### Migration Commands
```bash
# Test connectivity
podman compose exec clickhouse clickhouse-client --query "SELECT 1"

# Check system tables
podman compose exec clickhouse clickhouse-client --query "SELECT * FROM system.tables LIMIT 5"
```

### ScyllaDB

#### Key Considerations
- CQL compatibility with Cassandra
- Data partitioning
- Performance tuning

#### Migration Commands
```bash
# Test connectivity
podman compose exec scylla cqlsh -e "DESCRIBE KEYSPACE system"

# Check node status
podman compose exec scylla nodetool status
```

### NATS

#### Key Considerations
- JetStream for persistence
- Subject-based messaging
- Queue groups

#### Migration Commands
```bash
# Check server status
podman compose exec nats nats server --monitor

# List streams
podman compose exec nats nats stream ls
```

### Valkey

#### Key Considerations
- Redis-compatible key-value store
- Performance optimizations
- Memory efficiency

#### Migration Commands
```bash
# Test basic operations
podman compose exec valkey valkey-cli ping

# Check memory usage
podman compose exec valkey valkey-cli info memory
```

### DBGate

#### Key Considerations
- Database connection management
- Query execution interface
- Multiple database support

#### Migration Commands
```bash
# Check accessibility
curl -I http://localhost:30080

# Test database connections
podman compose exec dbgate curl -X POST http://localhost:30080/api/connections/test
```

### Traefik

#### Key Considerations
- Reverse proxy configuration
- SSL/TLS termination
- Load balancing

#### Migration Commands
```bash
# Check dashboard
curl -I http://localhost:80

# Check Traefik metrics
curl http://localhost:8080/api/metrics
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Podman Compose Not Found

```bash
# Check if podman-compose is installed
which podman-compose

# If not installed, install it
pip install podman-compose

# Or try alternative
alias podman-compose='podman compose'
```

#### Issue 2: Volume Permission Denied

```bash
# Check volume permissions
podman run --rm -v db-compose_postgres_data:/data alpine ls -la /data

# Fix permissions
podman run --rm -v db-compose_postgres_data:/data alpine chown -R 999:999 /data

# Or recreate volume
podman volume rm db-compose_postgres_data
podman volume create db-compose_postgres_data
```

#### Issue 3: Network Connectivity Problems

```bash
# Check network status
podman network inspect ct_shared_network

# Test DNS resolution
podman compose exec nslookup postgres

# Recreate network if needed
podman network rm ct_shared_network
podman network create ct_shared_network
```

#### Issue 4: Image Pull Failures

```bash
# Check registry access
podman pull docker.io/library/postgres:18-alpine

# Try alternative registry
podman pull registry-1.docker.io/library/postgres:18-alpine

# Clear image cache
podman image prune
```

#### Issue 5: Service Startup Failures

```bash
# Check logs
podman compose logs postgres

# Check container status
podman compose ps

# Inspect container
podman inspect postgres
```

### Debug Commands

#### Container Inspection

```bash
# Check container details
podman inspect postgres | grep -A 10 "State"

# Check container environment
podman inspect postgres | grep -A 5 "Env"

# Check container mounts
podman inspect postgres | grep -A 5 "Mounts"
```

#### Network Debugging

```bash
# Check network namespace
podman inspect postgres | grep -A 10 "NetworkSettings"

# Test connectivity between containers
podman compose exec postgres ping redis

# Check port mappings
podman port postgres
```

#### Volume Debugging

```bash
# Check volume location
podman volume inspect db-compose_postgres_data

# Check volume contents
podman run --rm -v db-compose_postgres_data:/data alpine ls -la /data

# Check volume permissions
podman run --rm -v db-compose_postgres_data:/data alpine stat /data
```

### Performance Tuning

#### Resource Limits

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

#### Memory Optimization

```bash
# Check memory usage
podman stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}"

# Adjust memory limits in compose files
services:
  redis:
    mem_limit: 512m
```

#### CPU Optimization

```bash
# Check CPU usage
podman stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}"

# Adjust CPU limits
services:
  kafka:
    deploy:
      resources:
        limits:
          cpus: '1.5'
```

---

## Performance Considerations

### Resource Usage Comparison

#### Memory Usage

```bash
# Monitor memory usage
podman stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}"

# Compare with Docker (if available)
echo "Docker memory usage:"
docker stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}"
```

#### CPU Usage

```bash
# Monitor CPU usage
podman stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}"

# Compare with Docker
echo "Docker CPU usage:"
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}"
```

#### Disk I/O

```bash
# Monitor disk I/O
podman stats --no-stream --format "table {{.Container}}\t{{.BlockIO}}"

# Check volume performance
podman run --rm -v db-compose_postgres_data:/data alpine dd if=/dev/zero of=/data/testfile bs=1M count=100
```

### Performance Optimization

#### Podman-Specific Optimizations

```bash
# Enable cgroups v2 for better resource management
sudo sysctl kernel.unprivileged_userns_clone=1

# Configure Podman for better performance
cat > ~/.config/containers/podman.toml << EOF
[engine]
events_logger="file"
EOF
```

#### Database-Specific Optimizations

**PostgreSQL:**
```yaml
services:
  postgres:
    environment:
      - POSTGRES_INITDB_ARGS="--auth-host=scram-sha-256"
      - SHM_SIZE=128m
    deploy:
      resources:
        limits:
          memory: 2G
```

**Redis:**
```yaml
services:
  redis:
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
```

### Monitoring and Observability

#### Podman System Monitoring

```bash
# Check Podman system info
podman info

# Check container statistics
podman stats --no-stream

# Check image usage
podman images
```

#### Service Monitoring

```bash
# Monitor service health
podman compose ps

# Monitor service logs
podman compose logs --tail=100 -f

# Monitor resource usage per service
podman stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

#### Database-Specific Monitoring

**PostgreSQL:**
```bash
# Check database status
podman compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT * FROM pg_stat_activity;"

# Check database performance
podman compose exec postgres psql -U ${DB_USERNAME} -d ${DB_NAME} -c "SELECT * FROM pg_stat_database;"
```

**Redis:**
```bash
# Check Redis info
podman compose exec redis redis-cli info

# Check Redis memory
podman compose exec redis redis-cli info memory
```

---

## Conclusion

This migration guide provides comprehensive instructions for transitioning from Docker to Podman while maintaining full functionality of the db-compose project. The migration process is designed to be safe, reversible, and minimally disruptive to your workflow.

### Key Takeaways

1. **Dual Runtime Support**: Both Docker and Podman can coexist on the same system
2. **Data Migration**: Volumes can be migrated with proper backup and restore procedures
3. **SELinux Considerations**: Proper volume labeling may be required on SELinux systems
4. **Image Registry**: Update from `mirror.gcr.io` to official registries for better compatibility
5. **Performance Benefits**: Podman typically offers better performance and security

### Next Steps

1. **Test in Development**: Practice the migration in a development environment first
2. **Schedule Maintenance Window**: Plan production migration during low-traffic periods
3. **Monitor Performance**: Compare performance metrics before and after migration
4. **Document Changes**: Update your team documentation with the new runtime setup

### Support Resources

- [Podman Documentation](https://podman.io/)
- [Podman Compose Documentation](https://github.com/containers/podman-compose)
- [db-compose Repository](https://github.com/your-repo/db-compose)
- [Docker to Podman Migration Guide](https://docs.docker.com/engine/migrate/podman/)

---

**Migration Complete!** Your db-compose project is now running on Podman with full functionality maintained.