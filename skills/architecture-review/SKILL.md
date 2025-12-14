---
name: architecture-review
description: Use when reviewing system design and architecture decisions, evaluating scalability/reliability/maintainability, planning major technical initiatives, onboarding to complex systems, or identifying architectural technical debt
---

# Architecture Review

## Overview

Comprehensive architecture analysis providing principal-level recommendations on scalability, reliability, maintainability, and operational excellence.

## When to Use

- Architecture review, design review, or system analysis
- Planning major refactoring or migration initiatives
- Understanding system architecture for technical decisions
- Creating or reviewing architecture decision records (ADRs)
- Evaluating technical debt at the architectural level
- Onboarding to complex distributed systems

## Methodology

### Phase 1: Context and Discovery

**Objective**: Build comprehensive understanding of the system architecture.

```bash
# Identify system boundaries and entry points
rg "main\(|class.*Application|@SpringBootApplication|app\.|server\.|listen" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Find configuration files
fd -e yaml -e yml -e json -e toml -e conf "config|settings|application" --max-depth 3

# Identify database schemas and migrations
fd "schema|migration|models" --type d
fd -e sql -e prisma -e graphql "schema|migration"

# Find API definitions
fd -e yaml -e json "openapi|swagger|api-spec|graphql"

# Identify infrastructure as code
fd -e tf -e yaml "terraform|k8s|kubernetes|docker-compose|helm"
```

**Key Information to Gather**:
- System components and their responsibilities
- Data flow and communication patterns
- External dependencies and integrations
- Deployment architecture
- Data storage and persistence layer
- Authentication and authorization approach
- Observability and monitoring setup

### Phase 2: Architectural Pattern Analysis

**Identify Current Patterns**:

```bash
# Monolith vs Microservices
fd "service|api|server" --type d | wc -l  # Multiple services?
[ -f docker-compose.yml ] && rg "services:" docker-compose.yml  # Service count

# Layered architecture
fd "controller|service|repository|model|entity|dto" --type d

# Event-driven patterns
rg "EventEmitter|EventBus|MessageQueue|Kafka|RabbitMQ|SQS|PubSub" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# CQRS/Event Sourcing
rg "Command|Query|EventStore|EventSourcing|Aggregate" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# API patterns
rg "REST|GraphQL|gRPC|@.*Mapping|router\.|endpoint" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code
```

### Phase 3: Quality Attribute Evaluation

Evaluate against key quality attributes:

#### Scalability

```bash
# Horizontal scalability indicators
rg "stateless|session.*redis|jwt|token" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Connection pooling
rg "pool|connection.*pool|HikariCP" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Caching strategy
rg "cache|redis|memcached|Cache.*annotation" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Load balancing configuration
fd "nginx|haproxy|traefik|ingress" -e conf -e yaml
```

**Questions to Answer**:
- Can the system scale horizontally?
- Are there single points of failure?
- How is state managed?
- What are the scaling bottlenecks?

#### Reliability

```bash
# Error handling and resilience
rg "retry|circuit.*breaker|fallback|timeout|deadline" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Health checks
rg "health|readiness|liveness|/ping|/status" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Graceful degradation
rg "graceful.*shutdown|SIGTERM|signal.*handler" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Backup and recovery
fd "backup|restore|disaster" -e sh -e yaml
```

**Questions to Answer**:
- How does the system handle failures?
- Is there automatic retry and circuit breaking?
- Are there proper health checks?
- What is the disaster recovery strategy?

#### Maintainability

```bash
# Code organization and modularity
# Use omen if available:
# omen analyze tdg -f markdown

# Dependency management
fd "package.json|Cargo.toml|go.mod|pom.xml|requirements.txt|Gemfile"

# Documentation
fd README.md ARCHITECTURE.md CONTRIBUTING.md docs/

# Test coverage
rg "test|spec" --type-add 'test:*.{test.js,test.ts,spec.js,spec.rb,_test.go,test.py}' -t test | wc -l
```

**Questions to Answer**:
- Is the code well-organized and modular?
- Are dependencies up to date and managed?
- Is there adequate documentation?
- What is the test coverage?

#### Performance

```bash
# Performance considerations
rg "index|@Index|createIndex" --type-add 'code:*.{py,js,go,java,rb,rs,ts,sql}' -t code

# N+1 query problems
rg "select.*from.*loop|for.*query|map.*find" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Pagination
rg "limit|offset|cursor|page.*size" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Async/concurrent processing
rg "async|await|goroutine|thread|concurrent|parallel" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code
```

#### Security

```bash
# Security architecture
rg "auth|jwt|oauth|saml|password|encrypt|hash" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# API security
rg "rate.*limit|throttle|cors|csrf|xss" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Secrets management
rg "secret|credential|api.*key|password" --type-add 'config:*.{yaml,yml,json,env}' -t config
fd ".env" -H
```

### Phase 4: Anti-Pattern Detection

**Common Architectural Anti-Patterns**:

**God Object/Service**:
```bash
# Files with excessive lines of code
fd -e py -e js -e go -e java -e rb -e rs -e ts --exec wc -l {} \; | sort -rn | head -20

# Classes/modules with too many responsibilities
rg "class.*\{" -A 500 | rg "public|def |func " | head -50
```

**Circular Dependencies**:
```bash
# Check for import cycles
go mod graph 2>/dev/null | grep cycle || true
```

**Tight Coupling**:
```bash
# High coupling indicators
rg "import.*\.\." --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code | wc -l
```

**Distributed Monolith**:
```bash
# Microservices with tight coupling
[ -f docker-compose.yml ] && rg "depends_on" docker-compose.yml
rg "http.*localhost|http.*service\." --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code
```

**Database as Integration**:
```bash
# Multiple services accessing same database
rg "database|connection.*string" docker-compose.yml 2>/dev/null || true
```

**Missing Observability**:
```bash
# Check for logging, metrics, tracing
rg "logger|log\.|metrics|trace|span" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code | wc -l
[ -d monitoring ] || [ -f prometheus.yml ] || [ -f grafana.json ] || echo "No observability configuration found"
```

### Phase 5: Data Architecture Review

```bash
# Database schema files
fd schema.sql schema.prisma models.py entity.java

# Migration files
fd -e sql migration | head -10

# Data models
fd "model|entity|schema" --type d
rg "class.*Model|@Entity|model\s+\w+|type.*struct" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code
```

**Evaluate**:
- Database choice and justification
- Schema design and normalization
- Indexing strategy
- Data consistency approach (ACID vs BASE)
- Data migration strategy
- Backup and recovery

### Phase 6: Integration Architecture

```bash
# External integrations
rg "http\.|fetch|axios|requests\.|HttpClient|curl" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Message queues and event systems
rg "kafka|rabbitmq|sqs|sns|pubsub|eventbridge" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code

# Third-party services
rg "stripe|twilio|sendgrid|aws|gcp|azure" --type-add 'code:*.{py,js,go,java,rb,rs,ts}' -t code
```

**Evaluate**:
- Integration patterns (sync vs async)
- Error handling and retry logic
- Rate limiting and backpressure
- API versioning strategy
- Circuit breakers and fallbacks

## Architecture Review Report Format

```markdown
# Architecture Review Report

**System**: <system_name>
**Reviewer**: Architecture Review Skill
**Date**: <date>
**Scope**: <full_system|specific_components>

---

## Executive Summary

[2-3 paragraphs summarizing architecture, strengths, key concerns, and top recommendations]

**Architecture Style**: <Monolith|Microservices|Modular Monolith|Event-Driven|Serverless|Hybrid>

**Overall Assessment**: <STRONG|GOOD|NEEDS IMPROVEMENT|CRITICAL ISSUES>

**Key Metrics**:
- Components: X
- Languages: X primary, X secondary
- External Dependencies: X
- Database(s): X
- Lines of Code: X

---

## System Overview

### Architecture Diagram

[ASCII diagram or description of main components and their relationships]

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Client    │─────▶│     API     │─────▶│  Database   │
│  (Web/App)  │      │   Gateway   │      │  (Postgres) │
└─────────────┘      └─────────────┘      └─────────────┘
                            │
                            ▼
                     ┌─────────────┐
                     │   Service   │
                     │    Layer    │
                     └─────────────┘
```

### Component Breakdown

**Core Components**:
1. **Component Name** (`path/to/component`)
   - Responsibility: [what it does]
   - Technology: [language/framework]
   - Complexity: [HIGH|MEDIUM|LOW]
   - LOC: X

[Repeat for each major component]

### Technology Stack

**Languages**: Go 80%, JavaScript 15%, Python 5%
**Frameworks**: [list]
**Databases**: PostgreSQL (primary), Redis (cache)
**Infrastructure**: Kubernetes, AWS
**Key Dependencies**: [list critical dependencies]

---

## Architectural Patterns

### Identified Patterns

**Applied Patterns**:
- Layered Architecture (Controller -> Service -> Repository)
- Repository Pattern for data access
- Dependency Injection
- API Gateway pattern
- Caching strategy (Redis)

**Missing Patterns**:
- Circuit Breaker pattern (recommend adding)
- Event Sourcing (could benefit for audit trail)
- CQRS (for read/write optimization)

### Anti-Patterns Found

#### CRITICAL: God Service

**Location**: `services/user_service.go:1-2500`
**Issue**: User service handles authentication, authorization, profile management, notifications, and audit logging
**Impact**: High coupling, difficult to test, scaling bottleneck
**Recommendation**: Split into:
- `auth-service` (authentication/authorization)
- `profile-service` (user profile management)
- `notification-service` (notifications)
- Extract audit logging to cross-cutting concern

#### HIGH: Database as Integration Point

**Location**: Multiple services accessing `users` table
**Issue**: Services A, B, and C all directly query the users table
**Impact**: Tight coupling, schema changes affect multiple services
**Recommendation**: Introduce user service as single source of truth, expose via API

[Continue for each anti-pattern]

---

## Quality Attribute Assessment

### Scalability: NEEDS IMPROVEMENT

**Current State**:
- [x] Stateless application servers (can scale horizontally)
- [x] Database connection pooling configured
- [x] Redis caching layer
- [ ] No sharding strategy for database
- [ ] File uploads stored on local disk (not cloud storage)
- [ ] Background jobs run in-process (scaling constraint)

**Bottlenecks**:
1. Database becomes bottleneck at >10K concurrent users
2. File storage not horizontally scalable
3. Background job processing limits horizontal scaling

**Recommendations**:
1. Implement database read replicas for read-heavy workloads
2. Move file storage to S3/object storage
3. Extract background jobs to separate worker service with queue
4. Consider database sharding strategy for >100K users

### Reliability: GOOD

**Current State**:
- [x] Health check endpoints implemented
- [x] Graceful shutdown handling
- [x] Database migrations with rollback support
- [x] Kubernetes liveness/readiness probes
- [ ] No circuit breaker for external API calls
- [ ] Missing retry logic with exponential backoff

**Failure Scenarios**:
1. External payment API failure: No fallback, user sees error
2. Database temporary unavailability: Application crashes
3. Redis cache failure: Performance degradation but functional

**Recommendations**:
1. Implement circuit breaker (hystrix, resilience4j, or similar)
2. Add retry with exponential backoff for transient failures
3. Implement graceful degradation when cache unavailable
4. Document runbooks for common failure scenarios

### Maintainability: STRONG

**Current State**:
- [x] Clear module boundaries
- [x] Comprehensive test coverage (78%)
- [x] Up-to-date documentation
- [x] Consistent code style (enforced by linters)
- [x] Dependency versions managed
- [x] CI/CD pipeline with automated tests

**Code Quality**:
- Average complexity: 15 (GOOD)
- High complexity files: 3 (flagged for refactoring)
- Technical debt ratio: 12% (ACCEPTABLE)

**Recommendations**:
1. Address high-complexity files identified in quality report
2. Add architecture decision records (ADRs) for major decisions
3. Consider dependency update automation (Dependabot, Renovate)

### Performance: GOOD

**Current State**:
- [x] Database indexes on key columns
- [x] API response caching
- [x] Connection pooling
- [x] Async processing for heavy operations
- [ ] Potential N+1 queries in some endpoints
- [ ] No query result pagination on some list endpoints

**Performance Hotspots**:
1. `GET /api/users` - No pagination, fetches all users
2. `services/order_processor.go:124` - N+1 query on order items
3. `handlers/reports.go:89` - Expensive aggregation query

**Recommendations**:
1. Add pagination to all list endpoints (cursor-based preferred)
2. Use ORM eager loading to fix N+1 queries
3. Consider materialized views or pre-computed reports
4. Add database query performance monitoring (e.g., pg_stat_statements)

### Security: NEEDS IMPROVEMENT

**Current State**:
- [x] JWT authentication
- [x] Password hashing (bcrypt)
- [x] HTTPS enforced
- [x] Input validation
- [ ] No rate limiting on API endpoints
- [ ] Secrets in configuration files (not using secrets manager)
- [ ] Missing CORS configuration
- [ ] No API authentication for internal service-to-service calls

**Security Risks**:
1. **CRITICAL**: API keys in `.env` files committed to repository
2. **HIGH**: No rate limiting - vulnerable to DoS
3. **HIGH**: Missing mTLS for service-to-service communication
4. **MEDIUM**: No audit logging for sensitive operations

**Recommendations**:
1. Migrate secrets to HashiCorp Vault or AWS Secrets Manager
2. Implement rate limiting (Redis-based token bucket)
3. Add mTLS for internal service communication
4. Implement comprehensive audit logging
5. Regular security scanning (already using semgrep - good)

---

## Data Architecture

### Current State

**Databases**:
- PostgreSQL 14 (primary data store)
- Redis 6 (caching, session storage)

**Schema Design**: 3NF normalized, well-designed
**Migrations**: Flyway, versioned, rollback support
**Backup Strategy**: Daily automated backups, 30-day retention

### Concerns

1. **No Read/Write Splitting**: All queries go to primary database
2. **Growing Data Volume**: Orders table at 50M rows, query performance degrading
3. **Missing Data Archival**: No strategy for archiving old data

### Recommendations

1. Implement read replicas for read-heavy workloads
2. Partition orders table by date (monthly partitions)
3. Archive orders older than 2 years to cold storage
4. Consider CQRS for reporting workloads

---

## Integration Architecture

### External Dependencies

| Service | Protocol | Purpose | Criticality | Resilience |
|---------|----------|---------|-------------|------------|
| Stripe API | REST/HTTPS | Payment processing | CRITICAL | No circuit breaker |
| SendGrid | REST/HTTPS | Email delivery | HIGH | No retry logic |
| Auth0 | REST/HTTPS | SSO authentication | CRITICAL | Has fallback |
| S3 | S3 API | File storage | MEDIUM | SDK handles retry |

### Concerns

1. No circuit breaker for Stripe - payment failures block user flow
2. Synchronous email sending - slow email provider delays API responses
3. No monitoring of external service health

### Recommendations

1. Implement circuit breaker for all external HTTP calls
2. Move email sending to async queue (SQS, Redis Queue)
3. Add external dependency health monitoring dashboard
4. Implement fallback mechanisms for critical paths

---

## Deployment Architecture

### Current State

**Platform**: AWS EKS (Kubernetes)
**Deployment Strategy**: Rolling updates
**Environments**: dev, staging, production
**CI/CD**: GitHub Actions -> ECR -> EKS

### Infrastructure as Code

- [x] Terraform for AWS resources
- [x] Helm charts for Kubernetes
- [x] GitOps workflow (ArgoCD)

### Concerns

1. No blue/green or canary deployment strategy
2. Database migrations run in application startup (risk during rollback)
3. No automatic rollback on deployment failure

### Recommendations

1. Implement canary deployments with automated rollback
2. Separate database migration job from application deployment
3. Add deployment health checks and automatic rollback triggers
4. Consider feature flags for safer releases

---

## Observability

### Current State

**Logging**: Structured logging to CloudWatch
**Metrics**: Prometheus + Grafana
**Tracing**: Not implemented
**Alerting**: Basic CPU/memory alerts

### Gaps

1. No distributed tracing (difficult to debug across services)
2. Limited business metrics (no funnel tracking)
3. No SLO/SLI definitions
4. Alert fatigue (too many low-priority alerts)

### Recommendations

1. Implement distributed tracing (Jaeger, OpenTelemetry)
2. Define and track SLIs/SLOs for critical user journeys
3. Add business metrics dashboards (conversion rates, revenue, etc.)
4. Review and tune alert thresholds (reduce noise)
5. Implement proper incident response runbooks

---

## Technical Debt

### High Priority

1. **Monolithic User Service** (Complexity: 342)
   - Effort: 3-4 weeks
   - Impact: Enables independent scaling, improves maintainability
   - Risk: Medium (requires careful data migration)

2. **Missing Circuit Breakers**
   - Effort: 1 week
   - Impact: Prevents cascade failures
   - Risk: Low

3. **Secrets in Configuration**
   - Effort: 1 week
   - Impact: Critical security improvement
   - Risk: Low (well-documented process)

### Medium Priority

4. **N+1 Query Issues** (5 identified locations)
5. **Missing Pagination** (8 endpoints)
6. **Test Coverage Gaps** (22% not covered)

### Low Priority

7. **Documentation Updates**
8. **Dependency Upgrades**

---

## Recommendations Roadmap

### Immediate (0-2 weeks)

1. **Migrate Secrets to Secrets Manager** - CRITICAL
   - Remove secrets from `.env` files
   - Use AWS Secrets Manager or HashiCorp Vault
   - Update deployment pipeline

2. **Implement Rate Limiting** - HIGH
   - Prevent DoS attacks
   - Use Redis-based token bucket
   - Apply to public API endpoints

3. **Add Circuit Breakers** - HIGH
   - Implement for Stripe, SendGrid, Auth0 calls
   - Use library like resilience4j (Java) or hystrix
   - Configure sensible defaults (50% failure rate, 10s timeout)

### Short-term (1-3 months)

4. **Split User Service**
   - Break into auth-service, profile-service, notification-service
   - Define clear API boundaries
   - Implement database-per-service pattern

5. **Implement Distributed Tracing**
   - Add OpenTelemetry instrumentation
   - Deploy Jaeger for trace visualization
   - Create traces for critical user flows

6. **Database Read Replicas**
   - Set up PostgreSQL read replicas
   - Route read queries to replicas
   - Implement read/write splitting in application

### Long-term (3-6 months)

7. **CQRS for Reporting**
   - Separate read and write models
   - Build optimized read stores for reports
   - Implement event-driven synchronization

8. **Observability Improvements**
   - Define SLIs/SLOs for critical paths
   - Build business metrics dashboards
   - Implement comprehensive alerting strategy

9. **Data Archival Strategy**
   - Archive old orders (>2 years)
   - Implement data lifecycle management
   - Reduce primary database size

---

## Architectural Decision Records (ADRs)

Recommend creating ADRs for the following decisions:

1. **ADR-001**: Choice of Monolith vs Microservices
2. **ADR-002**: Database Technology Selection (PostgreSQL)
3. **ADR-003**: Caching Strategy (Redis)
4. **ADR-004**: Authentication Approach (JWT + Auth0)
5. **ADR-005**: Deployment Platform (EKS)

Template for ADRs provided in appendix.

---

## Conclusion

The system demonstrates solid engineering practices with good maintainability and operational maturity. Key strengths include clear module boundaries, comprehensive testing, and modern CI/CD practices.

Primary concerns center around scalability constraints (monolithic services, database scaling) and security gaps (secrets management, rate limiting). The system is production-ready but will encounter scaling challenges beyond current load.

Recommended focus areas:
1. Security hardening (secrets, rate limiting)
2. Reliability improvements (circuit breakers, retry logic)
3. Scalability preparation (service decomposition, database scaling)

---

## Appendices

### A. Architecture Decision Record Template

```markdown
# ADR-NNN: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
[Describe the problem and why a decision is needed]

## Decision
[Describe the decision and rationale]

## Consequences
[Positive and negative consequences of the decision]

## Alternatives Considered
[Other options that were evaluated]
```

### B. References
- [List relevant documentation, RFCs, design docs]

---

**Report Generated**: <timestamp>
```

## Key Evaluation Criteria

When reviewing architecture, systematically evaluate:

1. **Single Responsibility** - Are components focused on one concern? Can you explain each component's purpose in one sentence?
2. **Separation of Concerns** - Are business logic, data access, and presentation separated? Can layers be changed independently?
3. **Dependency Direction** - Do dependencies point inward (toward business logic)? Are there circular dependencies?
4. **Abstraction Levels** - Are abstractions at appropriate levels? Is there leaky abstraction?
5. **Coupling and Cohesion** - Low coupling between modules? High cohesion within modules?
6. **Testability** - Can components be tested in isolation? Are external dependencies mockable?
7. **Deployability** - Can components be deployed independently? What is the deployment risk?
8. **Observability** - Can you debug production issues? Are critical paths instrumented?
9. **Security by Design** - Is security a first-class concern? Are security boundaries clear?
10. **Cost Efficiency** - Is the architecture cost-effective? Are resources used efficiently?

## Common Architecture Smells

- **Big Ball of Mud**: Everything depends on everything
- **Analysis Paralysis**: Over-engineering for hypothetical future
- **Resume-Driven Development**: Technology choices for resume building
- **Not Invented Here**: Rejecting proven solutions
- **Golden Hammer**: Applying one pattern to every problem
- **Premature Optimization**: Optimizing before measuring
- **Premature Distribution**: Microservices without necessity

## Best Practices

1. **Start Simple**: Begin with simplest architecture that works
2. **Evolve Gradually**: Refactor toward patterns as needs emerge
3. **Measure First**: Use data to drive architectural decisions
4. **Document Decisions**: Use ADRs for significant choices
5. **Consider Trade-offs**: Every decision has costs and benefits
6. **Think in Layers**: Separate concerns, define boundaries
7. **Plan for Failure**: Design for resilience from start
8. **Optimize for Change**: Make easy-to-change things easy to change

---

**Remember**: Good architecture enables rapid, safe feature development. The best architecture is the simplest one that meets current needs while remaining adaptable to future change.
