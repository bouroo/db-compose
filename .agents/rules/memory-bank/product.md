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
7. **Documentation Gaps** - Lack of comprehensive guides for different runtimes and use cases
8. **Production Parity** - Difficulty replicating production-like environments for development

### Current Solutions' Limitations

- **Manual Installation** - Time-consuming, version conflicts, system pollution
- **Individual Docker Commands** - Complex to manage, hard to reproduce, no coordination
- **Custom Scripts** - Maintenance burden, not portable, inconsistent across projects
- **Cloud Services** - Costs money, requires internet, not always suitable for development
- **Incomplete Documentation** - Many solutions lack comprehensive guides for different scenarios
- **Runtime Limitations** - Solutions often tied to a single container runtime

## Solution: db-compose

A unified, containerized database collection that provides:

- **One-Command Setup** - Start any combination of databases instantly
- **Zero Configuration** - Sensible defaults with simple environment variable overrides
- **Complete Isolation** - Each database runs in its own container with dedicated volumes
- **Production Parity** - Configurations mirror production setups including clustering
- **Shared Network** - Services can communicate seamlessly for integration testing
- **Visual Management** - Built-in DBGate web interface for database administration
- **Dual Runtime Support** - Works seamlessly with both Docker and Podman
- **Comprehensive Documentation** - Detailed guides for all use cases and runtimes

## User Personas

### 1. Full-Stack Developer (Primary)

**Background**: Works on applications using multiple databases  
**Needs**:
- Quick access to PostgreSQL for main application data
- Redis for caching and sessions
- MongoDB for analytics data
- RabbitMQ for async job processing
- Clear documentation for setup and configuration

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
- Comprehensive documentation for all scenarios

### 2. Backend Developer Testing HA

**Background**: Building distributed systems requiring high availability  
**Needs**:
- Test PostgreSQL replication behavior
- Validate read/write splitting with PgBouncer
- Ensure application handles failover correctly
- Detailed testing procedures and documentation

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
- Comprehensive testing documentation

### 3. DevOps Engineer

**Background**: Setting up CI/CD pipelines and testing infrastructure  
**Needs**:
- Consistent database configurations for testing
- Easy integration with automated test suites
- Quick teardown and recreation of environments
- Runtime-specific guides (Docker vs Podman)
- Production considerations and security best practices

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
- Runtime-specific documentation and guides

### 4. Student/Learner

**Background**: Learning database technologies and architectures  
**Needs**:
- Easy access to different database types
- Compare performance and features
- Learn clustering and replication concepts
- Comprehensive learning resources and examples
- Troubleshooting guides for common issues

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
- Comprehensive documentation for learning

### 5. Production Engineer

**Background**: Managing production database infrastructure  
**Needs**:
- Production-ready configurations
- Security best practices
- Performance optimization
- Backup and recovery strategies
- Resource requirements and scaling

**Usage Pattern**:
```bash
# Production deployment with resource limits
docker compose -f compose.yaml -f prod-overrides.yaml up -d

# Monitoring and maintenance
docker compose logs -f
docker stats
```

**Pain Points Solved**:
- Production-ready out of the box
- Security and performance documented
- Backup strategies and procedures
- Comprehensive production guidance

## UX Goals

### Simplicity
- Single `.env` file for all configuration
- Intuitive service naming matching database names
- Clear documentation with examples
- Runtime-specific quick start guides

### Discoverability
- All available services listed in README
- Service dependencies automatically handled
- DBGate provides visual database exploration
- Comprehensive documentation for all features

### Flexibility
- Start all services or only specific ones
- Override defaults via environment variables
- Compose files can be used independently
- Support for both Docker and Podman runtimes

### Reliability
- Health checks ensure services are ready
- Automatic restart on failures
- Images from reliable mirror registry
- Comprehensive error handling and logging

### Performance
- Minimal resource overhead with Alpine images
- Persistent volumes for data retention
- Optimized default configurations
- Performance tuning guidance provided

## Key Features

### 1. Modular Service Architecture
Each database in its own directory with independent compose file, making it easy to:
- Understand individual service configuration
- Reuse services in other projects
- Add new services without affecting existing ones
- Maintain consistent patterns across all services

### 2. Unified Configuration
Single `.env` file manages:
- Database credentials (username, password, database name)
- Replication credentials for clusters
- Admin UI credentials
- Timezone and locale settings
- Runtime-specific configurations

### 3. Advanced Clustering Support

**PostgreSQL Cluster**:
- 1 primary + 2 replicas with streaming replication
- PgBouncer for connection pooling and load balancing
- Automatic replica initialization via pg_basebackup
- Production-ready configuration with WAL settings

**MariaDB Galera Cluster**:
- 3-node multi-master synchronous replication
- MaxScale for query routing and load balancing
- Automatic cluster formation and node discovery
- Virtually synchronous replication

**Redis Cluster**:
- 6 nodes (3 masters + 3 replicas)
- Automatic cluster creation and sharding
- Built-in failover support
- 16384 hash slots distribution

**Valkey Cluster**:
- 6 nodes (3 masters + 3 replicas)
- Redis-compatible clustering
- High availability configuration
- Same architecture as Redis cluster

### 4. Integrated Database Management
**DBGate** web interface provides:
- Connection to all services via shared network
- SQL query execution and result browsing
- Schema visualization
- Data import/export
- Multi-database support
- Comprehensive connection management

### 5. Production-Ready Patterns
- Health checks for all critical services
- Proper dependency ordering
- Restart policies
- Volume management
- Environment-based configuration
- Security best practices
- Performance optimization guidance

### 6. Dual Runtime Support
- **Docker Compatibility**: Full support with Docker Compose v2+
- **Podman Compatibility**: Full support with Podman Compose
- **Runtime Comparison**: Clear guidance on choosing between runtimes
- **Migration Guides**: Easy migration between Docker and Podman

### 7. Comprehensive Documentation
- **Runtime-Specific Guides**: Separate guides for Docker and Podman
- **Configuration Management**: Detailed environment variable documentation
- **Production Considerations**: Resource requirements, security, backup strategies
- **Performance and Scaling**: Optimization strategies and tuning
- **Troubleshooting**: Common issues and solutions
- **Testing Procedures**: Unit, integration, and performance testing

## Use Cases

### Development Workflow
1. Developer clones project repository
2. Copies `example.env` to `.env` with secure passwords
3. Starts required databases: `podman compose up -d postgres redis`
4. Application connects to databases via localhost ports
5. Uses DBGate for database inspection and debugging
6. Refers to comprehensive documentation for guidance

### Integration Testing
1. CI pipeline starts fresh database instances
2. Runs migrations and seeds test data
3. Executes test suite
4. Tears down with `podman compose down --volumes`
5. Ensures clean state for each test run
6. Uses runtime-specific compose commands

### High Availability Testing
1. Start clustered database configuration
2. Test application behavior during node failures
3. Verify read/write splitting
4. Validate automatic failover
5. Monitor replication lag and consistency
6. Follow documented testing procedures

### Technology Evaluation
1. Start multiple database types simultaneously
2. Run same workload against different databases
3. Compare performance characteristics
4. Evaluate feature sets and query capabilities
5. Make informed technology decisions
6. Use performance testing tools documented

### Training and Onboarding
1. New team members get consistent environment
2. Learn database concepts hands-on
3. Experiment without fear of breaking production
4. Practice operational procedures (backup, restore, failover)
5. Follow comprehensive learning guides

### Production Deployment
1. Review production requirements and resource planning
2. Configure environment variables for production
3. Start services with production overrides
4. Implement monitoring and backup strategies
5. Follow security best practices
6. Use production-specific documentation

## Success Criteria

### Functional Requirements
- ✅ All services start without errors
- ✅ Services communicate via shared network
- ✅ DBGate connects to all database types
- ✅ Cluster configurations achieve synchronization
- ✅ Health checks report correct status
- ✅ Both Docker and Podman runtimes supported

### Non-Functional Requirements
- ✅ Services start within reasonable time (<2 minutes for clusters)
- ✅ Minimal memory footprint (Alpine images)
- ✅ Data persists across container restarts
- ✅ Configuration changes apply without rebuilding
- ✅ Clear error messages for common issues
- ✅ Comprehensive documentation available
- ✅ Runtime compatibility maintained

### User Experience
- ✅ Setup takes less than 5 minutes
- ✅ Documentation is clear and complete
- ✅ Service names are intuitive
- ✅ Common tasks require single command
- ✅ Troubleshooting guidance available
- ✅ Runtime-specific guidance provided
- ✅ Production guidance comprehensive

### Documentation Quality
- ✅ README.md comprehensive with all features
- ✅ Runtime-specific guides (Docker and Podman)
- ✅ Troubleshooting section with solutions
- ✅ Configuration management documented
- ✅ Production considerations covered
- ✅ Performance and scaling guidance
- ✅ Testing procedures and examples

## Future Enhancements

### Potential Additions
- ElasticSearch/OpenSearch for full-text search
- TimescaleDB for time-series data
- Neo4j for graph database workloads
- Apache Pulsar as Kafka alternative
- Consul for service discovery
- Monitoring stack (Prometheus + Grafana)

### Configuration Improvements
- Service-specific configuration overrides
- Multiple environment profiles (dev, test, prod)
- Backup and restore utilities
- Advanced monitoring and alerting
- Secret management integration

### Developer Experience
- CLI tool for common operations
- Health check dashboard
- Service dependency visualization
- Automated testing of compose configurations
- Migration helpers between database versions
- Enhanced debugging tools

### Runtime Enhancements
- Kubernetes deployment manifests
- Cloud provider integration
- Cross-runtime compatibility testing
- Performance benchmarking tools
- Resource optimization automation

### Documentation Expansion
- Architecture diagrams (cluster topologies)
- Performance tuning guides
- Troubleshooting section expansion
- Integration examples (application connection strings)
- Comparison guide (when to use which database)
- Video tutorials and walkthroughs