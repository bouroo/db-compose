# db-compose - Current Context

**Last Updated**: 2025-11-16  
**Memory Bank Status**: [Memory Bank: Active]

## Project Understanding Summary

db-compose is a mature, stable containerized database collection monorepo providing 17 different database and messaging services through modular Docker/Podman Compose configurations. The project emphasizes production-ready patterns with both standalone and clustered deployment options.

## Current State

### Development Phase
**Phase**: Maintenance and Enhancement (Stable Production-Ready State)

The project is in a stable state with:
- All core database services implemented and tested
- Comprehensive README documentation
- Production-ready cluster configurations
- MIT licensed open-source project

### Recent Changes
- No recent changes detected (analyzing stable codebase)
- Project appears complete and functional
- All configurations follow consistent patterns

### Active Work Focus
Currently initializing memory bank - no active development detected.

## Project Maturity Assessment

### Completeness: ~95%
**Implemented**:
- ✅ 11 database services (PostgreSQL, MariaDB, MongoDB, Redis, Valkey, ClickHouse, ScyllaDB)
- ✅ 3 message brokers (Kafka, RabbitMQ, NATS)
- ✅ 4 cluster configurations (PostgreSQL, MariaDB Galera, Redis, Valkey)
- ✅ Support services (DBGate, Traefik)
- ✅ Centralized configuration management
- ✅ Shared networking infrastructure
- ✅ Comprehensive documentation

**Potentially Missing**:
- ❓ Monitoring/observability stack
- ❓ Backup/restore utilities
- ❓ Additional databases (Elasticsearch, Neo4j, TimescaleDB)

### Code Quality: High
- Consistent file organization across all services
- Standard patterns for health checks
- Proper dependency management
- Clear naming conventions
- Production-ready restart policies

## Directory Structure

```
db-compose/
├── .agents/rules/memory-bank/     # Project memory (NEW)
│   ├── brief.md
│   ├── product.md
│   ├── context.md (this file)
│   ├── architecture.md
│   └── tech.md
├── databases/                      # All database services
│   ├── clickhouse/
│   ├── dbgate/
│   ├── kafka/
│   ├── mariadb/
│   ├── mariadb-galera/           # 3-node cluster with MaxScale
│   ├── mongo/
│   ├── nats/
│   ├── postgres/
│   ├── postgres-cluster/         # Primary + 2 replicas with PgBouncer
│   ├── rabbitmq/
│   ├── redis/
│   ├── redis-cluster/            # 6-node cluster
│   ├── scylladb/
│   ├── traefik/
│   ├── valkey/
│   └── valkey-cluster/           # 6-node cluster
├── compose.yaml                   # Root compose with includes
├── example.env                    # Configuration template
├── .env                          # User configuration (gitignored)
├── .gitignore
├── LICENSE
└── README.md
```

## Service Inventory

### Single-Instance Services (11)
1. **clickhouse** - Port 9000 - Analytics database
2. **dbgate** - Port 3000 - Database management UI
3. **kafka** - Port 9092 - Event streaming (+ Zookeeper dependency)
4. **mariadb** - Port 3306 - SQL database
5. **mongo** - Port 27017 - Document database
6. **nats** - Port 4222 - Messaging system
7. **postgres** - Port 5432 - SQL database
8. **rabbitmq** - Port 5672 - Message broker
9. **redis** - Port 6380 - Key-value store (note: non-standard port)
10. **scylladb** - Port 9042 - Wide-column store
11. **traefik** - Port 80 - Reverse proxy
12. **valkey** - Port 6379 - Redis-compatible key-value store

### Cluster Configurations (4)

1. **postgres-cluster**
   - postgres-primary (internal)
   - postgres-replica-1 (internal)
   - postgres-replica-2 (internal)
   - pgbouncer (port 5432) - Connection pooler
   - postgres-init (one-time setup)

2. **mariadb-galera**
   - galera-node-0, galera-node-1, galera-node-2 (internal)
   - maxscale (ports 3306, 8989) - Load balancer/router

3. **redis-cluster**
   - redis-node-0 through redis-node-5 (ports 6379-6384)
   - redis-cluster-init (one-time setup)

4. **valkey-cluster**
   - valkey-0 through valkey-5 (internal)
   - valkey-cluster-init (one-time setup)

## Configuration Management

### Environment Variables (from example.env)
```env
TZ=Asia/Bangkok              # Timezone
LANG=C.UTF-8                 # Language/locale
DB_USERNAME=common_user      # Shared database username
DB_PASSWORD=your_secure_password    # Shared database password
DB_NAME=common_database      # Shared database name
REPLICATION_USERNAME=replication_user
REPLICATION_PASSWORD=your_secure_password
ADMIN_UI_USERNAME=admin_user
ADMIN_UI_PASSWORD=your_secure_password
```

### Network Configuration
- **Network Name**: `ct_shared_network`
- **Purpose**: Inter-service communication
- **Scope**: All services connected
- **DNS**: Service name resolution (e.g., `postgres`, `redis`, `mongo`)

## Known Patterns

### Service Definition Pattern
Each service follows consistent structure:
```yaml
services:
  <service-name>:
    image: mirror.gcr.io/<image>:<version>
    container_name: <service-name>
    environment:
      - DB_USER=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_NAME=${DB_NAME}
    volumes:
      - <service>_data:/path/to/data
    ports:
      - "external:internal"
    restart: unless-stopped
    networks:
      - ct_shared_network

volumes:
  <service>_data:
```

### Cluster Pattern
Clusters include:
1. Multiple database nodes with health checks
2. Initialization container (one-time setup)
3. Load balancer/proxy (PgBouncer, MaxScale)
4. Dependency ordering via `depends_on` with health conditions
5. Inter-node communication via service names

### Volume Naming
- Simple: `<service>_data` (e.g., `postgres_data`)
- Cluster: `<service>_<role>_<number>_data` (e.g., `postgres_replica_1_data`)

## Common Operations

### Start Specific Services
```bash
podman compose up -d postgres redis mongo
```

### Start All Services
```bash
podman compose up -d
```

### View Service Status
```bash
podman compose ps
```

### View Logs
```bash
podman compose logs -f <service-name>
```

### Stop Services
```bash
podman compose down
```

### Reset Data (destructive)
```bash
podman compose down --volumes
```

### Access DBGate
```
http://dbgate.localhost:30080
```

## Next Steps & Opportunities

### Potential Enhancements
1. **Monitoring Stack**
   - Prometheus for metrics collection
   - Grafana for visualization
   - Service-specific exporters

2. **Backup Solutions**
   - Automated backup scripts
   - Volume snapshot utilities
   - Cross-database backup strategies

3. **Additional Databases**
   - Elasticsearch/OpenSearch for full-text search
   - Neo4j for graph workloads
   - TimescaleDB for time-series
   - InfluxDB for metrics storage

4. **Developer Tools**
   - Makefile for common operations
   - Health check dashboard
   - Service dependency visualization

5. **CI/CD Integration**
   - GitHub Actions workflows
   - Automated testing of compose files
   - Image vulnerability scanning

### Documentation Improvements
1. Architecture diagrams (cluster topologies)
2. Performance tuning guides
3. Troubleshooting section
4. Integration examples (application connection strings)
5. Comparison guide (when to use which database)

## Critical Files to Monitor

1. **compose.yaml** - Service inclusion list
2. **example.env** - Configuration template
3. **README.md** - User documentation
4. **databases/*/compose.yaml** - Individual service definitions

## Deployment Considerations

### Resource Requirements
- **Minimal** (1-2 services): 2GB RAM, 10GB disk
- **Medium** (5-7 services): 4GB RAM, 25GB disk
- **Full** (all services): 8GB+ RAM, 50GB+ disk
- **Clusters** (any): Add 2-4GB RAM per cluster

### Port Conflicts
Be aware of port usage:
- Standard ports: 5432 (PostgreSQL), 3306 (MariaDB), 27017 (MongoDB)
- Modified ports: 6380 (Redis single instance)
- Multiple ports: 6379-6384 (Redis cluster)

### Data Persistence
- All volumes persist data between restarts
- Use `--volumes` flag to delete data
- Regular backups recommended for important data

## Important Notes

1. **Image Source**: All images from `mirror.gcr.io` - provides reliability and consistent access
2. **Security**: Default credentials in `example.env` must be changed for any non-local use
3. **Compatibility**: Works with both Podman and Docker Compose
4. **Service Names**: Used for DNS resolution within `ct_shared_network`
5. **Cluster Init**: Some clusters have one-time init containers that exit after setup

## Questions for Future Clarification

1. Is there a specific use case driving this collection (company internal tool, open-source offering, learning platform)?
2. Are there plans to add monitoring/observability?
3. Should backup/restore utilities be added?
4. Is there interest in supporting additional databases?
5. Would a CLI tool for managing services be valuable?