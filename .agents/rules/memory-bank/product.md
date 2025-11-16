# db-compose - Product Specification

## Problem Statement

### Development Challenges

Developers and teams face several challenges when working with databases in local development environments:

1. **Setup Complexity** - Installing and configuring multiple database systems is time-consuming and error-prone
2. **Environment Inconsistency** - Different team members may have different database versions or configurations
3. **Port Conflicts** - Multiple databases often use conflicting default ports
4. **Configuration Management** - Managing credentials and settings across multiple services is tedious
5. **Cluster Testing** - Setting up high-availability configurations for testing is particularly difficult
6. **Resource Management** - Running multiple databases can be resource-intensive without proper isolation

### Current Solutions' Limitations

- **Manual Installation** - Time-consuming, version conflicts, system pollution
- **Individual Docker Commands** - Complex to manage, hard to reproduce, no coordination
- **Custom Scripts** - Maintenance burden, not portable, inconsistent across projects
- **Cloud Services** - Costs money, requires internet, not always suitable for development

## Solution: db-compose

A unified, containerized database collection that provides:

- **One-Command Setup** - Start any combination of databases instantly
- **Zero Configuration** - Sensible defaults with simple environment variable overrides
- **Complete Isolation** - Each database runs in its own container with dedicated volumes
- **Production Parity** - Configurations mirror production setups including clustering
- **Shared Network** - Services can communicate seamlessly for integration testing
- **Visual Management** - Built-in DBGate web interface for database administration

## User Personas

### 1. Full-Stack Developer (Primary)

**Background**: Works on applications using multiple databases  
**Needs**:
- Quick access to PostgreSQL for main application data
- Redis for caching and sessions
- MongoDB for analytics data
- RabbitMQ for async job processing

**Usage Pattern**:
```bash
# Start only needed services
podman compose up -d postgres redis mongo rabbitmq

# Access databases via DBGate
open http://dbgate.localhost:30080
```

**Pain Points Solved**:
- No need to install 4 different database systems
- Consistent configuration across team
- Easy to start/stop services as needed

### 2. Backend Developer Testing HA

**Background**: Building distributed systems requiring high availability  
**Needs**:
- Test PostgreSQL replication behavior
- Validate read/write splitting with PgBouncer
- Ensure application handles failover correctly

**Usage Pattern**:
```bash
# Start PostgreSQL cluster with replication
podman compose up -d postgres-cluster

# Application connects to PgBouncer on port 5432
# Reads distributed across replicas automatically
```

**Pain Points Solved**:
- HA testing without complex manual setup
- Production-like replication configuration
- Built-in connection pooling with PgBouncer

### 3. DevOps Engineer

**Background**: Setting up CI/CD pipelines and testing infrastructure  
**Needs**:
- Consistent database configurations for testing
- Easy integration with automated test suites
- Quick teardown and recreation of environments

**Usage Pattern**:
```bash
# CI pipeline script
cp example.env .env
podman compose up -d postgres
# Run tests
podman compose down --volumes
```

**Pain Points Solved**:
- Reproducible database environments
- Fast startup and teardown
- Volume management for clean state

### 4. Student/Learner

**Background**: Learning database technologies and architectures  
**Needs**:
- Easy access to different database types
- Compare performance and features
- Learn clustering and replication concepts

**Usage Pattern**:
```bash
# Try different databases
podman compose up -d clickhouse  # Analytics
podman compose up -d scylladb    # Wide-column
podman compose up -d redis-cluster  # Distributed cache
```

**Pain Points Solved**:
- No installation complexity
- Can explore enterprise features (clustering)
- Safe, isolated learning environment

## UX Goals

### Simplicity
- Single `.env` file for all configuration
- Intuitive service naming matching database names
- Clear documentation with examples

### Discoverability
- All available services listed in README
- Service dependencies automatically handled
- DBGate provides visual database exploration

### Flexibility
- Start all services or only specific ones
- Override defaults via environment variables
- Compose files can be used independently

### Reliability
- Health checks ensure services are ready
- Automatic restart on failures
- Images from reliable mirror registry

### Performance
- Minimal resource overhead with Alpine images
- Persistent volumes for data retention
- Optimized default configurations

## Key Features

### 1. Modular Service Architecture
Each database in its own directory with independent compose file, making it easy to:
- Understand individual service configuration
- Reuse services in other projects
- Add new services without affecting existing ones

### 2. Unified Configuration
Single `.env` file manages:
- Database credentials (username, password, database name)
- Replication credentials for clusters
- Admin UI credentials
- Timezone and locale settings

### 3. Advanced Clustering Support

**PostgreSQL Cluster**:
- 1 primary + 2 replicas with streaming replication
- PgBouncer for connection pooling and load balancing
- Automatic replica initialization via pg_basebackup

**MariaDB Galera Cluster**:
- 3-node multi-master synchronous replication
- MaxScale for query routing and load balancing
- Automatic cluster formation and node discovery

**Redis Cluster**:
- 6 nodes (3 masters + 3 replicas)
- Automatic cluster creation and sharding
- Built-in failover support

**Valkey Cluster**:
- 6 nodes (3 masters + 3 replicas)
- Redis-compatible clustering
- High availability configuration

### 4. Integrated Database Management
**DBGate** web interface provides:
- Connection to all services via shared network
- SQL query execution and result browsing
- Schema visualization
- Data import/export
- Multi-database support

### 5. Production-Ready Patterns
- Health checks for all critical services
- Proper dependency ordering
- Restart policies
- Volume management
- Environment-based configuration

## Use Cases

### Development Workflow
1. Developer clones project repository
2. Copies `example.env` to `.env` with secure passwords
3. Starts required databases: `podman compose up -d postgres redis`
4. Application connects to databases via localhost ports
5. Uses DBGate for database inspection and debugging

### Integration Testing
1. CI pipeline starts fresh database instances
2. Runs migrations and seeds test data
3. Executes test suite
4. Tears down with `podman compose down --volumes`
5. Ensures clean state for each test run

### High Availability Testing
1. Start clustered database configuration
2. Test application behavior during node failures
3. Verify read/write splitting
4. Validate automatic failover
5. Monitor replication lag and consistency

### Technology Evaluation
1. Start multiple database types simultaneously
2. Run same workload against different databases
3. Compare performance characteristics
4. Evaluate feature sets and query capabilities
5. Make informed technology decisions

### Training and Onboarding
1. New team members get consistent environment
2. Learn database concepts hands-on
3. Experiment without fear of breaking production
4. Practice operational procedures (backup, restore, failover)

## Success Criteria

### Functional Requirements
- ✅ All services start without errors
- ✅ Services communicate via shared network
- ✅ DBGate connects to all database types
- ✅ Cluster configurations achieve synchronization
- ✅ Health checks report correct status

### Non-Functional Requirements
- ✅ Services start within reasonable time (<2 minutes for clusters)
- ✅ Minimal memory footprint (Alpine images)
- ✅ Data persists across container restarts
- ✅ Configuration changes apply without rebuilding
- ✅ Clear error messages for common issues

### User Experience
- ✅ Setup takes less than 5 minutes
- ✅ Documentation is clear and complete
- ✅ Service names are intuitive
- ✅ Common tasks require single command
- ✅ Troubleshooting guidance available

## Future Enhancements

### Potential Additions
- ElasticSearch/OpenSearch for full-text search
- TimescaleDB for time-series data
- Neo4j for graph database workloads
- Apache Pulsar as Kafka alternative
- Consul for service discovery

### Configuration Improvements
- Service-specific configuration overrides
- Multiple environment profiles (dev, test, prod-like)
- Backup and restore utilities
- Monitoring stack (Prometheus + Grafana)

### Developer Experience
- CLI tool for common operations
- Health dashboard
- Automated testing of compose configurations
- Migration helpers between database versions