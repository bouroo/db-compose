# db-compose - Current Context

**Last Updated**: 2025-11-17  
**Memory Bank Status**: [Memory Bank: Active]

## Project Understanding Summary

db-compose is a mature, stable containerized database collection monorepo providing 17 different database and messaging services through modular Docker/Podman Compose configurations. The project has evolved significantly with enhanced documentation, production-ready features, and improved user experience for both Docker and Podman runtimes.

## Current State

### Development Phase
**Phase**: Production-Ready with Enhanced Documentation (Mature Stable State)

The project has reached a mature state with comprehensive enhancements:
- ✅ All 17 database and messaging services implemented and tested
- ✅ Enhanced README.md with comprehensive documentation
- ✅ Production-ready configurations with detailed guidance
- ✅ Dual runtime support (Docker and Podman) with specific usage guides
- ✅ MIT licensed open-source project
- ✅ Performance optimization and scaling strategies documented

### Recent Changes & Enhancements
- **Major README.md Enhancement**: Significant improvements including:
  - Runtime comparison table (Docker vs Podman)
  - Detailed configuration options and service profiles
  - Production considerations with resource requirements
  - Performance and scaling guidance
  - Comprehensive troubleshooting section
  - Enhanced service discovery and networking documentation
  - Testing procedures and development workflows
- **Documentation Focus**: Improved user experience with clear guides for all skill levels
- **Production Readiness**: Enhanced security best practices, backup strategies, and performance optimization

### Active Work Focus
Currently updating memory bank to reflect recent documentation improvements and project maturity. The project has shifted from basic functionality to comprehensive documentation and user experience enhancements.

## Project Maturity Assessment

### Completeness: ~100%
**Implemented**:
- ✅ 17 services total (11 single-instance + 4 cluster + 2 support services)
- ✅ Comprehensive documentation with runtime-specific guides
- ✅ Production-ready patterns with security and performance guidance
- ✅ Centralized configuration management
- ✅ Shared networking infrastructure
- ✅ Enhanced troubleshooting and support documentation
- ✅ Performance optimization and scaling strategies

**No Longer Missing**:
- ✅ Documentation now complete with user guides, troubleshooting, and best practices
- ✅ Production considerations fully documented
- ✅ Performance guidance available

### Code Quality: High
- Consistent file organization across all services
- Standard patterns for health checks
- Proper dependency management
- Clear naming conventions
- Production-ready restart policies
- Enhanced documentation quality

## Directory Structure

```
db-compose/
├── .agents/rules/memory-bank/     # Project memory (ACTIVE)
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
├── README.md                     # Enhanced comprehensive documentation
├── DOCKER_USAGE.md               # Docker-specific guide
├── PODMAN_USAGE.md               # Podman-specific guide
├── TROUBLESHOOTING.md            # Comprehensive troubleshooting
├── MIGRATION_GUIDE.md            # Migration between runtimes
└── test-*.yaml                   # Testing configurations
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

## Runtime Support

### Docker Compatibility
- **Status**: Fully supported
- **Version**: Docker Compose v2.0+ (for include directive)
- **Features**: All services, networking, volumes, health checks
- **Documentation**: [DOCKER_USAGE.md](DOCKER_USAGE.md)

### Podman Compatibility
- **Status**: Fully supported
- **Version**: Podman Compose v1.0+
- **Features**: All services, rootless by default, daemonless
- **Documentation**: [PODMAN_USAGE.md](PODMAN_USAGE.md)

### Runtime Comparison
| Feature | Docker | Podman |
|---------|--------|--------|
| Industry Standard | ✅ | ❌ |
| Daemonless | ❌ | ✅ |
| Rootless by Default | ❌ | ✅ |
| Kubernetes Integration | ✅ | ✅ |
| GUI Management | ✅ (Docker Desktop) | ✅ (Podman Desktop) |

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

### Service Profiles
- **Development**: Start minimal services
- **Production**: Start all services with production settings
- **Custom**: Create custom service combinations

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
docker compose up -d postgres redis mongo
# or
podman compose up -d postgres redis mongo
```

### Start All Services
```bash
docker compose up -d
# or
podman compose up -d
```

### View Service Status
```bash
docker compose ps
# or
podman compose ps
```

### View Logs
```bash
docker compose logs -f <service-name>
# or
podman compose logs -f <service-name>
```

### Stop Services
```bash
docker compose down
# or
podman compose down
```

### Reset Data (destructive)
```bash
docker compose down --volumes
# or
podman compose down --volumes
```

### Access DBGate
```
http://dbgate.localhost:30080
```

## Production Considerations

### Resource Requirements
- **Minimal** (1-2 services): 4 cores, 8GB RAM, 50GB SSD
- **Recommended** (full stack): 8+ cores, 16GB+ RAM, 100GB+ SSD (NVME preferred)
- **Network**: 1Gbps minimum, 10Gbps recommended

### Security Best Practices
1. **Environment Variables**: Use strong, unique passwords
2. **Network Security**: Use private networks, implement firewall rules
3. **Container Security**: Regular updates, minimal base images, non-root users
4. **Data Protection**: Encryption for data at rest and in transit

### Backup Strategies
- **Database Backups**: Use database-specific tools (pg_dump, mongodump, etc.)
- **Volume Backups**: Create volume snapshots
- **Automated Scripts**: Implement scheduled backups with compression

## Testing and Development

### Testing Procedures
- **Unit Testing**: Test individual service connections
- **Integration Testing**: Test inter-service communication
- **Performance Testing**: Test database performance under load

### Development Workflow
1. Clone repository
2. Copy `example.env` to `.env` with secure credentials
3. Start required services
4. Use DBGate for database inspection and debugging
5. Test and develop applications

### Testing Scenarios
- Service restart behavior
- Network failure simulation
- Load testing with pgbench and similar tools

## Performance and Scaling

### Resource Optimization
- **Memory Management**: Configure resource limits
- **CPU Optimization**: Set CPU limits and reservations
- **Storage Optimization**: Use appropriate storage drivers

### Database-Specific Tuning
- **PostgreSQL**: Configure shared_buffers, work_mem, max_connections
- **Redis**: Set maxmemory and eviction policies
- **MariaDB**: Configure innodb_buffer_pool_size

### Scaling Strategies
- **Horizontal Scaling**: For stateless services (Kafka, NATS, RabbitMQ)
- **Vertical Scaling**: For stateful services (databases)
- **Database Clustering**: Use built-in cluster configurations

## Documentation Enhancements

### Recent README.md Improvements
1. **Runtime Selection**: Detailed comparison of Docker vs Podman
2. **Configuration Options**: Comprehensive environment variable documentation
3. **Service Discovery**: Enhanced networking and inter-service communication
4. **Production Considerations**: Resource requirements, security, backup strategies
5. **Development and Testing**: Procedures, workflows, and scenarios
6. **Performance and Scaling**: Optimization strategies and tuning
7. **Troubleshooting**: Common issues and solutions

### Supporting Documentation
- **DOCKER_USAGE.md**: Docker-specific instructions
- **PODMAN_USAGE.md**: Podman-specific instructions
- **TROUBLESHOOTING.md**: Comprehensive troubleshooting guide
- **MIGRATION_GUIDE.md**: Migration between runtimes

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
3. Integration examples (application connection strings)
4. Comparison guide (when to use which database)

## Critical Files to Monitor

1. **compose.yaml** - Service inclusion list
2. **example.env** - Configuration template
3. **README.md** - Enhanced user documentation
4. **databases/*/compose.yaml** - Individual service definitions
5. **DOCKER_USAGE.md** - Docker-specific guide
6. **PODMAN_USAGE.md** - Podman-specific guide
7. **TROUBLESHOOTING.md** - Troubleshooting section

## Deployment Considerations

### Port Conflicts
Be aware of port usage:
- Standard ports: 5432 (PostgreSQL), 3306 (MariaDB), 27017 (MongoDB)
- Modified ports: 6380 (Redis single instance)
- Multiple ports: 6379-6384 (Redis cluster)

### Data Persistence
- All volumes persist data between restarts
- Use `--volumes` flag to delete data
- Regular backups recommended for important data

### Runtime Selection
- **Docker**: Industry standard, broader ecosystem
- **Podman**: Daemonless, rootless by default, better security

## Important Notes

1. **Image Source**: All images from `mirror.gcr.io` - provides reliability and consistent access
2. **Security**: Default credentials in `example.env` must be changed for any non-local use
3. **Compatibility**: Works with both Podman and Docker Compose
4. **Service Names**: Used for DNS resolution within `ct_shared_network`
5. **Cluster Init**: Some clusters have one-time init containers that exit after setup
6. **Documentation**: Enhanced README.md provides comprehensive guidance for all use cases
7. **Dual Runtime**: Full support for both Docker and Podman with specific guides

## Questions for Future Clarification

1. Is there a specific use case driving this collection (company internal tool, open-source offering, learning platform)?
2. Are there plans to add monitoring/observability?
3. Should backup/restore utilities be added?
4. Is there interest in supporting additional databases?
5. Would a CLI tool for managing services be valuable?
6. What are the primary user personas for the enhanced documentation?