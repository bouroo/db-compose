# db-compose - Project Brief

## Project Overview

**db-compose** is a comprehensive containerized database collection monorepo that provides pre-configured, production-ready database and messaging services using Docker/Podman Compose. The project simplifies local development, testing, and production deployment by offering a complete suite of databases with consistent configuration management, comprehensive documentation, and dual runtime support.

## Core Requirements

### Primary Goals
1. Provide isolated, modular database service configurations
2. Enable centralized management of multiple database services
3. Support both standalone and clustered database deployments
4. Ensure consistent configuration across all services
5. Facilitate easy service selection and management
6. Provide comprehensive documentation for all use cases
7. Support both Docker and Podman container runtimes
8. Deliver production-ready configurations with best practices

### Service Categories

#### Relational Databases
- **PostgreSQL 18** - Single instance and cluster (with replication + PgBouncer)
- **MariaDB LTS** - Single instance and Galera cluster (with MaxScale)

#### Key-Value Stores
- **Redis 8** - Single instance and cluster (6 nodes)
- **Valkey 9** - Single instance and cluster (6 nodes)

#### Document Databases
- **MongoDB 8** - NoSQL document store
- **ScyllaDB 2025.3** - Cassandra-compatible wide-column store

#### OLAP/Analytics
- **ClickHouse LTS** - Columnar database for analytics

#### Message Brokers
- **Kafka 7.4.10** - Distributed event streaming (with Zookeeper)
- **RabbitMQ 4** - Message queue with management UI
- **NATS 2** - Cloud-native messaging system

#### Support Services
- **DBGate** - Web-based database client
- **Traefik 3** - Reverse proxy and load balancer

## Key Design Principles

1. **Modularity** - Each service defined independently in its own compose file
2. **Reusability** - Individual compose files can be used in other projects
3. **Consistency** - Shared environment variables and network configuration
4. **Reliability** - Images from `mirror.gcr.io` for stable access
5. **Scalability** - Easy expansion by adding new service definitions
6. **Production-Ready** - Health checks, restart policies, and proper clustering
7. **Documentation-First** - Comprehensive guides for all use cases and runtimes
8. **Dual Runtime Support** - Seamless compatibility with Docker and Podman

## Configuration Management

### Shared Environment Variables
- `TZ` - Timezone (default: Asia/Bangkok)
- `LANG` - Language settings (default: C.UTF-8)
- `DB_USERNAME` - Common database username
- `DB_PASSWORD` - Common database password
- `DB_NAME` - Common default database name
- `REPLICATION_USERNAME` - For database replication
- `REPLICATION_PASSWORD` - For database replication
- `ADMIN_UI_USERNAME` - For admin interfaces
- `ADMIN_UI_PASSWORD` - For admin interfaces

### Network Architecture
- All services share `ct_shared_network` for inter-service communication
- Services can reference each other by service name
- Ports exposed for external access
- Runtime-specific network configurations

### Runtime Support
- **Docker**: Full compatibility with Docker Compose v2.0+
- **Podman**: Full compatibility with Podman Compose v1.0+
- **Migration**: Easy migration between runtimes with comprehensive guides

## Usage Patterns

### Development Mode
- Start only required services: `podman compose up -d postgres redis`
- Use DBGate to manage databases through web interface
- Refer to comprehensive documentation for guidance
- Runtime choice based on preference and requirements

### Testing Mode
- Start specific database clusters for high-availability testing
- Test application against replicated setups
- Use runtime-specific compose commands
- Follow documented testing procedures

### Production Mode
- Start all services with production configurations
- Implement resource limits and security hardening
- Use comprehensive production documentation
- Follow backup and monitoring strategies

### Full Stack Mode
- Start all services: `podman compose up -d`
- Complete database ecosystem for comprehensive testing
- All 17 services available for complex scenarios

## Target Users

1. **Backend Developers** - Need quick access to various databases for development
2. **DevOps Engineers** - Testing infrastructure configurations and deployments
3. **QA Teams** - Validating application behavior across different databases
4. **Students/Learners** - Exploring different database technologies
5. **Production Engineers** - Managing database infrastructure with best practices
6. **Full-Stack Developers** - Working with multiple database types simultaneously
7. **Site Reliability Engineers** - Implementing high-availability solutions

## Success Metrics

1. All services start successfully with minimal configuration
2. Services communicate properly within shared network
3. Configuration changes via `.env` propagate correctly
4. Cluster configurations achieve proper synchronization
5. DBGate can connect to all database services
6. Both Docker and Podman runtimes work seamlessly
7. Comprehensive documentation covers all use cases
8. Production deployment follows documented best practices
9. Performance meets documented requirements
10. Backup and recovery procedures work as documented

## Documentation and Support

### Documentation Resources
- **README.md**: Comprehensive overview and getting started guide
- **DOCKER_USAGE.md**: Docker-specific instructions and best practices
- **PODMAN_USAGE.md**: Podman-specific instructions and best practices
- **TROUBLESHOOTING.md**: Comprehensive troubleshooting guide
- **MIGRATION_GUIDE.md**: Migration between Docker and Podman

### Support and Community
- GitHub issues for bug reports and feature requests
- Comprehensive documentation for self-service support
- Runtime-specific guides and examples
- Production best practices and recommendations

## License

MIT License - Copyright (c) 2025 Kawin Viriyaprasopsook

## Future Enhancements

### Potential Additions
- Monitoring stack (Prometheus + Grafana)
- Backup and automation utilities
- Additional databases (Elasticsearch, Neo4j, TimescaleDB)
- CI/CD integration templates
- Advanced security features
- Performance benchmarking tools

### Documentation Expansion
- Architecture diagrams and visualizations
- Video tutorials and walkthroughs
- Advanced configuration examples
- Integration guides for popular frameworks
- Performance tuning guides
- Security hardening procedures

## Contributing

### Development Guidelines
- Follow established patterns in existing services
- Update documentation when adding new features
- Test with both Docker and Podman runtimes
- Add comprehensive health checks
- Follow security best practices
- Update relevant documentation files

### Code Quality
- Consistent YAML formatting
- Clear service naming conventions
- Comprehensive health checks
- Proper resource management
- Security-conscious configuration
- Documentation for complex configurations

## Performance and Resource Requirements

### Minimum Requirements
- **CPU**: 4 cores
- **Memory**: 8GB RAM
- **Storage**: 50GB SSD
- **Network**: 1Gbps

### Recommended Requirements
- **CPU**: 8+ cores
- **Memory**: 16GB+ RAM
- **Storage**: 100GB+ SSD (NVME preferred)
- **Network**: 10Gbps

### Cluster Considerations
- Add 2-4GB RAM per cluster
- Consider storage requirements for data persistence
- Network bandwidth for inter-node communication
- Backup storage for critical data

## Security Considerations

### Default Configuration
- Suitable for development and testing environments
- Default credentials should be changed for production
- Network isolation provided by container runtime
- Regular security updates recommended

### Production Recommendations
- Use Docker Secrets or external secret management
- Implement network segmentation
- Enable TLS for all connections
- Regular security audits and updates
- Access control and authentication hardening
- Runtime-specific security configurations

### Best Practices
- Never commit `.env` files to version control
- Use strong, unique passwords
- Implement proper backup procedures
- Monitor resource usage and performance
- Keep images updated with security patches
- Follow principle of least privilege