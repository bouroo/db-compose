# db-compose - Technology Stack

## Container Orchestration

### Docker Compose / Podman Compose
- **Version**: Requires Compose v2+ (for `include` directive support)
- **Purpose**: Service orchestration and lifecycle management
- **Key Features Used**:
  - `include` directive for modular composition
  - Service dependencies with health conditions
  - Named volumes for data persistence
  - Custom networks for service discovery
  - Environment variable substitution

### Image Registry
- **Source**: `mirror.gcr.io` (Google Container Registry Mirror)
- **Rationale**: 
  - Reliable availability
  - Consistent access
  - Reduced rate limiting
  - Enterprise-grade infrastructure

## Database Technologies

### Relational Databases

#### PostgreSQL
- **Version**: 18-alpine
- **Image**: `mirror.gcr.io/postgresql:18-alpine`
- **Port**: 5432
- **Data Path**: `/var/lib/postgresql/data`
- **Features**:
  - ACID compliance
  - Advanced SQL features
  - JSON support
  - Full-text search
  - Extensibility

**Cluster Configuration**:
- **Components**: Primary + 2 Replicas + PgBouncer
- **Replication**: Streaming replication (WAL-based)
- **Connection Pooler**: PgBouncer (RapidFort)
- **Initialization**: `pg_basebackup` for replica setup
- **Authentication**: SCRAM-SHA-256
- **Parameters**:
  - `max_wal_senders=10`
  - `wal_level=replica`
  - `hot_standby=on`
  - `hot_standby_feedback=on`
  - `archive_mode=on`

#### MariaDB
- **Version**: LTS (Long Term Support)
- **Image**: `mirror.gcr.io/rapidfort/mariadb-official:lts`
- **Port**: 3306
- **Data Path**: `/var/lib/mysql`
- **Features**:
  - MySQL compatibility
  - Advanced replication
  - Performance optimizations
  - Storage engines (InnoDB, Aria)

**Galera Cluster Configuration**:
- **Components**: 3 Galera nodes + MaxScale
- **Replication**: Galera (virtually synchronous)
- **Load Balancer**: MaxScale 24.02
- **Cluster Protocol**: Galera wsrep
- **Ports**: 
  - 3306 (database)
  - 8989 (MaxScale admin)
- **Features**:
  - Multi-master replication
  - Automatic node provisioning
  - Read-write splitting
  - Query routing

### Document Databases

#### MongoDB
- **Version**: 8
- **Image**: `mirror.gcr.io/rapidfort/mongodb-official:8`
- **Port**: 27017
- **Data Path**: `/data/db`
- **Features**:
  - JSON-like documents (BSON)
  - Flexible schema
  - Rich query language
  - Aggregation framework
  - Indexing capabilities

### Key-Value Stores

#### Redis
- **Version**: 8-alpine
- **Image**: `mirror.gcr.io/rapidfort/redis-official:8-alpine`
- **Port**: 6380 (single), 6379-6384 (cluster)
- **Data Path**: `/data`
- **Features**:
  - In-memory data structure store
  - Persistence (RDB + AOF)
  - Pub/Sub messaging
  - Lua scripting
  - Transactions

**Cluster Configuration**:
- **Nodes**: 6 (3 masters + 3 replicas)
- **Hash Slots**: 16384 (distributed across masters)
- **Replication**: Asynchronous master-replica
- **Failover**: Automatic consensus-based
- **Initialization**: `redis-cli --cluster create`
- **Features**:
  - Automatic sharding
  - High availability
  - Data partitioning
  - Client-side redirection

#### Valkey
- **Version**: 8-alpine
- **Image**: `mirror.gcr.io/valkey/valkey:8-alpine`
- **Port**: 6379
- **Data Path**: `/data`
- **Features**:
  - Redis-compatible fork
  - Same protocol and features
  - Open-source governance
  - Performance optimizations

**Cluster Configuration**:
- Same architecture as Redis cluster
- 6 nodes (3 masters + 3 replicas)
- Redis cluster protocol compatible
- Automatic failover and recovery

### Wide-Column Stores

#### ScyllaDB
- **Version**: 2025.3
- **Image**: `mirror.gcr.io/scylladb/scylla:2025.3`
- **Port**: 9042
- **Data Path**: `/var/lib/scylla`
- **Features**:
  - Cassandra-compatible
  - Written in C++ (vs Java)
  - Low latency
  - High throughput
  - CQL (Cassandra Query Language)

### OLAP / Analytics

#### ClickHouse
- **Version**: LTS
- **Image**: `mirror.gcr.io/rapidfort/clickhouse-official:lts`
- **Port**: 9000
- **Data Path**: `/var/lib/clickhouse`
- **Features**:
  - Columnar storage
  - Real-time analytics
  - SQL support
  - Compression
  - Distributed queries

## Message Brokers

### Apache Kafka
- **Version**: 7.4.10 (Confluent Platform)
- **Image**: `mirror.gcr.io/confluentinc/cp-kafka:7.4.10`
- **Port**: 9092
- **Data Path**: `/var/lib/kafka`
- **Dependencies**: Zookeeper (automatically started)
- **Features**:
  - Distributed event streaming
  - High throughput
  - Fault tolerance
  - Log compaction
  - Consumer groups

### RabbitMQ
- **Version**: 4-management-alpine
- **Image**: `mirror.gcr.io/rapidfort/rabbitmq-official:4-management-alpine`
- **Port**: 5672
- **Data Path**: `/var/lib/rabbitmq`
- **Features**:
  - AMQP protocol
  - Message routing (exchanges, queues)
  - Management UI
  - Plugins ecosystem
  - Priority queues

### NATS
- **Version**: 2-alpine
- **Image**: `mirror.gcr.io/nats:2-alpine`
- **Port**: 4222
- **Data Path**: `/data`
- **Features**:
  - Cloud-native messaging
  - Pub/Sub patterns
  - Request/Reply
  - JetStream (persistence)
  - Lightweight and fast

## Support Services

### DBGate
- **Version**: alpine (latest)
- **Image**: `mirror.gcr.io/dbgate/dbgate:alpine`
- **Port**: 3000
- **Access**: http://dbgate.localhost:30080
- **Data Path**: `/root/.dbgate`
- **Features**:
  - Web-based database client
  - Multi-database support
  - SQL query execution
  - Schema browser
  - Data import/export
  - Connection management

**Supported Databases**:
- PostgreSQL
- MariaDB/MySQL
- MongoDB
- Redis
- ClickHouse
- ScyllaDB (Cassandra)

### Traefik
- **Version**: 3
- **Image**: `mirror.gcr.io/rapidfort/traefik:3`
- **Port**: 80
- **Data Path**: `/var/lib/traefik`
- **Features**:
  - Reverse proxy
  - Load balancer
  - Automatic service discovery
  - Let's Encrypt integration
  - Metrics and tracing

## Infrastructure Components

### Networking

#### Docker/Podman Network
- **Type**: Bridge network
- **Name**: `ct_shared_network`
- **Driver**: bridge (default)
- **DNS**: Automatic service name resolution
- **Isolation**: From host and external networks
- **Communication**: Service-to-service via service names

**Network Features**:
- Internal DNS (service discovery)
- Port mapping to host
- Network isolation
- Container-to-container communication

### Storage

#### Volume Management
- **Type**: Named volumes (Docker/Podman managed)
- **Persistence**: Data survives container lifecycle
- **Backup**: Manual (volume export/import)
- **Performance**: Optimized for database I/O

**Volume Locations**:
- **Podman**: `~/.local/share/containers/storage/volumes/`
- **Docker**: `/var/lib/docker/volumes/`

**Volume Naming**:
- Single instance: `<service>_data`
- Cluster nodes: `<service>_<role>_<number>_data`
- Examples:
  - `postgres_data`
  - `postgres_primary_data`
  - `postgres_replica_1_data`
  - `redis_node_0_data`

### Configuration Management

#### Environment Variables
- **Source**: `.env` file (root directory)
- **Template**: `example.env`
- **Scope**: All services
- **Substitution**: `${VARIABLE}` syntax in compose files

**Standard Variables**:
```bash
# Locale
TZ=Asia/Bangkok                    # Timezone
LANG=C.UTF-8                       # Language/Locale

# Database Credentials
DB_USERNAME=common_user            # Shared username
DB_PASSWORD=your_secure_password   # Shared password
DB_NAME=common_database            # Shared database name

# Replication Credentials
REPLICATION_USERNAME=replication_user
REPLICATION_PASSWORD=your_secure_password

# Admin UI Credentials
ADMIN_UI_USERNAME=admin_user
ADMIN_UI_PASSWORD=your_secure_password
```

## Health Check Utilities

### Service-Specific Health Checks

**PostgreSQL**:
```bash
pg_isready -U ${DB_USERNAME} -d ${DB_NAME}
```

**MariaDB**:
```bash
mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_local_state';"
```

**Redis/Valkey**:
```bash
redis-cli -a ${DB_PASSWORD} ping
# or
valkey-cli -h localhost -p 6379 ping
```

**MaxScale**:
```bash
maxadmin -u ${MAXSCALE_ADMIN_USER} -p ${MAXSCALE_ADMIN_PASSWORD} list servers
```

## Connection Poolers & Load Balancers

### PgBouncer
- **Image**: `mirror.gcr.io/rapidfort/pgbouncer-official:latest`
- **Purpose**: PostgreSQL connection pooling
- **Features**:
  - Connection pooling (reduces overhead)
  - Query routing
  - Load distribution
  - Auth type: md5
  - Multiple database support

**Configuration**:
```
DATABASES: "primary=host=postgres-primary...,replica1=...,replica2=..."
PGBOUNCER_LISTEN_PORT: 5432
PGBOUNCER_AUTH_TYPE: md5
```

### MaxScale
- **Image**: `mirror.gcr.io/maxscale/maxscale:24.02`
- **Purpose**: MariaDB/MySQL load balancing and query routing
- **Features**:
  - Read-write splitting
  - Load balancing
  - Query routing
  - Server monitoring
  - Admin API (port 8989)

**Routing**:
- Service: `readwritesplit`
- Servers: 3 Galera nodes
- Protocol: MariaDBBackend
- Health monitoring

## Initialization Utilities

### PostgreSQL Init Container
- **Purpose**: Create replication user and configure primary
- **Lifecycle**: Runs once, then exits
- **Commands**:
  ```sql
  CREATE USER ${REPLICATION_USERNAME} WITH REPLICATION...
  ALTER SYSTEM SET wal_level = replica;
  ALTER SYSTEM SET max_wal_senders = 10;
  ALTER SYSTEM SET hot_standby = on;
  SELECT pg_reload_conf();
  ```

### Redis Cluster Init
- **Purpose**: Create cluster topology and slot distribution
- **Lifecycle**: Runs once, then exits
- **Command**:
  ```bash
  redis-cli --cluster create \
    redis-node-0:6379 redis-node-1:6379 ... \
    --cluster-replicas 1 \
    -a ${DB_PASSWORD} \
    --cluster-yes
  ```

### Valkey Cluster Init
- **Purpose**: Initialize Valkey cluster
- **Lifecycle**: Runs once, then exits
- **Command**:
  ```bash
  valkey-cli --cluster create \
    valkey-0:6379 valkey-1:6379 ... \
    --cluster-replicas 1 \
    -a ${DB_PASSWORD} \
    --cluster-yes
  ```

## Image Optimizations

### Alpine Linux Base
- **Size**: Minimal (~5MB base)
- **Benefits**:
  - Faster image pulls
  - Reduced disk usage
  - Smaller attack surface
  - Security-focused

**Images Using Alpine**:
- PostgreSQL 18-alpine
- Redis 8-alpine
- Valkey 8-alpine
- NATS 2-alpine
- RabbitMQ 4-management-alpine
- DBGate alpine

### RapidFort Hardened Images
- **Purpose**: Security-hardened container images
- **Features**:
  - Reduced attack surface
  - Vulnerability scanning
  - Runtime optimization
  - Performance tuning

**RapidFort Images Used**:
- MariaDB LTS
- Redis 8-alpine
- MongoDB 8
- RabbitMQ 4-management-alpine
- ClickHouse LTS
- Traefik 3
- PgBouncer latest

## Port Allocations

### Standard Ports
| Service | Port | Protocol | Notes |
|---------|------|----------|-------|
| PostgreSQL | 5432 | TCP | Standard PostgreSQL port |
| PostgreSQL Cluster | 5432 | TCP | Via PgBouncer |
| MariaDB | 3306 | TCP | Standard MySQL/MariaDB port |
| Galera Cluster | 3306 | TCP | Via MaxScale |
| MaxScale Admin | 8989 | HTTP | Management interface |
| MongoDB | 27017 | TCP | Standard MongoDB port |
| Redis | 6380 | TCP | Non-standard (avoid conflict) |
| Redis Cluster | 6379-6384 | TCP | 6 nodes |
| Valkey | 6379 | TCP | Standard Redis port |
| ClickHouse | 9000 | TCP | Native protocol |
| ScyllaDB | 9042 | TCP | CQL native transport |
| Kafka | 9092 | TCP | Kafka protocol |
| RabbitMQ | 5672 | TCP | AMQP protocol |
| NATS | 4222 | TCP | NATS protocol |
| DBGate | 3000 | HTTP | Web interface |
| Traefik | 80 | HTTP | Reverse proxy |

### Port Conflicts to Note
- **Redis single instance**: Port 6380 (non-standard to avoid conflict)
- **Redis cluster**: Ports 6379-6384 (standard range)
- **Valkey**: Port 6379 (standard Redis port)

⚠️ **Warning**: Cannot run Redis single instance and Valkey simultaneously without port changes.

## Technology Dependencies

### Kafka → Zookeeper
- Kafka requires Zookeeper for coordination
- Automatically started via `depends_on`
- Zookeeper not directly exposed in compose files (handled by Kafka image)

### Cluster Dependencies

**PostgreSQL Cluster**:
```
postgres-init → postgres-primary (healthy)
postgres-replica-1 → postgres-primary (healthy)
postgres-replica-2 → postgres-primary (healthy)
pgbouncer → all postgres nodes (healthy)
```

**MariaDB Galera**:
```
galera-node-1 → galera-node-0 (healthy)
galera-node-2 → galera-node-0 (healthy) + galera-node-1 (healthy)
maxscale → all galera nodes (healthy)
```

**Redis Cluster**:
```
redis-cluster-init → all redis-node-* (healthy)
```

**Valkey Cluster**:
```
valkey-cluster-init → all valkey-* (healthy)
```

## Version Strategy

### LTS (Long Term Support)
- **MariaDB**: `lts` tag (stable, long-term support)
- **ClickHouse**: `lts` tag (production-ready)

### Specific Versions
- **PostgreSQL**: `18-alpine` (latest major version)
- **Redis**: `8-alpine` (latest major version)
- **Valkey**: `8-alpine` (Redis-compatible version)
- **MongoDB**: `8` (latest major version)
- **ScyllaDB**: `2025.3` (year.month versioning)
- **Kafka**: `7.4.10` (Confluent Platform version)
- **RabbitMQ**: `4-management-alpine` (major version)
- **NATS**: `2-alpine` (major version)
- **Traefik**: `3` (major version)
- **MaxScale**: `24.02` (year.month versioning)

### Latest Tags
- **PgBouncer**: `latest` (RapidFort hardened)
- **DBGate**: `alpine` (rolling release)

## Compatibility Matrix

### Container Runtimes
| Runtime | Version | Supported | Notes |
|---------|---------|-----------|-------|
| Podman | 4.0+ | ✅ | Recommended by project |
| Docker | 20.10+ | ✅ | Fully compatible |
| Podman Compose | 1.0+ | ✅ | Compose v2 features required |
| Docker Compose | 2.0+ | ✅ | `include` directive support |

### Operating Systems
| OS | Tested | Notes |
|----|--------|-------|
| Linux | ✅ | Primary target |
| macOS | ✅ | Via Docker Desktop/Podman |
| Windows | ✅ | Via Docker Desktop/WSL2 |

### Architectures
| Architecture | Supported | Images |
|--------------|-----------|--------|
| amd64 (x86_64) | ✅ | All images |
| arm64 (Apple Silicon) | ⚠️ | Most images, verify individually |

## Development Tools

### Required
- Podman Desktop or Docker Desktop
- Text editor for `.env` configuration
- Web browser for DBGate access

### Optional
- Database clients (psql, mysql, mongosh, redis-cli)
- Monitoring tools (docker stats, podman stats)
- Log viewers (docker logs, podman logs)

### Recommended IDE Extensions
- Docker/Podman extension
- YAML language support
- Compose file validation

## Security Considerations

### Image Security
- ✅ Images from trusted registries
- ✅ RapidFort hardened images where available
- ✅ Alpine-based for reduced attack surface
- ⚠️ Regular updates needed for security patches

### Network Security
- ✅ Isolated network for service communication
- ✅ Port exposure limited to necessary services
- ⚠️ All services on single network (no segmentation)
- ❌ No TLS/SSL by default (add if needed)

### Credential Security
- ✅ `.env` excluded from version control
- ✅ Template provided for new setups
- ⚠️ Suitable for development only
- ❌ No secret rotation mechanism
- ❌ Credentials in plain text

### Production Recommendations
1. Use Docker Secrets or external secret management
2. Enable TLS for all connections
3. Implement network segmentation
4. Regular security updates
5. Audit logging enabled
6. Access control and authentication hardening

## Performance Tuning

### Resource Limits (Not Configured)
Currently no resource limits set. Add if needed:
```yaml
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
```

### Database-Specific Tuning

**PostgreSQL**:
- `shared_buffers`: 25% of RAM
- `work_mem`: Based on connections
- `max_connections`: Application dependent

**MariaDB**:
- `innodb_buffer_pool_size`: 70% of RAM
- `max_connections`: Application dependent

**Redis/Valkey**:
- `maxmemory`: Set limit to prevent OOM
- `maxmemory-policy`: Choose eviction policy

**MongoDB**:
- WiredTiger cache: 50% of RAM by default
- Connection pooling in application

## Monitoring & Observability

### Built-in
- Health checks (all services)
- Docker/Podman logs
- Container stats (CPU, memory, I/O)

### External (Not Included)
- Prometheus exporters
- Grafana dashboards
- APM tools
- Log aggregation

### Future Additions
- Database-specific exporters
- Centralized metrics collection
- Alert manager integration
- Distributed tracing

## Backup & Recovery

### Current State
- ❌ No automated backups
- ❌ No backup utilities included
- ✅ Data persists in volumes
- ⚠️ Manual backup required

### Recommended Approach
1. Volume snapshots
2. Database-specific backup tools
3. Regular export procedures
4. Off-site backup storage

### Future Enhancements
- Backup automation scripts
- Point-in-time recovery
- Backup scheduling
- Restore testing automation