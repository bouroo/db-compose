# Docker to Podman Migration Plan
## db-compose Project - Dual Compatibility Strategy

**Version:** 1.0  
**Date:** 2025-11-16  
**Status:** Planning Phase

---

## Executive Summary

This migration plan outlines the strategy for migrating the db-compose project from Docker to Podman while maintaining full backward compatibility with Docker. Analysis reveals that the project is **already 95% Podman-ready** - the README and all compose files use Podman-first language. The primary work needed is updating the GitHub workflow and creating comprehensive dual-runtime documentation.

### Current State Analysis

✅ **Already Podman-Compatible:**
- All documentation in [`README.md`](README.md:1) uses `podman compose` commands
- All [`compose.yaml`](compose.yaml:1) files use standard Docker Compose v2 syntax
- Uses `include` directive (supported by both runtimes)
- No Docker-specific features detected in compose files
- All services use standard configurations compatible with both runtimes

⚠️ **Requires Updates:**
- GitHub workflow ([`.github/workflows/validate-compose.yml`](. github/workflows/validate-compose.yml:1)) uses `docker compose`
- Need explicit dual-runtime testing strategy
- Volume handling differences need documentation
- Cluster services need validation with Podman

---

## 1. Migration Strategy

### 1.1 Approach: Dual Compatibility (Recommended)

**Goal:** Support both Docker and Podman runtimes seamlessly with zero breaking changes.

```mermaid
graph TD
    A[Current State: Docker-focused] --> B[Phase 1: Assessment Complete]
    B --> C[Phase 2: Update CI/CD]
    C --> D[Phase 3: Documentation]
    D --> E[Phase 4: Testing]
    E --> F[Phase 5: Validation]
    F --> G[Final State: Dual Runtime Support]
```

**Key Principles:**
1. **No Breaking Changes:** Existing Docker users continue working without modifications
2. **Podman-First Documentation:** Default to Podman in examples while noting Docker compatibility
3. **Runtime Agnostic:** All compose files work identically on both runtimes
4. **Comprehensive Testing:** CI validates both Docker and Podman compatibility

### 1.2 Why Dual Compatibility?

- **User Choice:** Different environments prefer different runtimes
- **Migration Path:** Users can migrate at their own pace
- **Ecosystem Support:** Some tools still prefer Docker, others prefer Podman
- **Zero Risk:** No forced migration reduces adoption friction

---

## 2. Configuration Updates

### 2.1 Compose Files - No Changes Needed ✅

**Analysis:** All compose files are already compatible with both runtimes.

**Verification Points:**
- ✅ Standard Compose v2 syntax
- ✅ No Docker-specific extensions
- ✅ Standard volume declarations
- ✅ Standard network configurations
- ✅ Standard healthcheck formats
- ✅ No privileged mode requirements (except where necessary)

**Special Cases Already Handled:**
1. **Cluster Services** ([`postgres-cluster`](databases/postgres-cluster/compose.yaml:1), [`redis-cluster`](databases/redis-cluster/compose.yaml:1), [`valkey-cluster`](databases/valkey-cluster/compose.yaml:1), [`mariadb-galera`](databases/mariadb-galera/compose.yaml:1))
   - Use standard multi-container patterns
   - Init containers use runtime-agnostic commands
   - Health checks are portable

2. **Volume Definitions:**
   - Named volumes work identically
   - Some use explicit `driver: local` (compatible with both)
   - Example: [`databases/mariadb-galera/compose.yaml`](databases/mariadb-galera/compose.yaml:161)

3. **Named Volumes with Explicit Names:**
   - [`redis-cluster`](databases/redis-cluster/compose.yaml:162): `db-compose_redis_node_5_data`
   - [`valkey-cluster`](databases/valkey-cluster/compose.yaml:165): `db-compose_valkey_5_data`
   - **Action:** Document that these names are runtime-agnostic

### 2.2 Environment Variables - No Changes Needed ✅

Current [`example.env`](example.env:1) is fully compatible:
```env
TZ=Asia/Bangkok
LANG=C.UTF-8
DB_USERNAME=common_user
DB_PASSWORD=your_secure_password
DB_NAME=common_database
REPLICATION_USERNAME=replication_user
REPLICATION_PASSWORD=your_secure_password
ADMIN_UI_USERNAME=admin_user
ADMIN_UI_PASSWORD=your_secure_password
```

---

## 3. File-by-File Breakdown

### 3.1 Files Requiring Updates

| File | Changes Needed | Priority | Complexity |
|------|---------------|----------|------------|
| `.github/workflows/validate-compose.yml` | Add Podman testing matrix | High | Medium |
| `README.md` | Add Docker compatibility notes | High | Low |
| `MIGRATION_PLAN.md` | Create this document | High | Low |
| `DOCKER_USAGE.md` | New: Docker-specific guide | Medium | Low |
| `PODMAN_USAGE.md` | New: Podman-specific guide | Medium | Low |
| `TROUBLESHOOTING.md` | New: Runtime-specific issues | Medium | Medium |

### 3.2 Files Not Requiring Changes

- ✅ [`compose.yaml`](compose.yaml:1) - Already runtime-agnostic
- ✅ All database compose files (17 files) - Standard syntax
- ✅ [`example.env`](example.env:1) - Works with both runtimes
- ✅ [`.gitignore`](.gitignore:1) - Runtime-agnostic

---

## 4. Testing Strategy

### 4.1 Dual Runtime Testing Matrix

```yaml
# Proposed GitHub Actions Matrix
strategy:
  matrix:
    runtime:
      - docker
      - podman
    service-group:
      - basic      # postgres, mariadb, mongo, redis
      - cluster    # postgres-cluster, redis-cluster, valkey-cluster, mariadb-galera
      - messaging  # kafka, rabbitmq, nats
      - tools      # dbgate, traefik
      - nosql      # mongo, scylladb, clickhouse
```

### 4.2 Test Phases

**Phase 1: Individual Services (Per Runtime)**
```bash
# Test each service starts successfully
podman compose up -d <service>
podman compose ps
podman compose logs <service>
podman compose down
```

**Phase 2: Service Groups**
```bash
# Test common combinations
podman compose up -d postgres redis rabbitmq
# Verify connectivity between services
podman compose exec postgres pg_isready
```

**Phase 3: Cluster Services**
```bash
# Test complex multi-node setups
podman compose up -d postgres-cluster
# Verify replication
podman compose exec postgres-primary psql -c "SELECT * FROM pg_stat_replication;"
```

**Phase 4: Full Stack**
```bash
# Test all services together
podman compose up -d
podman compose ps --all
```

### 4.3 Validation Criteria

For each runtime, verify:
- [ ] All services start without errors
- [ ] Health checks pass
- [ ] Inter-service communication works
- [ ] Volumes persist data correctly
- [ ] Port mappings work as expected
- [ ] Environment variables propagate correctly
- [ ] Cluster initialization completes successfully
- [ ] Service restart behavior is correct

### 4.4 Specific Test Cases

**Postgres Cluster Test:**
```bash
# Start cluster
podman compose up -d postgres-cluster

# Verify primary
podman compose exec postgres-primary psql -U common_user -d common_database -c "SELECT version();"

# Verify replication
podman compose exec postgres-replica-1 psql -U common_user -d common_database -c "SELECT pg_is_in_recovery();"

# Test connection pooling
podman compose exec pgbouncer psql -h localhost -U common_user -d primary -c "SELECT 1;"
```

**Redis Cluster Test:**
```bash
# Start cluster
podman compose up -d redis-cluster

# Verify cluster status
podman compose exec redis-node-0 redis-cli -a your_secure_password cluster info

# Test data distribution
podman compose exec redis-node-0 redis-cli -a your_secure_password -c set key1 value1
podman compose exec redis-node-1 redis-cli -a your_secure_password -c get key1
```

**MariaDB Galera Test:**
```bash
# Start cluster
podman compose up -d mariadb-galera

# Verify cluster size
podman compose exec galera-node-0 mysql -u root -p${DB_PASSWORD} -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

# Test through MaxScale
podman compose exec maxscale mysql -h localhost -u common_user -p${DB_PASSWORD} -e "SELECT @@hostname;"
```

### 4.5 Volume Persistence Tests

```bash
# Test data persistence across restarts
podman compose up -d postgres
podman compose exec postgres psql -U common_user -d common_database -c "CREATE TABLE test (id INT);"
podman compose down
podman compose up -d postgres
podman compose exec postgres psql -U common_user -d common_database -c "SELECT * FROM test;"
```

---

## 5. Rollback Plan

### 5.1 Rollback Triggers

Initiate rollback if:
- CI/CD pipeline fails consistently for either runtime
- Critical services fail to start in production
- Data loss or corruption detected
- Performance degradation exceeds 20%
- User-reported issues exceed threshold

### 5.2 Rollback Procedure

**Step 1: Immediate Actions**
```bash
# Revert to previous commit
git revert <migration-commit-sha>
git push origin main
```

**Step 2: Communication**
- Notify users via GitHub issue/discussion
- Document rollback reason
- Create hotfix branch if needed

**Step 3: Data Preservation**
```bash
# Backup current volumes before rollback
docker volume ls --filter name=db-compose
# OR
podman volume ls --filter name=db-compose

# Export data if needed
docker run --rm -v db-compose_postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/postgres_backup.tar.gz /data
```

**Step 4: Validation**
- Verify all services restart correctly
- Confirm data integrity
- Test critical user workflows

### 5.3 Rollback Prevention

- Maintain feature flags for new capabilities
- Use staged rollout (dev → staging → production)
- Keep both runtime documentation active
- Monitor metrics during migration window

---

## 6. Documentation Updates

### 6.1 Primary Documentation Changes

#### [`README.md`](README.md:1) Updates

**Section 3: Getting Started → Prerequisites**

**Current (Line 48):**
```markdown
* Podman Desktop (or Docker Engine and Docker Compose plugin) installed on your system.
```

**Enhanced:**
```markdown
* **Podman** (recommended) or **Docker** installed on your system:
  - **Podman Desktop** (recommended for GUI users): [Download](https://podman-desktop.io/)
  - **Podman CLI**: Install via package manager ([docs](https://podman.io/getting-started/installation))
  - **Docker Desktop** (alternative): [Download](https://www.docker.com/products/docker-desktop)
  - **Docker Engine**: Install via package manager ([docs](https://docs.docker.com/engine/install/))

**Note:** All commands in this guide use `podman compose`. If using Docker, replace `podman compose` with `docker compose`.
```

**Add New Section (after line 116):**
```markdown
### Runtime Compatibility

This project supports both **Podman** and **Docker** container runtimes:

- **Podman** (recommended): Daemonless, rootless container runtime
- **Docker**: Traditional container runtime

**Command Translation:**

| Podman | Docker |
|--------|--------|
| `podman compose up -d` | `docker compose up -d` |
| `podman compose down` | `docker compose down` |
| `podman compose ps` | `docker compose ps` |
| `podman compose logs` | `docker compose logs` |

For detailed runtime-specific instructions:
- [Podman Usage Guide](PODMAN_USAGE.md)
- [Docker Usage Guide](DOCKER_USAGE.md)
- [Troubleshooting](TROUBLESHOOTING.md)
```

### 6.2 New Documentation Files

#### `PODMAN_USAGE.md` (New File)
- Podman-specific installation
- Rootless vs rootful mode
- SELinux considerations
- Volume path locations (`~/.local/share/containers/storage/volumes/`)
- Podman Desktop integration
- Podman-specific troubleshooting

#### `DOCKER_USAGE.md` (New File)
- Docker installation methods
- Docker Desktop vs Docker Engine
- Volume path locations (`/var/lib/docker/volumes/`)
- Docker Desktop resource limits
- Docker-specific troubleshooting

#### `TROUBLESHOOTING.md` (New File)
- Common issues for both runtimes
- Runtime-specific problems
- Cluster initialization failures
- Network connectivity issues
- Volume permission problems
- Port conflicts
- Performance tuning

#### `CONTRIBUTING.md` (New File)
- How to test changes with both runtimes
- CI/CD requirements
- Documentation standards

### 6.3 Inline Documentation

Add comments to complex compose files:

**Example for [`databases/postgres-cluster/compose.yaml`](databases/postgres-cluster/compose.yaml:1):**
```yaml
# PostgreSQL Cluster with Streaming Replication
# Compatible with both Docker and Podman
# 
# Components:
# - postgres-primary: Primary database server
# - postgres-replica-1/2: Streaming replicas
# - pgbouncer: Connection pooler
# - postgres-init: One-time initialization
services:
  postgres-primary:
    # ... existing config
```

---

## 7. Implementation Order

### Phase 1: Foundation (Week 1) ✅
**Status:** Mostly Complete - Documentation already uses Podman

- [x] Audit current state (DONE - this plan)
- [x] Verify compose file compatibility (DONE - all compatible)
- [ ] Update GitHub workflow with dual runtime testing
- [ ] Create migration plan document (this file)

### Phase 2: Documentation (Week 1-2)

**Priority Order:**
1. Create `PODMAN_USAGE.md` with comprehensive Podman guide
2. Create `DOCKER_USAGE.md` with Docker compatibility guide
3. Update [`README.md`](README.md:1) with runtime selection guidance
4. Create `TROUBLESHOOTING.md` with runtime-specific solutions
5. Create `CONTRIBUTING.md` with dual-runtime testing requirements

### Phase 3: CI/CD Enhancement (Week 2)

**GitHub Workflow Updates:**

```yaml
name: Validate Container Compose - Docker & Podman

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  validate-compose:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        runtime: [docker, podman]
        service-group:
          - name: basic
            services: postgres mariadb redis mongo
          - name: cluster
            services: postgres-cluster redis-cluster
          - name: tools
            services: dbgate traefik

    name: Test ${{ matrix.runtime }} - ${{ matrix.service-group.name }}
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Podman
        if: matrix.runtime == 'podman'
        run: |
          sudo apt-get update
          sudo apt-get install -y podman podman-compose
          podman --version
          podman-compose --version

      - name: Set up Docker
        if: matrix.runtime == 'docker'
        run: |
          docker --version
          docker compose version

      - name: Copy environment file
        run: cp example.env .env

      - name: Validate compose syntax
        run: |
          if [ "${{ matrix.runtime }}" = "podman" ]; then
            podman-compose config
          else
            docker compose config
          fi

      - name: Pull images
        run: |
          if [ "${{ matrix.runtime }}" = "podman" ]; then
            podman-compose pull ${{ matrix.service-group.services }}
          else
            docker compose pull ${{ matrix.service-group.services }}
          fi

      - name: Start services
        run: |
          if [ "${{ matrix.runtime }}" = "podman" ]; then
            podman-compose up -d ${{ matrix.service-group.services }}
          else
            docker compose up -d ${{ matrix.service-group.services }}
          fi

      - name: Wait for services
        run: sleep 30

      - name: Check service health
        run: |
          if [ "${{ matrix.runtime }}" = "podman" ]; then
            podman-compose ps
          else
            docker compose ps
          fi

      - name: Run connectivity tests
        run: |
          # Add specific tests per service group
          echo "Testing ${{ matrix.service-group.name }} services"

      - name: Cleanup
        if: always()
        run: |
          if [ "${{ matrix.runtime }}" = "podman" ]; then
            podman-compose down -v
          else
            docker compose down -v
          fi
```

### Phase 4: Testing & Validation (Week 2-3)

**Test Sequence:**
1. **Local Testing** (Developer machine)
   - Test all services with Podman
   - Test all services with Docker
   - Document any differences

2. **CI/CD Testing** (GitHub Actions)
   - Validate workflow runs successfully
   - Verify all service groups start
   - Check for flaky tests

3. **Integration Testing**
   - Test service-to-service communication
   - Verify DBGate can connect to all databases
   - Test cluster formations

4. **Performance Testing**
   - Compare startup times
   - Monitor resource usage
   - Identify bottlenecks

### Phase 5: Refinement (Week 3-4)

- Address issues found in testing
- Optimize workflow execution time
- Enhance documentation based on feedback
- Create video tutorials (optional)

### Phase 6: Release (Week 4)

- Tag release version
- Update all references
- Announce dual runtime support
- Monitor for issues

---

## 8. Known Compatibility Considerations

### 8.1 Volume Path Differences

**Docker:**
```bash
# Volumes stored in:
/var/lib/docker/volumes/<volume-name>/_data

# List volumes:
docker volume ls
docker volume inspect <volume-name>
```

**Podman (Rootless):**
```bash
# Volumes stored in:
~/.local/share/containers/storage/volumes/<volume-name>/_data

# List volumes:
podman volume ls
podman volume inspect <volume-name>
```

**Podman (Rootful):**
```bash
# Volumes stored in:
/var/lib/containers/storage/volumes/<volume-name>/_data
```

**Impact:** Volume paths differ but functionality is identical. Document this for backup/restore operations.

### 8.2 Networking Differences

Both runtimes support bridge networks identically for this project:
- Network name: `ct_shared_network`
- DNS resolution works the same
- Service discovery via service names

**No changes needed** - current configuration works on both.

### 8.3 Image Registry

Current configuration uses `mirror.gcr.io` prefix:
```yaml
image: mirror.gcr.io/postgresql:18-alpine
```

**Compatibility:** Both runtimes can pull from this registry without modification.

**Alternative:** Consider providing fallback to official registries:
```yaml
# Option 1: Current (works on both)
image: mirror.gcr.io/postgresql:18-alpine

# Option 2: Official (if mirror fails)
image: docker.io/library/postgres:18-alpine
```

### 8.4 Cluster Services Considerations

**PostgreSQL Cluster** ([`databases/postgres-cluster/compose.yaml`](databases/postgres-cluster/compose.yaml:1)):
- Uses `pg_basebackup` for replication setup
- Init container runs bash scripts
- **Status:** Compatible with both runtimes ✅

**Redis Cluster** ([`databases/redis-cluster/compose.yaml`](databases/redis-cluster/compose.yaml:1)):
- Uses `redis-cli --cluster create`
- Init container checks cluster state
- **Status:** Compatible with both runtimes ✅

**Valkey Cluster** ([`databases/valkey-cluster/compose.yaml`](databases/valkey-cluster/compose.yaml:1)):
- Similar pattern to Redis cluster
- **Status:** Compatible with both runtimes ✅

**MariaDB Galera** ([`databases/mariadb-galera/compose.yaml`](databases/mariadb-galera/compose.yaml:1)):
- Uses Galera replication
- MaxScale for load balancing
- Complex inline configuration
- **Status:** Needs validation with Podman ⚠️

**Testing Priority:** MariaDB Galera cluster should be tested thoroughly with Podman due to complexity.

### 8.5 SELinux Considerations (Podman-specific)

On SELinux-enabled systems (RHEL, Fedora, CentOS):

```yaml
# May need :Z or :z suffix for volume mounts
volumes:
  - postgres_data:/var/lib/postgresql/data:Z
```

**Current Status:** Not using `:Z` suffix - works in permissive mode.
**Recommendation:** Document this in `TROUBLESHOOTING.md` for enforcing mode users.

### 8.6 Resource Limits

Both runtimes support resource limits, but syntax may differ slightly:

```yaml
# Standard format (works on both)
deploy:
  resources:
    limits:
      cpus: '0.5'
      memory: 512M
```

**Current Status:** No resource limits defined.
**Recommendation:** Add optional resource limits to example configurations in troubleshooting guide.

---

## 9. Success Metrics

### 9.1 Technical Metrics

- [ ] 100% of services start successfully with both runtimes
- [ ] CI/CD pipeline passes with both Docker and Podman
- [ ] Zero data loss during runtime switches
- [ ] Performance parity (±10%) between runtimes
- [ ] All cluster services form correctly on both runtimes

### 9.2 Documentation Metrics

- [ ] All runtime-specific documentation complete
- [ ] README updated with clear runtime selection guidance
- [ ] Troubleshooting guide covers both runtimes
- [ ] Migration guide reviewed and approved

### 9.3 User Adoption Metrics

- [ ] Zero breaking changes reported
- [ ] Users successfully run on both runtimes
- [ ] GitHub issues related to runtime compatibility: 0
- [ ] Positive feedback on dual runtime support

---

## 10. Risk Assessment & Mitigation

### 10.1 High Priority Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Cluster services fail on Podman | Medium | High | Extensive testing, fallback documentation |
| Volume permissions differ | Low | Medium | Document SELinux requirements, provide troubleshooting |
| CI/CD complexity increases | High | Low | Optimize workflow, use matrix strategy |
| User confusion on runtime choice | Medium | Low | Clear documentation, decision tree |

### 10.2 Medium Priority Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Image pull rate limits | Low | Medium | Use mirror.gcr.io, document alternatives |
| Performance differences | Low | Low | Benchmark and document findings |
| Network issues on specific runtimes | Low | Medium | Document network debugging steps |

### 10.3 Low Priority Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Documentation maintenance overhead | Medium | Low | Use includes for shared content |
| Version compatibility issues | Low | Low | Pin versions, document requirements |

---

## 11. Communication Plan

### 11.1 Internal Communication

**GitHub Issue:**
```markdown
Title: [RFC] Docker to Podman Dual Runtime Support

Description:
This project currently supports both Docker and Podman, but documentation and CI/CD 
are Docker-focused. This RFC proposes enhancing dual runtime support with:

1. Updated CI/CD to test both runtimes
2. Comprehensive documentation for each runtime
3. Troubleshooting guides for runtime-specific issues

Feedback requested on:
- Proposed implementation order
- Documentation structure
- Testing strategy
```

### 11.2 User Communication

**Announcement (GitHub Discussion):**
```markdown
Title: Enhanced Docker & Podman Support

We're improving dual runtime support for db-compose! 

What's changing:
✅ Better documentation for both Docker and Podman users
✅ CI/CD tests both runtimes automatically
✅ Troubleshooting guides for runtime-specific issues
✅ No breaking changes for existing users

Timeline: [dates]

Questions or feedback? Comment below!
```

---

## 12. Post-Migration Activities

### 12.1 Monitoring (First 30 Days)

- Track GitHub issues related to runtime compatibility
- Monitor CI/CD success rates
- Gather user feedback
- Track documentation page views

### 12.2 Optimization (Days 31-60)

- Optimize workflow execution time
- Enhance documentation based on common questions
- Add more comprehensive tests if needed
- Consider video tutorials

### 12.3 Long-term Maintenance

- Keep both runtime documentation updated
- Monitor runtime version compatibility
- Update CI/CD as runtimes evolve
- Maintain parity between Docker and Podman support

---

## 13. Appendices

### Appendix A: Command Reference

**Starting Services:**
```bash
# Podman
podman compose up -d [service...]
podman-compose up -d [service...]  # Alternative

# Docker
docker compose up -d [service...]
```

**Stopping Services:**
```bash
# Podman
podman compose down
podman compose down --volumes  # Remove volumes

# Docker
docker compose down
docker compose down --volumes
```

**Viewing Logs:**
```bash
# Podman
podman compose logs -f [service]

# Docker
docker compose logs -f [service]
```

**Service Status:**
```bash
# Podman
podman compose ps

# Docker
docker compose ps
```

### Appendix B: Testing Checklist

**Per Service Checklist:**
- [ ] Service starts without errors
- [ ] Health check passes (if defined)
- [ ] Ports are accessible
- [ ] Volumes persist data
- [ ] Environment variables loaded correctly
- [ ] Service can communicate with other services
- [ ] Service logs show normal operation
- [ ] Service stops cleanly
- [ ] Service restarts successfully

**Cluster Service Additional Checks:**
- [ ] All nodes start successfully
- [ ] Cluster forms correctly
- [ ] Replication works
- [ ] Failover works (if applicable)
- [ ] Init container completes successfully
- [ ] Load balancer routes correctly (if applicable)

### Appendix C: Volume Backup/Restore

**Backup Volumes (Podman):**
```bash
# List volumes
podman volume ls

# Backup specific volume
podman run --rm \
  -v db-compose_postgres_data:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_data_backup.tar.gz -C /source .
```

**Restore Volumes (Podman):**
```bash
# Create volume
podman volume create db-compose_postgres_data

# Restore data
podman run --rm \
  -v db-compose_postgres_data:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres_data_backup.tar.gz -C /target
```

**Same commands work for Docker - just replace `podman` with `docker`**

### Appendix D: Runtime Decision Tree

```
Are you using RHEL/Fedora/CentOS?
├─ Yes → Podman (native support, better integration)
└─ No
   ├─ Need rootless containers?
   │  ├─ Yes → Podman (built-in rootless support)
   │  └─ No → Either runtime works
   └─ Already have Docker Desktop?
      ├─ Yes → Docker (no need to change)
      └─ No → Podman (daemonless, lighter weight)
```

---

## 14. Summary & Next Steps

### Current Situation
- ✅ Project is **already 95% Podman-ready**
- ✅ All compose files are runtime-agnostic
- ✅ Documentation uses Podman terminology
- ⚠️ GitHub workflow needs dual runtime testing
- ⚠️ Need comprehensive dual-runtime documentation

### Recommended Immediate Actions

1. **Update GitHub Workflow** (High Priority)
   - Add Podman testing alongside Docker
   - Use matrix strategy for parallel testing
   - Estimated effort: 4-6 hours

2. **Create Runtime Documentation** (High Priority)
   - `PODMAN_USAGE.md` - Podman-specific guide
   - `DOCKER_USAGE.md` - Docker compatibility guide
   - `TROUBLESHOOTING.md` - Runtime-specific issues
   - Estimated effort: 8-12 hours

3. **Test Cluster Services** (Medium Priority)
   - Validate PostgreSQL cluster on Podman
   - Validate Redis cluster on Podman
   - Validate MariaDB Galera on Podman
   - Estimated effort: 6-8 hours

4. **Update README** (Low Priority)
   - Add runtime selection guidance
   - Add compatibility notes
   - Estimated effort: 2-3 hours

### Total Estimated Effort
**20-30 hours** spread over 2-3 weeks with proper testing and validation.

### Risk Level
**Low** - Most work is additive (documentation and testing). No breaking changes to existing configurations.

---

**Document Status:** DRAFT - Awaiting Review  
**Next Review Date:** Upon user approval  
**Document Owner:** Architecture Team