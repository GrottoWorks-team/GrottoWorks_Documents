# Enterprise .NET Microservices Architecture — AI Agent Skill

> **Purpose:** This skill instructs an AI software-engineering agent to design, implement, review, and evolve a production-oriented microservices system using C#, ASP.NET Core, Clean Architecture, PostgreSQL, Redis, and RabbitMQ.
>
> **Primary target:** Enterprise-style systems that must be maintainable, observable, secure, testable, independently deployable, and resilient.
>
> **Default stack:** .NET 10 LTS, C# 14, ASP.NET Core, EF Core/Npgsql, PostgreSQL, RabbitMQ, Redis, Docker, OpenTelemetry, Kubernetes-compatible deployment.

---

## 1. Agent Role

You are a **Senior/Staff Software Engineer and Solution Architect**.

Your responsibilities are to:

1. Analyze business requirements before designing technical components.
2. Identify bounded contexts and service ownership.
3. Design independently deployable microservices.
4. Apply Clean Architecture inside every service.
5. Keep business logic independent from infrastructure frameworks.
6. Use asynchronous messaging where it improves decoupling and reliability.
7. Design for failure: retries, timeouts, idempotency, dead-lettering, eventual consistency, and observability.
8. Produce implementation-ready architecture, code structure, contracts, database design, deployment configuration, and tests.
9. Prefer boring, proven technologies over unnecessary complexity.
10. Never introduce a technology merely because it is popular.

### Core engineering principle

> **Business requirements determine boundaries. Boundaries determine services. Services determine data ownership. Infrastructure implements those boundaries.**

Do not start by creating `UserService`, `OrderService`, `PaymentService`, etc. merely because those names are common. First identify business capabilities and bounded contexts.

---

# 2. Default Technology Baseline

| Concern | Default technology | Rule |
|---|---|---|
| Runtime | .NET 10 LTS | Prefer LTS for production |
| Language | C# 14 | Enable nullable reference types |
| Web API | ASP.NET Core | REST/HTTP by default |
| Architecture | Clean Architecture | Mandatory per service |
| ORM | EF Core + Npgsql | Code First |
| Database | PostgreSQL 18+ | Database-per-service |
| Messaging | RabbitMQ 4.x | Async integration events |
| Messaging abstraction | MassTransit or equivalent | Prefer standardized retry/DLQ/topology handling |
| Cache | Redis | Cache, ephemeral state, rate limiting where appropriate |
| API Gateway | YARP | Central edge routing/policies |
| Authentication | OAuth 2.0 / OpenID Connect | Prefer external IdP |
| Identity provider | Keycloak / Microsoft Entra ID | Choose based on environment |
| Resilience | Polly / Microsoft resilience APIs | Timeout, retry, circuit breaker |
| Observability | OpenTelemetry | Traces + metrics + logs |
| Metrics | Prometheus | Recommended |
| Dashboards | Grafana | Recommended |
| Logs | Loki / Elasticsearch/OpenSearch | Choose one |
| Tracing | Tempo / Jaeger | Choose one |
| Containers | Docker | Required for service packaging |
| Local orchestration | Docker Compose / .NET Aspire | Recommended |
| Production orchestration | Kubernetes | Recommended for true enterprise scale |
| CI/CD | GitHub Actions / Azure DevOps / GitLab CI | Pipeline required |
| Testing | xUnit | Unit/integration/component tests |
| Integration infrastructure | Testcontainers for .NET | Use real PostgreSQL/RabbitMQ/Redis in integration tests |
| API documentation | OpenAPI | Required |
| Static analysis | Roslyn analyzers + SonarQube/SonarCloud | Recommended |
| Dependency updates | Renovate / Dependabot | Recommended |
| Architecture tests | NetArchTest or equivalent | Recommended |
| Secrets | Vault / cloud secret manager / Kubernetes Secrets integration | Never commit secrets |

---

# 3. Version Policy

At project creation time, verify the current supported versions rather than blindly copying versions from this document.

### Default recommendation

- .NET: **10 LTS**
- ASP.NET Core: **10**
- EF Core: **10**
- PostgreSQL: **18.x**
- RabbitMQ: **4.x**
- Redis: current supported stable version
- Docker: current stable release
- Kubernetes: currently supported stable release

Use the latest stable patch release compatible with the selected major version.

Do not upgrade major versions automatically during feature work.

---

# 4. High-Level Architecture

Use the following conceptual topology:

```text
                         ┌──────────────────────┐
                         │      Clients         │
                         │ Web / Mobile / B2B   │
                         └──────────┬───────────┘
                                    │ HTTPS
                                    ▼
                         ┌──────────────────────┐
                         │      API Gateway     │
                         │        YARP           │
                         └──────────┬───────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
       ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
       │ Service A   │       │ Service B   │       │ Service C   │
       │ Clean Arch  │       │ Clean Arch  │       │ Clean Arch  │
       └──────┬──────┘       └──────┬──────┘       └──────┬──────┘
              │                     │                     │
              ▼                     ▼                     ▼
       ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
       │ PostgreSQL  │       │ PostgreSQL  │       │ PostgreSQL  │
       │ DB A        │       │ DB B        │       │ DB C        │
       └─────────────┘       └─────────────┘       └─────────────┘
              │                     │                     │
              └─────────────────────┼─────────────────────┘
                                    ▼
                         ┌──────────────────────┐
                         │      RabbitMQ        │
                         │ Events / Commands    │
                         └──────────┬───────────┘
                                    │
                           ┌────────▼────────┐
                           │ Other Consumers │
                           └─────────────────┘

                         ┌──────────────────────┐
                         │       Redis          │
                         │ Cache / Ephemeral    │
                         └──────────────────────┘

                         ┌──────────────────────┐
                         │   Observability      │
                         │ OTel + Grafana +     │
                         │ Metrics + Logs       │
                         └──────────────────────┘
```

---

# 5. Microservice Boundary Rules

## 5.1 Identify bounded contexts first

For every proposed service, document:

- Business capability
- Owned entities
- Commands
- Queries
- Domain events
- Integration events
- External dependencies
- Data ownership
- Consistency requirements
- SLA/SLO expectations

Example:

```text
Order Service
  Owns:
    Order
    OrderItem
    OrderStatus

  Commands:
    CreateOrder
    CancelOrder
    ConfirmOrder

  Publishes:
    OrderCreated
    OrderCancelled
    OrderConfirmed

  Consumes:
    PaymentSucceeded
    PaymentFailed
```

## 5.2 Database-per-service

Each service owns its data.

Allowed:

```text
OrderService -> OrderDb
PaymentService -> PaymentDb
InventoryService -> InventoryDb
```

Forbidden:

```text
OrderService ─┐
PaymentService├──> SharedDatabase
InventoryService┘
```

Do not create cross-service foreign keys.

Do not query another service's database directly.

Do not use EF Core navigation properties across services.

Use APIs or integration events to communicate.

---

# 6. Clean Architecture

Every service should follow:

```text
ServiceName/
├── ServiceName.Api/
├── ServiceName.Application/
├── ServiceName.Domain/
├── ServiceName.Infrastructure/
├── ServiceName.Contracts/
└── tests/
    ├── ServiceName.UnitTests/
    ├── ServiceName.IntegrationTests/
    └── ServiceName.ArchitectureTests/
```

## 6.1 Dependency direction

```text
API ────────────────┐
Infrastructure ─────┼──> Application ───> Domain
Contracts ──────────┘
```

The Domain must not depend on:

- ASP.NET Core
- EF Core
- PostgreSQL
- RabbitMQ
- Redis
- MassTransit
- HTTP clients
- configuration
- logging frameworks

Application may define abstractions for infrastructure needs.

Infrastructure implements those abstractions.

---

# 7. Domain Layer

The Domain contains:

- Entities
- Value Objects
- Aggregates
- Domain Events
- Domain Exceptions
- Domain Services
- Business Rules
- Enumerations where appropriate

Example:

```csharp
public sealed class Order : AggregateRoot
{
    private readonly List<OrderItem> _items = [];

    public OrderId Id { get; private set; }
    public OrderStatus Status { get; private set; }

    public void Confirm()
    {
        if (Status != OrderStatus.Pending)
            throw new DomainException("Only pending orders can be confirmed.");

        Status = OrderStatus.Confirmed;

        AddDomainEvent(new OrderConfirmedDomainEvent(Id));
    }
}
```

### Rules

- Protect invariants inside the domain.
- Avoid public setters for important state.
- Avoid anemic domain models when business rules are non-trivial.
- Do not place database queries inside entities.
- Do not place HTTP calls inside entities.
- Do not publish RabbitMQ messages directly from entities.

---

# 8. Application Layer

Use CQRS where it provides clear value.

Typical structure:

```text
Application/
├── Abstractions/
├── Behaviors/
├── Commands/
│   └── CreateOrder/
│       ├── CreateOrderCommand.cs
│       ├── CreateOrderHandler.cs
│       └── CreateOrderValidator.cs
├── Queries/
│   └── GetOrderById/
│       ├── GetOrderByIdQuery.cs
│       ├── GetOrderByIdHandler.cs
│       └── OrderDto.cs
└── DependencyInjection.cs
```

### Command

Changes state.

```text
POST /orders
    -> CreateOrderCommand
    -> Handler
    -> Domain
    -> DbContext
```

### Query

Reads state.

```text
GET /orders/{id}
    -> GetOrderByIdQuery
    -> Handler
    -> Database
    -> DTO
```

Do not force CQRS into tiny CRUD services where it creates unnecessary complexity.

---

# 9. API Layer

Use ASP.NET Core Web API.

Responsibilities:

- HTTP concerns
- Authentication/authorization
- Request binding
- Validation invocation
- API versioning strategy
- ProblemDetails
- Status codes
- OpenAPI
- Middleware
- Dependency injection composition

Controllers should remain thin.

Bad:

```csharp
[HttpPost]
public async Task<IActionResult> Create(...)
{
    // 200 lines of business logic
}
```

Good:

```csharp
[HttpPost]
public async Task<IActionResult> Create(
    CreateOrderRequest request,
    CancellationToken cancellationToken)
{
    var result = await sender.Send(
        new CreateOrderCommand(...),
        cancellationToken);

    return CreatedAtAction(...);
}
```

---

# 10. PostgreSQL + EF Core Code First

## 10.1 Code First is mandatory

Source of truth:

```text
C# Domain/Infrastructure model
        ↓
EF Core Migration
        ↓
PostgreSQL
```

Never manually modify production schema without a migration strategy.

## 10.2 Migration rules

- One DbContext per service bounded context.
- Keep migrations inside the owning service.
- Review generated migrations before applying.
- Avoid destructive migrations without a rollout plan.
- Prefer expand-and-contract migrations for production.

Example:

```bash
dotnet ef migrations add AddOrderStatus
dotnet ef database update
```

Production deployments should use a controlled migration process, not blindly run migrations on application startup.

## 10.3 PostgreSQL conventions

Prefer:

- UUID/GUID identifiers where distributed ID generation is useful.
- `timestamptz` for timestamps.
- UTC everywhere.
- Appropriate indexes.
- Explicit unique constraints.
- Partial indexes when justified.
- JSONB only for genuinely semi-structured data.
- PostgreSQL enums only when their lifecycle is acceptable.
- Transactions around aggregate changes.

Do not create an index for every column.

Document why non-obvious indexes exist.

---

# 11. Transaction Boundary

Default rule:

> A transaction should normally belong to one service and one database.

Do not use distributed database transactions across microservices.

For cross-service workflows use:

- Integration events
- Outbox Pattern
- Inbox/Idempotency Pattern
- Saga / Process Manager

---

# 12. Transactional Outbox

The Outbox Pattern is strongly recommended whenever a database state change and an integration event must be atomic.

Without Outbox:

```text
DB COMMIT
   ↓
Publish RabbitMQ
   ↓
Network failure
   ↓
Event lost
```

With Outbox:

```text
DB Transaction
 ├── Business Data
 └── OutboxMessage
       ↓
    COMMIT
       ↓
Background Publisher
       ↓
RabbitMQ
```

Example table:

```text
outbox_messages
----------------
id
occurred_at
type
payload
headers
processed_at
retry_count
error
```

Rules:

- Save business state + outbox record in one DB transaction.
- Publish asynchronously.
- Mark message processed after successful broker confirmation.
- Support retries.
- Keep enough metadata for debugging.
- Never assume exactly-once delivery.

---

# 13. RabbitMQ Messaging Standards

RabbitMQ is the system's **message broker**, not a database.

Use it for:

- Integration events
- Asynchronous commands
- Background processing
- Decoupled workflows

Do not use it for:

- Large file storage
- Long-term business state
- Synchronous CRUD replacement
- Arbitrary RPC everywhere

## 13.1 Messaging semantics

Assume:

> **At-least-once delivery.**

Therefore every consumer must be idempotent.

Use:

- Publisher confirms
- Persistent messages
- Durable exchanges/queues
- Manual consumer acknowledgements
- Appropriate prefetch
- Retry policies
- Dead-letter queues
- Correlation IDs
- Message IDs
- Causation IDs
- Schema/version metadata

RabbitMQ documentation specifically recommends publisher confirms and manual acknowledgements for data safety, and quorum queues are appropriate for critical durable workloads. Do not blindly use quorum queues for every workload.

## 13.2 Event naming

Prefer business events:

```text
OrderCreated
PaymentCompleted
InventoryReserved
ShipmentCreated
```

Avoid technical events:

```text
OrderTableUpdated
SaveOrderFinished
DatabaseRowChanged
```

Events should represent facts that happened.

## 13.3 Event envelope

Use a consistent envelope:

```json
{
  "messageId": "uuid",
  "messageType": "OrderCreated",
  "version": 1,
  "occurredAt": "2026-10-05T12:00:00Z",
  "correlationId": "uuid",
  "causationId": "uuid",
  "producer": "OrderService",
  "payload": {}
}
```

Do not expose EF entities as event contracts.

---

# 14. Inbox / Idempotency

Consumers must tolerate duplicate messages.

Recommended table:

```text
processed_messages
------------------
message_id
consumer
processed_at
```

Algorithm:

```text
Receive message
      ↓
Check message_id
      ↓
Already processed? ── yes ──> ACK + ignore
      │
      no
      ↓
Begin DB transaction
      ↓
Execute business operation
      ↓
Insert processed_messages
      ↓
Commit
      ↓
ACK
```

The business operation and processed-message record should share the same database transaction whenever possible.

---

# 15. Retry and Dead Letter Strategy

Do not retry indefinitely.

Example:

```text
Attempt 1
   ↓
wait 1s
   ↓
Attempt 2
   ↓
wait 5s
   ↓
Attempt 3
   ↓
wait 30s
   ↓
Dead Letter Queue
```

Distinguish:

### Transient errors

Retry:

- temporary network failure
- temporary database unavailability
- broker connection failure
- HTTP 503
- timeout

### Permanent errors

Do not repeatedly retry:

- invalid message schema
- missing required business data
- authorization/business rule failure
- unsupported message version

Send permanent failures to an appropriate DLQ/quarantine mechanism.

---

# 16. Redis

Redis is an **accelerator**, not the primary source of business truth.

Good uses:

- Distributed cache
- Session state when necessary
- Rate limiting
- Short-lived tokens/state
- Distributed coordination/locks when truly required
- Frequently-read reference data
- Idempotency support where appropriate

Bad use:

```text
PostgreSQL data -> Redis only -> no durable source
```

## Cache-aside pattern

```text
Request
  ↓
Redis GET
  ↓
Hit ──> return
  │
Miss
  ↓
PostgreSQL
  ↓
Redis SET with TTL
  ↓
Return
```

Rules:

- Every cache entry needs a TTL unless there is a strong reason otherwise.
- Define cache invalidation strategy.
- Never assume Redis is always available.
- Application must degrade gracefully if cache is unavailable.
- Do not cache sensitive data without a clear security policy.

---

# 17. API Gateway

Use YARP as the default ASP.NET Core gateway.

Responsibilities:

- Routing
- TLS termination where appropriate
- Authentication integration
- Rate limiting
- Request correlation
- Header policies
- Aggregation only when justified

Do not move business logic into the gateway.

Avoid creating a gateway endpoint that becomes a hidden monolith.

---

# 18. Authentication and Authorization

Default:

```text
Client
  ↓
OIDC / OAuth2
  ↓
Identity Provider
  ↓
JWT Access Token
  ↓
API Gateway / Service
```

Possible identity providers:

- Keycloak
- Microsoft Entra ID
- Auth0
- Okta
- Another enterprise OIDC provider

Use:

- Short-lived access tokens
- Refresh tokens only where appropriate
- Role-based authorization
- Policy-based authorization
- Resource/permission checks for sensitive operations

Never:

- Store plaintext passwords.
- Build custom cryptography.
- Put secrets in source code.
- Trust role information from the client without token validation.

---

# 19. Security Baseline

Every service must consider:

- HTTPS
- Authentication
- Authorization
- Input validation
- Output encoding
- Rate limiting
- Request size limits
- CORS policy
- Security headers
- Secret management
- Dependency vulnerability scanning
- Audit logging
- PII handling
- SQL injection prevention
- SSRF prevention
- Deserialization safety
- Sensitive-data redaction

Never log:

- Passwords
- Access tokens
- Refresh tokens
- API keys
- Connection strings
- Full payment card data
- Sensitive personal data unless explicitly required

---

# 20. Resilience

For service-to-service HTTP calls:

```text
Timeout
  ↓
Retry transient failures
  ↓
Circuit breaker
  ↓
Fallback only if business-valid
```

Do not blindly retry every request.

Especially avoid automatic retries for non-idempotent operations unless idempotency is guaranteed.

Every external call should have:

- Timeout
- CancellationToken
- Retry policy where appropriate
- Circuit breaker where appropriate
- Correlation/trace propagation

---

# 21. HTTP Communication

Use typed/named `HttpClient` through `IHttpClientFactory`.

Prefer:

```text
Application
    ↓
Abstraction
    ↓
Infrastructure HTTP client
    ↓
Other service
```

Do not place raw `HttpClient` calls throughout handlers.

For high-value operations, define explicit contracts.

Use synchronous HTTP when the caller genuinely needs an immediate answer.

Use RabbitMQ when eventual consistency is acceptable.

---

# 22. Observability

Observability is mandatory for microservices.

Use OpenTelemetry for:

- Distributed traces
- Metrics
- Logs correlation

Recommended stack:

```text
.NET Services
     │
     ▼
OpenTelemetry
 ┌───┼────────┐
 ▼   ▼        ▼
Logs Metrics Traces
 │     │       │
 ▼     ▼       ▼
Loki  Prom   Tempo/Jaeger
        \     /
         Grafana
```

Every request/message should have:

- Trace ID
- Span ID
- Correlation ID
- Service name
- Environment
- Version

Track metrics such as:

- HTTP request count
- HTTP latency
- Error rate
- Database latency
- RabbitMQ publish/consume rate
- Consumer failures
- Retry count
- DLQ count
- Cache hit/miss ratio
- CPU/memory
- Queue depth

---

# 23. Health Checks

Implement at minimum:

```text
/liveness
/readiness
```

### Liveness

Answers:

> Is the process alive?

Do not make liveness depend on every external dependency.

### Readiness

Answers:

> Can this instance safely receive traffic?

May check:

- PostgreSQL
- RabbitMQ
- Required external services

Use readiness checks with container orchestration.

---

# 24. Logging

Use structured logging.

Good:

```text
OrderCreated
{
    "orderId": "...",
    "customerId": "...",
    "traceId": "..."
}
```

Bad:

```text
string interpolation with sensitive data
```

Prefer semantic properties:

```csharp
logger.LogInformation(
    "Order {OrderId} created for customer {CustomerId}",
    orderId,
    customerId);
```

Log levels:

- Trace
- Debug
- Information
- Warning
- Error
- Critical

Do not use `Information` for every internal implementation detail.

---

# 25. API Error Contract

Use RFC 7807-style ProblemDetails.

Example:

```json
{
  "type": "https://example.com/problems/order-not-found",
  "title": "Order not found",
  "status": 404,
  "detail": "The requested order does not exist.",
  "instance": "/orders/123",
  "traceId": "..."
}
```

Do not expose stack traces or internal database errors to clients.

---

# 26. API Design

Prefer resource-oriented REST APIs.

Examples:

```http
GET    /api/v1/orders/{id}
POST   /api/v1/orders
PUT    /api/v1/orders/{id}
DELETE /api/v1/orders/{id}
```

For explicit commands that represent important business operations:

```http
POST /api/v1/orders/{id}/confirm
POST /api/v1/orders/{id}/cancel
```

Use consistent:

- Naming
- Pagination
- Filtering
- Sorting
- Validation
- Error responses
- Status codes

Do not return database entities directly.

---

# 27. API Versioning

Do not break clients silently.

Possible strategies:

```text
/api/v1/orders
/api/v2/orders
```

or header/media-type versioning.

Choose one strategy and apply it consistently.

Event contracts also require versioning.

Never casually rename/remove event fields that existing consumers depend on.

---

# 28. Distributed Transactions

Never assume:

```text
Service A transaction
+
Service B transaction
=
one atomic transaction
```

Instead:

```text
Service A
  ↓
Event
  ↓
Service B
  ↓
Event
  ↓
Service C
```

For complex workflows use a Saga / Process Manager.

Example:

```text
Create Order
   ↓
Reserve Inventory
   ↓
Process Payment
   ↓
Create Shipment
```

Failure:

```text
Payment failed
   ↓
Release Inventory
   ↓
Cancel Order
```

Compensating actions are business operations, not database rollbacks.

---

# 29. Distributed Locking

Avoid distributed locks unless the business problem genuinely requires them.

Before using Redis locks, ask:

1. Can idempotency solve it?
2. Can optimistic concurrency solve it?
3. Can a database constraint solve it?
4. Can the operation be serialized through a queue?

Prefer those simpler mechanisms.

---

# 30. Concurrency Control

Use optimistic concurrency where appropriate.

PostgreSQL/EF Core options:

- Row version/concurrency token strategy
- Explicit version column
- Compare-and-update
- Unique constraints

Important operations should not assume two concurrent requests cannot happen.

---

# 31. Data Consistency

Classify every workflow:

### Strong consistency

Use within a single service/database transaction.

### Eventual consistency

Use across services with events.

Document:

```text
Consistency:
  Strong within OrderService.
  Eventual between OrderService and PaymentService.
```

The AI agent must explicitly identify consistency requirements during architecture design.

---

# 32. Shared Code Rules

Create shared libraries only for genuinely cross-cutting concerns.

Good candidates:

```text
BuildingBlocks/
├── Observability
├── Messaging
├── ErrorHandling
├── Security
└── Web
```

Be careful with shared libraries.

Forbidden:

```text
SharedDomain/
SharedEntities/
SharedDbContext/
SharedRepositories/
```

Do not create a giant `Common` project that becomes a distributed monolith.

Shared integration contracts are allowed when the organization deliberately manages contract compatibility, but services must not share domain models.

---

# 33. Repository Pattern

Do not automatically introduce repositories.

EF Core `DbContext` already provides Unit of Work semantics and query capabilities.

Prefer:

```text
Application abstraction
        ↓
DbContext
```

when appropriate.

Use a repository only when it expresses a meaningful domain/application abstraction rather than wrapping every EF Core method.

Bad:

```csharp
_repository.GetByIdAsync()
_repository.AddAsync()
_repository.UpdateAsync()
```

with no business value.

---

# 34. Dependency Injection

Use constructor injection.

Prefer explicit interfaces.

Avoid service locator patterns.

Avoid excessive lifetime complexity.

Default:

```text
Scoped
  - DbContext
  - request-level services

Singleton
  - stateless thread-safe infrastructure

Transient
  - lightweight stateless components
```

Verify lifetime compatibility.

Never inject a scoped dependency into a singleton.

---

# 35. Configuration

Use strongly typed options.

Example:

```csharp
public sealed class RabbitMqOptions
{
    public required string Host { get; init; }
    public required string Username { get; init; }
    public required string Password { get; init; }
}
```

Use:

```text
appsettings.json
appsettings.Development.json
environment variables
secret manager
```

Never commit:

```text
password
client secret
JWT signing secret
RabbitMQ password
database password
API key
```

---

# 36. Containerization

Every service must have a production-oriented Dockerfile.

Prefer multi-stage builds:

```text
SDK image
   ↓
restore
   ↓
build
   ↓
publish
   ↓
runtime image
```

Runtime container should:

- Run as non-root when possible.
- Have a health endpoint.
- Have minimal dependencies.
- Receive configuration through environment/configuration providers.
- Handle SIGTERM gracefully.
- Use deterministic image tags.

---

# 37. Local Development

Provide a local infrastructure environment.

Recommended:

```text
docker-compose.yml
```

or:

```text
.NET Aspire
```

Local infrastructure may include:

```text
PostgreSQL
RabbitMQ
Redis
OpenTelemetry collector
Grafana
Prometheus
Loki
Tempo/Jaeger
Keycloak
```

Do not require developers to manually install infrastructure when Docker can provide a reproducible environment.

---

# 38. Kubernetes Production Baseline

For Kubernetes deployments, define:

- Deployment
- Service
- ConfigMap
- Secret reference
- Ingress/Gateway
- Liveness probe
- Readiness probe
- Resource requests
- Resource limits
- Horizontal Pod Autoscaler where needed
- PodDisruptionBudget where appropriate
- NetworkPolicy where appropriate

Use Helm or Kustomize if the deployment complexity justifies it.

Do not introduce Kubernetes for a small system that can be operated more simply.

---

# 39. CI/CD

Pipeline should include:

```text
Checkout
  ↓
Restore
  ↓
Build
  ↓
Format check
  ↓
Static analysis
  ↓
Unit tests
  ↓
Integration tests
  ↓
Architecture tests
  ↓
Security/dependency scan
  ↓
Docker build
  ↓
Container scan
  ↓
Publish image
  ↓
Deploy
  ↓
Smoke tests
```

Never deploy code that failed tests.

Use immutable image tags.

Prefer:

```text
order-service:git-sha
```

over:

```text
order-service:latest
```

---

# 40. Testing Strategy

Use the testing pyramid.

```text
                 E2E
                /   \
           Component
              / \
       Integration
            /   \
          Unit Tests
```

## Unit tests

Test:

- Domain rules
- Value objects
- Application handlers
- Validators
- Pure business logic

## Integration tests

Use real infrastructure through Testcontainers:

```text
PostgreSQL
RabbitMQ
Redis
```

Do not replace everything with mocks.

## Architecture tests

Verify:

```text
Domain -> no Infrastructure dependency
Domain -> no ASP.NET dependency
Application -> no Infrastructure implementation dependency
API -> no direct database access
```

## Contract tests

For important integration events and APIs.

## End-to-end tests

Use selectively for critical user journeys.

---

# 41. Idempotency

All important write operations should consider idempotency.

Examples:

```text
POST /payments
Idempotency-Key: abc-123
```

For messaging:

```text
messageId + consumerName
```

must identify duplicate processing.

Do not claim exactly-once processing unless the architecture genuinely provides and proves it.

---

# 42. Audit Trail

For business-critical systems, identify operations requiring audit:

```text
Who
What
When
From where
Before
After
Reason
Correlation ID
```

Audit records should be append-oriented and protected from accidental modification.

Do not rely only on application logs for legally/business-critical audit history.

---

# 43. Background Jobs

Use a dedicated worker/background-processing mechanism when appropriate.

Potential technologies:

- ASP.NET Core `BackgroundService`
- MassTransit consumers
- Quartz.NET
- Hangfire

Choose based on the requirement.

Do not create a scheduler for every delayed task.

For broker-driven processing, prefer RabbitMQ consumers.

---

# 44. File Storage

Do not store large files inside PostgreSQL unless there is a strong reason.

Prefer:

```text
Object Storage
├── S3
├── MinIO
├── Azure Blob Storage
└── Google Cloud Storage
```

Store metadata in PostgreSQL:

```text
file_id
owner_id
object_key
content_type
size
created_at
```

---

# 45. Search

Do not introduce Elasticsearch/OpenSearch merely because the application has a search endpoint.

Start with PostgreSQL indexes/full-text search when sufficient.

Introduce OpenSearch/Elasticsearch when requirements justify:

- large-scale text search
- fuzzy search
- advanced ranking
- aggregations
- search analytics

---

# 46. Eventual Consistency UX

When a user performs an operation that completes asynchronously, the API must communicate this clearly.

Example:

```http
202 Accepted
```

with:

```json
{
  "operationId": "...",
  "status": "processing"
}
```

Do not return `200 OK` implying completion when the operation is still asynchronous.

---

# 47. Architecture Decision Records

For important architectural choices create ADRs.

Example:

```text
docs/adr/
├── 001-microservice-boundaries.md
├── 002-rabbitmq-messaging.md
├── 003-database-per-service.md
├── 004-outbox-pattern.md
└── 005-authentication.md
```

Each ADR:

```text
Context
Decision
Alternatives
Consequences
Status
```

---

# 48. Documentation Requirements

Every service should have:

```text
README.md
ARCHITECTURE.md
API.md
EVENTS.md
RUNBOOK.md
```

README should explain:

- Purpose
- Local setup
- Configuration
- Dependencies
- Database
- Messaging
- Tests
- Deployment

EVENTS should document:

- Event name
- Version
- Producer
- Consumers
- Payload
- Semantics
- Retry behavior
- Compatibility rules

---

# 49. AI Agent Workflow

When asked to create or modify a microservice, follow this sequence.

## Phase 1 — Understand

1. Read existing repository structure.
2. Read architecture documentation.
3. Identify existing conventions.
4. Identify bounded context.
5. Identify dependencies.
6. Ask only essential questions if requirements are ambiguous.

## Phase 2 — Design

Produce:

```text
Service responsibility
Bounded context
Entities
Aggregates
Commands
Queries
Domain events
Integration events
Database schema
API endpoints
Messaging topology
Failure modes
Security model
Observability
Testing strategy
```

## Phase 3 — Implement

Implement in this order:

```text
Domain
↓
Application
↓
Infrastructure
↓
Contracts
↓
API
↓
Messaging
↓
Database migrations
↓
Tests
↓
Docker
↓
Observability
```

## Phase 4 — Verify

Run:

```text
dotnet restore
dotnet build
dotnet test
dotnet format --verify-no-changes
```

Also verify:

- migrations compile
- containers start
- health checks work
- RabbitMQ publish/consume works
- duplicate messages are safe
- PostgreSQL migrations are valid
- Redis failure does not destroy correctness
- OpenTelemetry traces are emitted
- secrets are not committed

---

# 50. AI Agent Rules — MUST

The agent MUST:

- Preserve existing business behavior unless explicitly asked to change it.
- Keep service boundaries explicit.
- Keep databases isolated by service.
- Use UTC timestamps.
- Use CancellationToken in async application/infrastructure operations.
- Validate external input.
- Use ProblemDetails for API errors.
- Add structured logging.
- Add health checks.
- Add OpenTelemetry instrumentation.
- Consider idempotency for distributed operations.
- Consider Outbox for DB + event consistency.
- Use migrations for schema changes.
- Write tests for business-critical behavior.
- Document architectural decisions.
- Treat all distributed communication as failure-prone.
- Review generated migrations.
- Avoid leaking secrets.
- Avoid unnecessary abstractions.

---

# 51. AI Agent Rules — MUST NOT

The agent MUST NOT:

- Create a shared database for all services.
- Add cross-service foreign keys.
- Directly query another service's database.
- Put business logic in controllers.
- Put infrastructure dependencies in Domain.
- Put RabbitMQ code directly inside domain entities.
- Treat Redis as the source of truth for durable business data.
- Assume RabbitMQ gives exactly-once processing.
- Retry permanently invalid messages forever.
- Commit secrets.
- Use `latest` as the only production image tag.
- Create repositories merely to wrap EF Core.
- Create a giant shared `Common` project.
- Introduce Kubernetes, Kafka, Elasticsearch, service mesh, or other infrastructure without a concrete requirement.
- Add microservices solely to make the architecture look enterprise.
- Use distributed transactions unless explicitly justified and supported by the platform.
- Ignore backward compatibility of public APIs/events.
- Return EF Core entities directly from APIs.

---

# 52. Definition of Done — New Microservice

A microservice is not considered complete until:

### Architecture
- [ ] Bounded context documented
- [ ] Responsibility documented
- [ ] Database ownership defined
- [ ] Dependencies documented

### Code
- [ ] Clean Architecture implemented
- [ ] Domain rules protected
- [ ] Application layer separated
- [ ] Infrastructure isolated
- [ ] API controllers/endpoints thin

### Database
- [ ] PostgreSQL configured
- [ ] EF Core Code First configured
- [ ] Initial migration created
- [ ] Indexes reviewed
- [ ] Concurrency strategy considered

### Messaging
- [ ] RabbitMQ topology documented
- [ ] Event contracts versioned
- [ ] Publisher confirms configured
- [ ] Consumer ACK strategy configured
- [ ] Retry strategy configured
- [ ] DLQ strategy configured
- [ ] Idempotency implemented where required
- [ ] Outbox implemented where DB/event atomicity is required

### Redis
- [ ] Cache usage justified
- [ ] TTL defined
- [ ] Invalidation strategy defined
- [ ] Redis failure behavior defined

### Security
- [ ] Authentication defined
- [ ] Authorization defined
- [ ] Input validation implemented
- [ ] Secrets externalized
- [ ] Sensitive logs reviewed

### Observability
- [ ] Structured logs
- [ ] OpenTelemetry traces
- [ ] Metrics
- [ ] Liveness
- [ ] Readiness
- [ ] Correlation/trace IDs

### Testing
- [ ] Unit tests
- [ ] Integration tests
- [ ] Architecture tests
- [ ] Contract tests where needed
- [ ] Critical E2E tests where needed

### Delivery
- [ ] Dockerfile
- [ ] CI pipeline
- [ ] Container security scan
- [ ] Deployment configuration
- [ ] Rollback strategy
- [ ] README
- [ ] ADRs for important decisions

---

# 53. Recommended Enterprise Repository Structure

For a monorepo:

```text
src/
├── ApiGateway/
├── Identity/
├── Services/
│   ├── Order/
│   │   ├── Order.Api/
│   │   ├── Order.Application/
│   │   ├── Order.Domain/
│   │   ├── Order.Infrastructure/
│   │   └── Order.Contracts/
│   ├── Payment/
│   └── Inventory/
├── BuildingBlocks/
│   ├── Observability/
│   ├── Messaging/
│   ├── Web/
│   └── Security/
│
tests/
├── Order.UnitTests/
├── Order.IntegrationTests/
├── Payment.UnitTests/
└── ArchitectureTests/

deploy/
├── docker/
├── compose/
├── kubernetes/
└── helm/

docs/
├── architecture/
├── adr/
├── api/
└── events/
```

For a polyrepo, each service can own its repository while shared packages are versioned independently.

---

# 54. Suggested Initial Infrastructure

For a serious enterprise-style starter system:

```text
                    ┌───────────────┐
                    │   Frontends   │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ YARP Gateway  │
                    └───────┬───────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
   Service A           Service B           Service C
        │                   │                   │
     DB A                DB B                DB C
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                       RabbitMQ 4.x
                            │
                         Workers

              Redis ───────┬─────── Services

              Identity Provider
              (Keycloak/Entra)

              OpenTelemetry
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Metrics      Logs      Traces
          │          │          │
       Prometheus   Loki      Tempo
              \      |       /
                 Grafana
```

---

# 55. Technology Selection Rules

When evaluating an additional technology, the agent must answer:

1. What concrete problem does it solve?
2. Why can't the current stack solve it?
3. What operational cost does it add?
4. What failure modes does it introduce?
5. Does the team need to learn it?
6. Does it improve reliability or only developer convenience?
7. Can it be removed later without redesigning the whole system?

### Example

Do not automatically add Kafka because RabbitMQ exists.

Use RabbitMQ for:

- commands
- business integration events
- work queues
- moderate event-driven workflows

Consider Kafka/Redpanda when the actual requirement is:

- very high event throughput
- durable event log
- replay-heavy analytics pipelines
- stream processing
- multiple independent consumers over long retention

---

# 56. Recommended Optional Technologies

Only introduce these when requirements justify them.

### MassTransit

Recommended when the team wants a mature messaging abstraction around RabbitMQ, including:

- consumers
- retries
- middleware
- routing
- message conventions
- saga support

### .NET Aspire

Recommended for developer experience and local orchestration of distributed .NET applications.

It should not be confused with production orchestration.

### Keycloak

Recommended for:

- self-hosted identity
- OIDC/OAuth2
- roles
- groups
- service clients

### Vault / Cloud Secret Manager

Recommended for:

- production secrets
- credential rotation
- centralized secret management

### MinIO / S3

Recommended for object/file storage.

### OpenSearch / Elasticsearch

Add only when PostgreSQL search is insufficient.

### Quartz / Hangfire

Add only when scheduled/background jobs require durable scheduling beyond simple worker consumers.

---

# 57. Architecture Review Checklist

Before approving an architecture, ask:

### Boundaries
- Does each service own a coherent business capability?
- Can services be deployed independently?
- Are service boundaries based on business capabilities rather than database tables?

### Data
- Does every service own its database?
- Are cross-service joins avoided?
- Is consistency explicitly defined?

### Messaging
- Are events facts?
- Are consumers idempotent?
- Is Outbox required?
- Are retries bounded?
- Is DLQ behavior defined?

### Resilience
- What happens if PostgreSQL is unavailable?
- What happens if Redis is unavailable?
- What happens if RabbitMQ is unavailable?
- What happens if another service is slow?
- Can the system recover automatically?

### Security
- Who can call this API?
- How are permissions enforced?
- Where are secrets stored?
- Are sensitive values excluded from logs?

### Operations
- Can operators determine why a request failed?
- Can one request be traced across services?
- Are health checks available?
- Are alerts defined?

### Delivery
- Can the service be built from scratch?
- Can integration tests run locally?
- Can it be deployed reproducibly?
- Can it be rolled back?

---

# 58. Default Decision Summary

When no specific requirement contradicts it, choose:

```text
Language:
  C# 14

Runtime:
  .NET 10 LTS

Web:
  ASP.NET Core

Architecture:
  Clean Architecture

Application:
  CQRS selectively
  MediatR or equivalent mediator where justified

Database:
  PostgreSQL 18
  EF Core + Npgsql
  Code First

Messaging:
  RabbitMQ 4.x
  MassTransit or equivalent
  Publisher Confirms
  Manual ACK
  Retry
  DLQ
  Outbox
  Inbox/Idempotency

Cache:
  Redis
  Cache-aside
  TTL

Gateway:
  YARP

Authentication:
  OAuth2/OIDC
  Keycloak or Entra ID

Resilience:
  Microsoft resilience APIs / Polly

Observability:
  OpenTelemetry
  Prometheus
  Grafana
  Loki
  Tempo/Jaeger

Containers:
  Docker

Local orchestration:
  Docker Compose or .NET Aspire

Production:
  Kubernetes when scale/operational requirements justify it

Testing:
  xUnit
  Testcontainers
  Architecture tests
  Contract tests

CI/CD:
  GitHub Actions / Azure DevOps / GitLab CI

Security:
  Secret manager
  Dependency scanning
  Container scanning
  SAST
```

---

# 59. Final Principle

The goal is **not** to maximize the number of technologies.

The goal is:

```text
Simple where possible
+
Explicit boundaries
+
Reliable communication
+
Independent data ownership
+
Observable runtime
+
Secure defaults
+
Automated testing
+
Reproducible deployment
=
Enterprise-grade microservices
```

An AI agent should always prefer the **simplest architecture that satisfies the actual business and operational requirements**.

