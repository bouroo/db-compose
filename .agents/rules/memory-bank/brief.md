# db-compose - Project Brief

## Project Overview

**db-compose** is a containerized database collection monorepo that provides pre-configured, production-ready database and messaging services using Docker/Podman Compose. The project simplifies local development and testing by offering a comprehensive suite of databases with consistent configuration management.

## Core Requirements

### Primary Goals
1. Provide isolated, modular database service configurations
2. Enable centralized management of multiple database services
3. Support both standalone and clustered database deployments
4. Ensure consistent configuration across all services
5. Facilitate easy service selection and management

### Service Categories

#### Relational Databases
- **PostgreSQL 18** - Single instance and cluster (with replication + PgBouncer)
- **MariaDB LTS** - Single instance and Galera cluster (with MaxScale)

#### Key-Value Stores
- **Redis 8** - Single instance and cluster (6 nodes)
- **Valkey 8** - Single instance and cluster (6 nodes)

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

## Usage Patterns

### Development Mode
- Start only required services: `podman compose up -d postgres redis`
- Use DBGate to manage databases through web interface

### Testing Mode
- Start specific database clusters for high-availability testing
- Test application against replicated setups

### Full Stack Mode
- Start all services: `podman compose up -d`
- Complete database ecosystem for comprehensive testing

## Target Users

1. **Backend Developers** - Need quick access to various databases for development
2. **DevOps Engineers** - Testing infrastructure configurations
3. **QA Teams** - Validating application behavior across different databases
4. **Students/Learners** - Exploring different database technologies

## Success Metrics

1. All services start successfully with minimal configuration
2. Services communicate properly within shared network
3. Configuration changes via `.env` propagate correctly
4. Cluster configurations achieve proper synchronization
5. DBGate can connect to all database services

## License

MIT License - Copyright (c) 2025 Kawin Viriyaprasopsook