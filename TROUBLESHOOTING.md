# Troubleshooting Guide for db-compose

This comprehensive troubleshooting guide covers common issues and solutions for both Podman and Docker runtimes when using the db-compose project.

## Table of Contents

- [Common Issues](#common-issues)
- [Runtime-Specific Issues](#runtime-specific-issues)
- [Volume and Network Debugging](#volume-and-network-debugging)
- [Permission Issues](#permission-issues)
- [Port Conflicts](#port-conflicts)
- [Service Startup Failures](#service-startup-failures)
- [Cluster Configuration Issues](#cluster-configuration-issues)
- [Performance Tuning](#performance-tuning)
- [Log Collection and Analysis](#log-collection-and-analysis)
- [Service-Specific Troubleshooting](#service-specific-troubleshooting)
- [Advanced Diagnostics](#advanced-diagnostics)

## Common Issues

### Installation and Setup Issues

#### Docker/Podman Not Found

**Error:**
```bash
$ docker: command not found
# or
$ podman: command not found
```

**Solutions:**

**For Docker:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
```

**For Podman:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y podman

# Install podman-compose
pip install podman-compose

# Verify installation
podman --version
podman-compose --version
```

#### Compose Plugin Not Available

**Error:**
```bash
$ docker compose: command not found
```

**Solution:**
```bash
# For Docker
sudo apt install docker-compose-plugin

# For Podman
pip install podman-compose
```

### Environment Configuration Issues

#### Missing .env File

**Error:**
```bash
ERROR: Couldn't find env file: .env
```

**Solution:**
```bash
# Create .env file from example
cp example.env .env

# Edit with your credentials
nano .env
```

#### Invalid Environment Variables

**Error:**
```bash
ERROR: The service "postgres" failed to build with:
invalid keys: "INVALID_VAR"
```

**Solution:**
```bash
# Validate .env file
cat .env

# Check for syntax errors
grep -n '=' .env

# Validate required variables
grep -E '^(DB_USERNAME|DB_PASSWORD|DB_NAME)' .env
```

## Runtime-Specific Issues

### Docker-Specific Issues

#### Docker Daemon Not Running

**Error:**
```bash
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

**Solution:**
```bash
# Check Docker status
sudo systemctl status docker

# Start Docker
sudo systemctl start docker

# Enable auto-start on boot
sudo systemctl enable docker

# For systemd users
sudo usermod -aG docker $USER
newgrp docker
```

#### Docker Resource Limits

**Error:**
```bash
ERROR: for postgres: Cannot start service postgres: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "docker-init": executable file not found in $PATH: unknown
```

**Solution:**
```bash
# Check Docker resource limits
docker info | grep -E "(CPU|Memory)"

# Adjust Docker Desktop settings if using GUI
# Or adjust daemon configuration
sudo nano /etc/docker/daemon.json

{
  "default-shm-size": "2g"
}

# Restart Docker
sudo systemctl restart docker
```

#### Docker Network Issues

**Error:**
```bash
ERROR: for postgres: Network ct_shared_network is declared as external, but could not be found
```

**Solution:**
```bash
# Create network manually
docker network create ct_shared_network

# Or check compose file syntax
docker compose config
```

### Podman-Specific Issues

#### Podman Socket Not Available

**Error:**
```bash
Error: cannot access Podman socket: no socket found at /run/user/1000/podman/podman.sock
```

**Solution:**
```bash
# Check Podman socket status
systemctl --user status podman

# Start Podman socket
systemctl --user start podman

# Enable auto-start
systemctl --user enable podman

# For rootful mode
sudo systemctl enable --now podman.socket
```

#### SELinux Issues

**Error:**
```bash
Error: container creation failed: SELinux is not supported
```

**Solution:**
```bash
# Check SELinux status
sestatus

# Set to permissive mode (development)
sudo setenforce 0

# Or add volume mount labels
# Edit compose.yaml to include :Z suffix
volumes:
  - postgres_data:/var/lib/postgresql/data:Z
```

#### User Namespace Issues

**Error:**
```bash
Error: cannot create container: user namespace mapping failed
```

**Solution:**
```bash
# Check user namespace mapping
podman info | grep userns

# Add user namespace mapping
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subuid
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subgid

# Reset Podman system
podman system reset
```

## Volume and Network Debugging

### Volume Issues

#### Volume Permission Denied

**Error:**
```bash
ERROR: for postgres: failed to mount volume: mount /var/lib/docker/volumes/db-compose_postgres_data/_data: permission denied
```

**Solutions:**

**For Docker:**
```bash
# Fix volume permissions
sudo chown -R 999:999 /var/lib/docker/volumes/db-compose_postgres_data/_data

# Or run with proper user
docker run --user 999:999 postgres ls -la /var/lib/postgresql/data
```

**For Podman:**
```bash
# Fix volume permissions
podman run --user 999:999 postgres ls -la /var/lib/postgresql/data

# Or adjust user namespace mapping
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subuid
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subgid
```

#### Volume Not Found

**Error:**
```bash
ERROR: for postgres: Cannot start service postgres: error while creating volume mount path: no such file or directory
```

**Solution:**
```bash
# Create volume manually
docker volume create db-compose_postgres_data

# Or check volume paths
docker volume ls
docker volume inspect db-compose_postgres_data
```

### Network Issues

#### Network Connectivity Problems

**Error:**
```bash
ERROR: Service cannot connect to other services
```

**Solution:**
```bash
# Check network configuration
docker network inspect ct_shared_network

# Test connectivity
docker compose exec ping postgres
docker compose exec ping redis

# Check DNS resolution
docker compose exec nslookup postgres
```

#### Port Conflicts

**Error:**
```bash
ERROR: for postgres: Bind for 0.0.0.0:5432 failed: port is already allocated
```

**Solution:**
```bash
# Check port usage
netstat -tlnp | grep 5432

# Change port mapping in compose.yaml
ports:
  - "5433:5432"  # Use different host port

# Or stop conflicting service
sudo systemctl stop postgresql
```

## Permission Issues

### User Permission Problems

#### Docker Group Membership

**Error:**
```bash
Got permission denied while trying to connect to the Docker daemon socket
```

**Solution:**
```bash
# Check Docker group membership
groups

# Add user to docker group
sudo usermod -aG docker $USER

# Activate group changes
newgrp docker

# Verify
docker ps
```

#### Rootless Container Issues

**Error:**
```bash
Error: cannot create container: permission denied
```

**Solution:**

**For Docker:**
```bash
# Use sudo
sudo docker compose up -d

# Or set up docker group
sudo usermod -aG docker $USER
newgrp docker
```

**For Podman:**
```bash
# Enable user namespace mapping
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subuid
echo "$(id -u):$(id -g):1" | sudo tee -a /etc/subgid

# Reset Podman
podman system reset
```

### File Permission Issues

#### Mount Permission Problems

**Error:**
```bash
ERROR: for postgres: failed to mount volume: mount /path/to/volume: permission denied
```

**Solution:**
```bash
# Fix directory permissions
sudo chown -R $(whoami):$(id -gn) /path/to/volume

# Or use proper user in compose
services:
  postgres:
    user: "999:999"
```

## Port Conflicts

### Common Port Conflicts

#### PostgreSQL Port Conflict

**Error:**
```bash
ERROR: for postgres: Bind for 0.0.0.0:5432 failed: port is already allocated
```

**Solution:**
```bash
# Check what's using the port
sudo lsof -i :5432
sudo netstat -tlnp | grep 5432

# Stop conflicting service
sudo systemctl stop postgresql

# Or change port mapping
# Edit databases/postgres/compose.yaml
ports:
  - "5433:5432"  # Use different host port
```

#### Redis Port Conflict

**Error:**
```bash
ERROR: for redis: Bind for 0.0.0.0:6380 failed: port is already allocated
```

**Solution:**
```bash
# Check Redis service
sudo systemctl status redis

# Stop Redis service
sudo systemctl stop redis

# Or change port mapping
# Edit databases/redis/compose.yaml
ports:
  - "6381:6379"  # Use different host port
```

### Port Mapping Verification

#### Check Port Usage

```bash
# Check all used ports
sudo ss -tulpn | grep LISTEN

# Check specific port
sudo lsof -i :5432

# Check container port mappings
docker compose ps --format "table {{.Service}}\t{{.Ports}}"
```

#### Dynamic Port Assignment

```yaml
# Use dynamic port assignment
services:
  redis:
    ports:
      - "6380"  # Random host port
```

## Service Startup Failures

### General Service Issues

#### Service Failing to Start

**Error:**
```bash
ERROR: for postgres: failed to create container: invalid mode for service
```

**Solution:**
```bash
# Check service configuration
docker compose config

# Validate compose file syntax
docker compose config

# Check resource availability
free -h
df -h
```

#### Health Check Failures

**Error:**
```bash
ERROR: Service "postgres" is unhealthy
```

**Solution:**
```bash
# Check service status
docker compose ps

# View logs
docker compose logs postgres

# Manual health check
docker compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}
```

### Database Service Issues

#### PostgreSQL Startup Issues

**Error:**
```bash
ERROR: database files are incompatible with server
```

**Solution:**
```bash
# Remove volume and recreate
docker compose down --volumes
docker volume rm db-compose_postgres_data
docker compose up -d postgres

# Or check PostgreSQL version
docker compose exec postgres psql -c "SELECT version();"
```

#### Redis Startup Issues

**Error:**
```bash
ERROR: Failed to create cluster
```

**Solution:**
```bash
# Check Redis configuration
docker compose exec redis redis-cli config get *

# Reset cluster
docker compose down --volumes
docker compose up -d redis
```

### Cluster Service Issues

#### PostgreSQL Cluster Issues

**Error:**
```bash
ERROR: postgres-primary service is unhealthy
```

**Solution:**
```bash
# Check cluster status
docker compose exec postgres-primary pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Check replication
docker compose exec postgres-primary psql -c "SELECT * FROM pg_stat_replication;"

# Reset cluster
docker compose down --volumes
docker compose up -d postgres-cluster
```

#### Redis Cluster Issues

**Error:**
```bash
ERROR: Redis cluster is not healthy
```

**Solution:**
```bash
# Check cluster status
docker compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info

# Check cluster nodes
docker compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster nodes

# Recreate cluster
docker compose down --volumes
docker compose up -d redis-cluster
```

## Cluster Configuration Issues

### PostgreSQL Cluster Issues

#### Replication Problems

**Error:**
```bash
ERROR: could not connect to primary server
```

**Solution:**
```bash
# Check primary server
docker compose exec postgres-primary pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Check replica status
docker compose exec postgres-replica-1 pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Check replication status
docker compose exec postgres-primary psql -c "SELECT * FROM pg_stat_replication;"

# Recreate cluster
docker compose down --volumes
docker compose up -d postgres-cluster
```

#### Connection Pooler Issues

**Error:**
```bash
ERROR: pgbouncer connection failed
```

**Solution:**
```bash
# Check pgbouncer status
docker compose exec pgbouncer psql -U ${ADMIN_UI_USERNAME} -c "SHOW POOLS;"

# Check pgbouncer logs
docker compose logs pgbouncer

# Reset connection pooler
docker compose restart pgbouncer
```

### Redis Cluster Issues

#### Cluster Formation Problems

**Error:**
```bash
ERROR: Cluster state is not ok
```

**Solution:**
```bash
# Check cluster status
docker compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info

# Check cluster nodes
docker compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster nodes

# Recreate cluster
docker compose down --volumes
docker compose up -d redis-cluster
```

#### Node Communication Issues

**Error:**
```bash
ERROR: Node communication failed
```

**Solution:**
```bash
# Check node connectivity
docker compose exec redis-node-0 ping redis-node-1

# Check cluster configuration
docker compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info

# Reset cluster
docker compose down --volumes
docker compose up -d redis-cluster
```

### MariaDB Galera Issues

#### Cluster Synchronization Problems

**Error:**
```bash
ERROR: wsrep status not synchronized
```

**Solution:**
```bash
# Check Galera status
docker compose exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

# Check node status
docker compose exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_local_state';"

# Reset cluster
docker compose down --volumes
docker compose up -d mariadb-galera
```

#### MaxScale Issues

**Error:**
```bash
ERROR: MaxScale connection failed
```

**Solution:**
```bash
# Check MaxScale status
docker compose exec maxscale maxadmin -u ${ADMIN_UI_USERNAME} -p ${ADMIN_UI_PASSWORD} list servers

# Check MaxScale logs
docker compose logs maxscale

# Reset MaxScale
docker compose restart maxscale
```

## Performance Tuning

### Resource Optimization

#### Memory Management

**Issue:**
```bash
WARNING: memory usage is high
```

**Solution:**
```yaml
# Add resource limits
services:
  postgres:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
```

#### CPU Optimization

**Issue:**
```bash
WARNING: CPU usage is high
```

**Solution:**
```yaml
# Add CPU limits
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '1.0'
        reservations:
          cpus: '0.5'
```

### Storage Optimization

#### Volume Performance

**Issue:**
```bash
WARNING: I/O performance is poor
```

**Solution:**
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

#### Database Configuration

**PostgreSQL:**
```yaml
services:
  postgres:
    environment:
      - POSTGRES_SHARED_BUFFERS=256MB
      - POSTGRES_EFFECTIVE_CACHE_SIZE=1GB
      - POSTGRES_MAINTENANCE_WORK_MEM=64MB
      - POSTGRES_CHECKPOINT_TARGET=32MB
```

**Redis:**
```yaml
services:
  redis:
    command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
```

### Network Optimization

#### Network Performance

**Issue:**
```bash
WARNING: network latency is high
```

**Solution:**
```yaml
# Configure network settings
services:
  postgres:
    networks:
      ct_shared_network:
        aliases:
          - db-primary
```

#### DNS Optimization

**Issue:**
```bash
WARNING: DNS resolution is slow
```

**Solution:**
```yaml
# Add DNS servers
services:
  postgres:
    dns:
      - 8.8.8.8
      - 8.8.4.4
```

## Log Collection and Analysis

### Log Collection Methods

#### Basic Log Collection

```bash
# Collect all logs
docker compose logs > logs.txt

# Collect specific service logs
docker compose logs postgres > postgres_logs.txt

# Follow logs in real-time
docker compose logs -f postgres
```

#### Advanced Log Collection

```bash
# Collect logs with timestamps
docker compose logs --timestamps > timestamped_logs.txt

# Filter logs by level
docker compose logs --filter "level=error" > error_logs.txt

# Collect logs for debugging
docker compose logs --tail=1000 > debug_logs.txt
```

### Log Analysis

#### Log Pattern Analysis

```bash
# Search for error patterns
grep -i "error" debug_logs.txt

# Count error occurrences
grep -i "error" debug_logs.txt | wc -l

# Find slow queries
grep -i "slow" debug_logs.txt
```

#### Log Aggregation

```bash
# Use jq for JSON log parsing
docker compose logs --format json | jq '. | select(.log | contains("error"))'

# Extract specific fields
docker compose logs --format json | jq '.log'
```

### Service-Specific Logs

#### PostgreSQL Logs

```bash
# View PostgreSQL logs
docker compose logs postgres

# Check PostgreSQL error logs
docker compose exec postgres cat /var/log/postgresql/postgresql.log

# Monitor PostgreSQL in real-time
docker compose logs -f postgres
```

#### Redis Logs

```bash
# View Redis logs
docker compose logs redis

# Check Redis server logs
docker compose exec redis cat /var/log/redis/redis-server.log

# Monitor Redis in real-time
docker compose logs -f redis
```

## Service-Specific Troubleshooting

### PostgreSQL Troubleshooting

#### Connection Issues

**Error:**
```bash
psql: could not connect to server: Connection refused
```

**Solution:**
```bash
# Check PostgreSQL status
docker compose exec postgres pg_isready -U ${DB_USERNAME} -d ${DB_NAME}

# Check PostgreSQL logs
docker compose logs postgres

# Check port mapping
docker compose ps postgres
```

#### Authentication Issues

**Error:**
```bash
FATAL: password authentication failed for user "common_user"
```

**Solution:**
```bash
# Check environment variables
cat .env

# Reset password
docker compose down --volumes
docker compose up -d postgres
```

#### Performance Issues

**Error:**
```bash
WARNING: database is overloaded
```

**Solution:**
```bash
# Check database performance
docker compose exec postgres psql -c "SELECT * FROM pg_stat_activity;"

# Check memory usage
docker compose exec postgres psql -c "SELECT * FROM pg_stat_bgwriter;"

# Optimize configuration
# Edit compose.yaml to add resource limits
```

### Redis Troubleshooting

#### Connection Issues

**Error:**
```bash
Could not connect to Redis at localhost:6380
```

**Solution:**
```bash
# Check Redis status
docker compose exec redis redis-cli ping

# Check Redis logs
docker compose logs redis

# Check port mapping
docker compose ps redis
```

#### Memory Issues

**Error:**
```bash
OOM command not allowed when used memory > 'maxmemory'
```

**Solution:**
```bash
# Check memory usage
docker compose exec redis redis-cli info memory

# Check configuration
docker compose exec redis redis-cli config get maxmemory

# Update memory limits
# Edit compose.yaml to add memory limits
```

#### Cluster Issues

**Error:**
```bash
ERR This instance has cluster support disabled
```

**Solution:**
```bash
# Check cluster status
docker compose exec redis-node-0 redis-cli -a ${DB_PASSWORD} cluster info

# Recreate cluster
docker compose down --volumes
docker compose up -d redis-cluster
```

### MongoDB Troubleshooting

#### Connection Issues

**Error:**
```bash
MongoServerError: not authorized on admin to execute command
```

**Solution:**
```bash
# Check MongoDB status
docker compose exec mongo mongosh --eval "db.runCommand('ping')"

# Check MongoDB logs
docker compose logs mongo

# Check authentication
cat .env
```

#### Performance Issues

**Error:**
```bash
MongoServerError: connection timeout
```

**Solution:**
```bash
# Check MongoDB performance
docker compose exec mongo mongosh --eval "db.stats()"

# Check memory usage
docker compose exec mongo mongosh --eval "db.serverStatus().mem"

# Update resource limits
```

### Kafka Troubleshooting

#### Connection Issues

**Error:**
```bash
org.apache.kafka.common.errors.NetworkException: The broker is not available
```

**Solution:**
```bash
# Check Kafka status
docker compose exec kafka kafka-topics --list --bootstrap-server localhost:9092

# Check Kafka logs
docker compose logs kafka

# Check ZooKeeper status
docker compose exec zookeeper echo "ruok" | nc localhost 2181
```

#### Performance Issues

**Error:**
```bash
WARN Error while fetching metadata with correlation id 1
```

**Solution:**
```bash
# Check Kafka performance
docker compose exec kafka kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group test-group

# Check broker logs
docker compose logs kafka
```

## Advanced Diagnostics

### Container Inspection

#### Container Details

```bash
# Inspect container configuration
docker compose exec postgres env

# Check container health
docker compose exec postgres cat /proc/1/status

# Check container resources
docker compose exec postgres cat /sys/fs/cgroup/cpu/cpu.shares
```

#### Network Inspection

```bash
# Check container network configuration
docker compose exec postgres ip a

# Check container DNS
docker compose exec postgres cat /etc/resolv.conf

# Check container routes
docker compose exec postgres ip route
```

### Performance Analysis

#### Resource Monitoring

```bash
# Monitor resource usage
docker stats --no-stream

# Check container performance
docker compose exec postgres cat /proc/meminfo

# Check I/O performance
docker compose exec postgres iostat -x 1
```

#### Database Performance

**PostgreSQL:**
```bash
# Check database performance
docker compose exec postgres psql -c "SELECT * FROM pg_stat_activity;"

# Check cache hit ratio
docker compose exec postgres psql -c "SELECT sum(heap_blks_hit) / nullif(sum(heap_blks_hit) + sum(heap_blks_read), 0) AS ratio FROM pg_statio_user_tables;"

# Check query performance
docker compose exec postgres psql -c "SELECT query, calls, total_time, mean_time FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;"
```

**Redis:**
```bash
# Check Redis performance
docker compose exec redis redis-cli info

# Check slow queries
docker compose exec redis redis-cli slowlog get 10

# Check memory usage
docker compose exec redis redis-cli info memory
```

### System Diagnostics

#### System Resources

```bash
# Check system memory
free -h

# Check CPU usage
top -bn1 | grep "Cpu(s)"

# Check disk usage
df -h
```

#### Kernel Parameters

```bash
# Check kernel parameters
sysctl -a | grep -E "(shm|file|max)"

# Adjust kernel parameters
sudo sysctl -w kernel.shmmax=4294967295
sudo sysctl -w fs.file-max=100000
```

## Error Reference

### Common Error Codes

#### Docker Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `Cannot connect to Docker daemon` | Docker daemon not running | Start Docker service |
| `Port is already allocated` | Port conflict | Change port mapping or stop conflicting service |
| `Permission denied` | Permission issues | Add user to docker group |
| `Volume not found` | Volume missing | Create volume manually |

#### Podman Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `Cannot access Podman socket` | Podman socket not running | Start Podman socket |
| `SELinux is not supported` | SELinux issues | Set to permissive mode or add labels |
| `User namespace mapping failed` | User namespace issues | Configure user namespace mapping |
| `Container creation failed` | Resource limits | Adjust resource limits |

#### Database Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `Connection refused` | Service not running | Check service status |
| `Authentication failed` | Wrong credentials | Check environment variables |
| `Database overloaded` | Performance issues | Optimize configuration |
| `Cluster not healthy` | Cluster issues | Recreate cluster |

## Conclusion

This troubleshooting guide provides comprehensive solutions for common issues encountered when using db-compose with both Docker and Podman runtimes. By following these steps, you should be able to diagnose and resolve most problems that arise during setup, operation, or maintenance of your database services.

For additional help:
- Check the [Docker documentation](https://docs.docker.com/)
- Review the [Podman documentation](https://podman.io/)
- Create an issue in the db-compose repository for specific problems

Remember to:
1. Always check logs first (`docker compose logs` or `podman compose logs`)
2. Verify environment variables and configuration
3. Check resource availability and limits
4. Test network connectivity between services
5. Consider using the issue templates when reporting problems

Happy troubleshooting!