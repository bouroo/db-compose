# db-compose - System Architecture

## Architectural Overview

db-compose implements a **Modular Monorepo Architecture** using Docker/Podman Compose's `include` directive to create a unified, manageable collection of independent database services.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     Root compose.yaml                        │
│                    (Service Orchestrator)                    │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                │  include directives   │
                └───────────┬───────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Single DBs  │    │   Clusters   │    │   Support    │
├──────────────┤    ├──────────────┤    ├──────────────┤
│ postgres     │    │ postgres-    │    │ dbgate       │
│ mariadb      │    │  cluster     │    │ traefik      │
│ mongo        │    │ mariadb-     │    └──────────────┘
│ redis        │    │  galera      │
│ valkey       │    │ redis-       │
│ clickhouse   │    │  cluster     │
│ scylladb     │    │ valkey-      │
│ kafka        │    │  cluster     │
│ rabbitmq     │    └──────────────┘
│ nats         │
└──────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │  ct_shared_network    │
                │  (Service Discovery)  │
                └───────────────────────┘
```

## Core Architectural Principles

### 1. Modularity (ADR-001)

**Decision**: Each database service is defined in its own isolated compose file.

**Rationale**:
- Services can be developed and maintained independently
- Individual compose files are reusable in other projects
- Changes to one service don't affect others
- Easier to understand and troubleshoot

**Implementation**:
```
databases/<service-name>/compose.yaml
```

Each file is self-contained with:
- Service definition
- Volume declarations
- Service-specific configuration

### 2. Composition over Inheritance (ADR-002)

**Decision**: Use Docker Compose `include` directive rather than service extension or complex inheritance.

**Rationale**:
- Clear, explicit service inclusion
- No hidden dependencies or overrides
- Easy to see which services are active
- Simple to add/remove services

**Implementation**:
```yaml
# compose.yaml
include:
  - ./databases/postgres/compose.yaml
  - ./databases/redis/compose.yaml
  # ... more services
```

### 3. Shared Configuration (ADR-003)

**Decision**: Use `.env` file for common configuration across all services.

**Rationale**:
- Single source of truth for credentials
- Easy to change configuration globally
- Prevents configuration drift between services
- Supports multiple environments (dev, test, prod-like)

**Implementation**:
```env
# .env (generated from example.env)
DB_USERNAME=common_user
DB_PASSWORD=secure_password
DB_NAME=common_database
```

All services reference: `${DB_USERNAME}`, `${DB_PASSWORD}`, `${DB_NAME}`

### 4. Service Discovery via Shared Network (ADR-004)

**Decision**: All services join a common Docker network (`ct_shared_network`).

**Rationale**:
- Services can communicate using service names as hostnames
- No need for IP address management
- Automatic DNS resolution
- Isolated from host network

**Implementation**:
```yaml
networks:
  ct_shared_network:
    name: ct_shared_network

services:
  postgres:
    networks:
      - ct_shared_network
```

Services reference each other by name: `postgres:5432`, `redis:6379`, etc.

### 5. Production-Ready Patterns (ADR-005)

**Decision**: Include health checks, restart policies, and proper dependency ordering.

**Rationale**:
- Services automatically recover from failures
- Dependencies start in correct order
- Health status is visible and actionable
- Mirrors production configurations

**Implementation**:
```yaml
services:
  postgres-replica:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    depends_on:
      postgres-primary:
        condition: service_healthy
```

## Service Patterns

### Pattern 1: Simple Single-Instance Service

**Use Cases**: Development databases, basic testing

**Components**:
- Single container
- Named volume for persistence
- External port mapping
- Standard health check

**Example**: PostgreSQL single instance

```yaml
services:
  postgres:
    image: mirror.gcr.io/postgresql:18-alpine
    container_name: postgres
    environment:
      - DB_USER=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_NAME=${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped
    networks:
      - ct_shared_network

volumes:
  postgres_data:
```

**Data Flow**:
```
Application → localhost:5432 → postgres container → postgres_data volume
```

### Pattern 2: Primary-Replica Cluster with Connection Pooler

**Use Cases**: HA testing, read scaling, failover scenarios

**Components**:
- 1 primary database (read-write)
- N replica databases (read-only)
- Connection pooler (PgBouncer)
- Initialization container (one-time setup)

**Example**: PostgreSQL Cluster

**Architecture**:
```
                    ┌──────────────┐
                    │  PgBouncer   │
                    │  (port 5432) │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│   Primary     │  │   Replica 1   │  │   Replica 2   │
│   (Master)    │  │   (Standby)   │  │   (Standby)   │
└───────┬───────┘  └───────▲───────┘  └───────▲───────┘
        │                  │                  │
        └──────────────────┴──────────────────┘
              WAL Streaming Replication
```

**Data Flow**:
1. Application connects to PgBouncer (port 5432)
2. PgBouncer routes writes to primary
3. PgBouncer distributes reads across all nodes
4. Primary streams WAL to replicas
5. Replicas apply WAL for consistency

**Key Features**:
- Streaming replication with WAL
- Automatic replica initialization via `pg_basebackup`
- Connection pooling reduces connection overhead
- Read load distribution

### Pattern 3: Multi-Master Cluster with Load Balancer

**Use Cases**: High availability, write scaling, zero-downtime deployments

**Components**:
- N database nodes (all read-write capable)
- Load balancer/query router
- Cluster synchronization protocol

**Example**: MariaDB Galera Cluster

**Architecture**:
```
                    ┌──────────────┐
                    │   MaxScale   │
                    │ (Router/LB)  │
                    │ (port 3306)  │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  Galera Node  │  │  Galera Node  │  │  Galera Node  │
│      0        │  │      1        │  │      2        │
│  (Master)     │  │  (Master)     │  │  (Master)     │
└───────┬───────┘  └───────┬───────┘  └───────┬───────┘
        │                  │                  │
        └──────────────────┴──────────────────┘
           Galera Synchronous Replication
```

**Data Flow**:
1. Application connects to MaxScale
2. MaxScale routes queries based on policy
3. Writes go to one node, replicated synchronously
4. Reads distributed across all nodes
5. All nodes maintain identical state

**Key Features**:
- Multi-master synchronous replication
- Automatic conflict resolution
- Virtually synchronous replication
- No data loss on node failure

### Pattern 4: Distributed Hash Cluster

**Use Cases**: Horizontal scaling, partitioning, fault tolerance

**Components**:
- N master nodes (data owners)
- N replica nodes (redundancy)
- Cluster initialization container
- Automatic sharding and failover

**Example**: Redis/Valkey Cluster

**Architecture**:
```
┌─────────────────────────────────────────────────────┐
│              Redis Cluster (6 nodes)                │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │ Master 0 │  │ Master 1 │  │ Master 2 │         │
│  │ Slots:   │  │ Slots:   │  │ Slots:   │         │
│  │ 0-5461   │  │ 5462-    │  │ 10923-   │         │
│  │          │  │ 10922    │  │ 16383    │         │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘         │
│       │             │             │                │
│       │             │             │                │
│  ┌────▼─────┐  ┌────▼─────┐  ┌────▼─────┐         │
│  │ Replica  │  │ Replica  │  │ Replica  │         │
│  │    3     │  │    4     │  │    5     │         │
│  └──────────┘  └──────────┘  └──────────┘         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Data Flow**:
1. Client issues command
2. Cluster determines which slot/shard owns the key
3. Command redirected to appropriate master
4. Data replicated to corresponding replica
5. Automatic failover if master fails

**Key Features**:
- Automatic sharding (16384 hash slots)
- Master-replica pairs for each shard
- Automatic failover and recovery
- Cluster topology awareness

## Component Interactions

### Service-to-Service Communication

**DBGate → Databases**:
```
DBGate (http://dbgate.localhost:30080)
  │
  ├─→ postgres:5432
  ├─→ mariadb:3306
  ├─→ mongo:27017
  ├─→ redis:6379
  └─→ ... (all other databases via service names)
```

**Application → Cluster**:
```
Application
  │
  └─→ PgBouncer/MaxScale (exposed port)
       │
       ├─→ Primary/Master nodes
       └─→ Replica/Standby nodes
```

### Volume Management

**Volume Lifecycle**:
1. **Creation**: Automatic on first service start
2. **Persistence**: Survives container restart/recreation
3. **Deletion**: Explicit via `--volumes` flag

**Volume Naming Convention**:
- Single instance: `<service>_data`
- Cluster: `<service>_<role>_<number>_data`

**Volume Locations** (typical):
- Podman: `~/.local/share/containers/storage/volumes/`
- Docker: `/var/lib/docker/volumes/`

## Configuration Flow

```
example.env (template)
    │
    ▼
.env (user copy with secrets)
    │
    ▼
${VARIABLE} references in compose files
    │
    ▼
Container environment variables
    │
    ▼
Database initialization/configuration
```

## High Availability Patterns

### PostgreSQL HA Strategy

**Replication**: Streaming (asynchronous by default)
**Failover**: Manual (requires external tool like Patroni for auto-failover)
**Split-Brain Protection**: Not included (single primary architecture)
**Connection Pooling**: PgBouncer reduces connection overhead

**Topology**:
- 1 primary (writes)
- 2+ replicas (reads)
- PgBouncer (connection pool/router)

### MariaDB HA Strategy

**Replication**: Galera (virtually synchronous)
**Failover**: Automatic (built into Galera)
**Split-Brain Protection**: Quorum-based (requires majority)
**Load Balancing**: MaxScale query routing

**Topology**:
- 3+ nodes (all masters)
- Odd number for quorum
- MaxScale as single entry point

### Redis HA Strategy

**Replication**: Asynchronous master-replica
**Failover**: Automatic (cluster consensus)
**Partitioning**: Hash slot distribution (16384 slots)
**Client Redirection**: MOVED/ASK responses

**Topology**:
- 3+ master nodes (sharding)
- 3+ replica nodes (redundancy)
- Minimum 6 nodes for standard setup

## Security Architecture

### Network Isolation

**Principles**:
- Services communicate via private network
- Only necessary ports exposed to host
- No direct container-to-container IP addressing

**Implementation**:
```yaml
networks:
  ct_shared_network:
    name: ct_shared_network
    # Uses bridge driver by default
    # Isolated from external networks
```

### Credential Management

**Current Approach**:
- Environment variables via `.env`
- `.env` excluded from git (`.gitignore`)
- Template provided (`example.env`)

**Security Considerations**:
- ⚠️ Suitable for development only
- ❌ Not recommended for production
- ✅ Easy to rotate credentials
- ✅ Single point of update

**Production Recommendations**:
- Use Docker Secrets for sensitive data
- Integrate with secret management (Vault, AWS Secrets Manager)
- Implement secret rotation policies

## Performance Considerations

### Image Selection

**Alpine-based images** (where available):
- Smaller size (faster pulls, less disk usage)
- Reduced attack surface
- Minimal dependencies

**Mirror Registry** (`mirror.gcr.io`):
- Reliable availability
- Consistent access
- Reduced rate limiting

### Resource Optimization

**Connection Pooling**:
- PgBouncer for PostgreSQL
- MaxScale for MariaDB
- Reduces connection overhead
- Better resource utilization

**Volume I/O**:
- Named volumes (Docker-managed)
- Optimized for database workloads
- Consider bind mounts for specific scenarios

### Scaling Patterns

**Vertical Scaling** (single instance):
- Increase container resource limits
- Tune database-specific parameters
- Monitor with health checks

**Horizontal Scaling** (clusters):
- Add more replicas (read scaling)
- Partition data (write scaling via sharding)
- Load balance across nodes

## Observability

### Health Checks

**Purpose**:
- Verify service readiness
- Enable dependency ordering
- Support automated recovery

**Implementation Pattern**:
```yaml
healthcheck:
  test: ["CMD-SHELL", "<health-check-command>"]
  interval: 5s
  timeout: 5s
  retries: 5
```

**Examples**:
- PostgreSQL: `pg_isready -U ${DB_USERNAME} -d ${DB_NAME}`
- Redis: `redis-cli -a ${DB_PASSWORD} ping`
- MariaDB: `mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS"`

### Logging

**Docker/Podman Logs**:
```bash
podman compose logs -f <service-name>
```

**Log Drivers** (configurable):
- json-file (default)
- syslog
- journald
- etc.

### Future Observability

**Potential Additions**:
- Prometheus exporters per database
- Grafana dashboards
- Centralized logging (ELK, Loki)
- Distributed tracing

## Deployment Topology

### Development Environment

```
Developer Machine
├── Podman/Docker Engine
├── db-compose project
│   ├── Selected services running
│   ├── DBGate for management
│   └── Application connecting locally
└── Volumes for data persistence
```

**Characteristics**:
- Start only needed services
- Quick iteration
- Data persists across restarts
- Easy cleanup

### CI/CD Environment

```
CI Runner
├── Fresh checkout
├── Copy example.env to .env
├── Start required services
├── Run tests
├── Tear down with --volumes
└── Clean state for next run
```

**Characteristics**:
- Isolated test runs
- Reproducible environments
- Fast setup/teardown
- No state carry-over

## Extension Points

### Adding New Database

**Steps**:
1. Create `databases/<new-db>/compose.yaml`
2. Follow standard service pattern
3. Add to root `compose.yaml` includes
4. Document in README
5. Test connectivity via DBGate

**Template**:
```yaml
services:
  <new-db>:
    image: mirror.gcr.io/<image>:<tag>
    container_name: <new-db>
    environment:
      - DB_USER=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_NAME=${DB_NAME}
    volumes:
      - <new-db>_data:/path/to/data
    ports:
      - "<external>:<internal>"
    restart: unless-stopped
    networks:
      - ct_shared_network

volumes:
  <new-db>_data:
```

### Custom Service Configuration

**Per-Service Overrides**:
- Create service-specific `.env` file
- Use `env_file` directive in compose
- Override specific environment variables

**Example**:
```yaml
services:
  postgres:
    env_file:
      - .env
      - ./databases/postgres/.env.local
```

## Architecture Decision Records (ADRs)

### ADR-001: Modular Service Definitions
- **Status**: Accepted
- **Decision**: One compose file per service
- **Consequences**: +maintainability, +reusability, +clarity

### ADR-002: Include-based Composition
- **Status**: Accepted
- **Decision**: Use `include` directive
- **Consequences**: +explicit, +simple, -requires Compose v2

### ADR-003: Centralized Configuration
- **Status**: Accepted
- **Decision**: Single `.env` file for common config
- **Consequences**: +consistency, +ease of use, -flexibility for service-specific needs

### ADR-004: Shared Network
- **Status**: Accepted
- **Decision**: Single network for all services
- **Consequences**: +service discovery, +simple communication, -network isolation

### ADR-005: Production-Ready Patterns
- **Status**: Accepted
- **Decision**: Include health checks and restart policies
- **Consequences**: +reliability, +operational readiness, +complexity

### ADR-006: Mirror Registry
- **Status**: Accepted
- **Decision**: Use `mirror.gcr.io` for all images
- **Consequences**: +reliability, +consistency, -vendor lock-in

## Limitations and Trade-offs

### Current Limitations

1. **No Built-in Monitoring**: Requires external tools
2. **Manual Failover**: Clusters don't auto-promote replicas (except Galera/Redis)
3. **No Backup Automation**: Manual backup procedures required
4. **Development Focus**: Security suitable for dev, not production
5. **Single Network**: No network segmentation between service types

### Design Trade-offs

| Aspect | Choice | Trade-off |
|--------|--------|-----------|
| Configuration | Shared `.env` | Simplicity vs. per-service flexibility |
| Images | Alpine-based | Size vs. feature completeness |
| Registry | Mirror GCR | Reliability vs. vendor independence |
| Network | Single shared | Simplicity vs. security isolation |
| Clusters | Pre-configured | Ease of use vs. customization |

## Future Architecture Considerations

1. **Service Mesh**: Integrate Consul/Istio for advanced networking
2. **Secret Management**: Vault integration for production use
3. **Monitoring**: Prometheus + Grafana stack
4. **Backup**: Automated backup/restore utilities
5. **Multi-Environment**: Dev/test/prod configuration profiles