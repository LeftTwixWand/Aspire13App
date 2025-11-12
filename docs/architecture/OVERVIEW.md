# Architecture Overview - Aspire13App

## System Architecture

Aspire13App is a cloud-native distributed application built using .NET Aspire 13's orchestration capabilities. The architecture follows microservices patterns with service discovery, health monitoring, and observability built-in.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      Aspire Dashboard                           │
│             (Observability & Management)                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Monitors & Controls
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     AppHost (Orchestrator)                      │
│                                                                 │
│  • Service Discovery Configuration                              │
│  • Dependency Management                                        │
│  • Resource Lifecycle Management                                │
│  • Health Check Coordination                                    │
└─────────────────────────────────────────────────────────────────┘
           │                   │                   │
           │                   │                   │
           ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐      ┌──────────────┐
    │   Web    │        │   API    │      │    Redis     │
    │ Frontend │───────▶│ Service  │      │    Cache     │
    └──────────┘        └──────────┘      └──────────────┘
         │                   │                   ▲
         │                   └───────────────────┘
         │                  Uses for caching
         │
         ▼
    ┌──────────┐
    │  Users   │
    └──────────┘
```

## Components

### 1. AppHost (Aspire13App.AppHost)

**Purpose:** Orchestration and service composition layer

**Responsibilities:**
- Define application topology
- Configure service dependencies
- Manage resource lifecycle
- Coordinate health checks
- Provide service discovery configuration

**Key Code:**
```csharp
var builder = DistributedApplication.CreateBuilder(args);

var cache = builder.AddRedis("cache");

var apiService = builder.AddProject<Projects.Aspire13App_ApiService>("apiservice")
    .WithHttpHealthCheck("/health");

builder.AddProject<Projects.Aspire13App_Web>("webfrontend")
    .WithExternalHttpEndpoints()
    .WithHttpHealthCheck("/health")
    .WithReference(cache)
    .WaitFor(cache)
    .WithReference(apiService)
    .WaitFor(apiService);
```

**Technologies:**
- Aspire.AppHost.Sdk 13.0.0
- .NET 10.0

### 2. Web Frontend (Aspire13App.Web)

**Purpose:** User-facing web interface

**Responsibilities:**
- Render Blazor UI components
- Communicate with API service
- Implement output caching
- Provide interactive user experience

**Architecture:**
- **Framework:** ASP.NET Core Blazor Server
- **Rendering:** Interactive Server Components
- **Communication:** HTTP Client with Service Discovery
- **Caching:** Redis-backed output cache

**Key Endpoints:**
- `/` - Home page
- `/weather` - Weather forecast display
- `/health` - Health check endpoint

**Service Discovery:**
```csharp
builder.Services.AddHttpClient<WeatherApiClient>(client =>
    client.BaseAddress = new("https+http://apiservice"));
```

### 3. API Service (Aspire13App.ApiService)

**Purpose:** Backend API providing data services

**Responsibilities:**
- Expose RESTful endpoints
- Process business logic
- Provide health status
- Generate sample data

**Architecture:**
- **Framework:** ASP.NET Core Minimal APIs
- **API Documentation:** OpenAPI/Swagger
- **Observability:** Built-in telemetry

**Key Endpoints:**
- `GET /` - API status
- `GET /weatherforecast` - Weather data
- `GET /health` - Health check
- `GET /alive` - Liveness probe

**Sample Response:**
```json
[
  {
    "date": "2025-11-13",
    "temperatureC": 15,
    "temperatureF": 59,
    "summary": "Mild"
  }
]
```

### 4. Redis Cache

**Purpose:** Distributed caching layer

**Responsibilities:**
- Cache HTTP responses
- Improve response times
- Reduce backend load

**Configuration:**
- **Container:** redis:latest
- **Port:** Dynamically assigned
- **Persistence:** In-memory (development)

**Usage:**
```csharp
builder.AddRedisOutputCache("cache");
```

### 5. Service Defaults (Aspire13App.ServiceDefaults)

**Purpose:** Shared configuration library

**Responsibilities:**
- Configure OpenTelemetry
- Setup health checks
- Configure service discovery
- Provide common middleware

**Applied To:** All service projects

**Usage:**
```csharp
builder.AddServiceDefaults();
```

## Communication Patterns

### 1. Service Discovery

Services communicate using logical names that are resolved at runtime:

```
Web Frontend → "https+http://apiservice" → API Service
```

**Benefits:**
- No hardcoded URLs
- Environment-agnostic
- Automatic load balancing
- HTTPS preferred, HTTP fallback

### 2. HTTP Communication

**Protocol:** HTTP/HTTPS
**Format:** JSON
**Client:** System.Net.Http.HttpClient with named clients

**Example:**
```csharp
public class WeatherApiClient(HttpClient httpClient)
{
    public async Task<WeatherForecast[]> GetWeatherAsync()
    {
        return await httpClient.GetFromJsonAsync<WeatherForecast[]>("/weatherforecast");
    }
}
```

### 3. Caching Strategy

**Pattern:** Output caching with Redis
**Scope:** HTTP responses
**Invalidation:** Time-based

```csharp
app.MapGet("/weather", async (WeatherApiClient client) =>
{
    return await client.GetWeatherAsync();
})
.CacheOutput(); // Cached in Redis
```

## Observability Architecture

### Telemetry Stack

```
Application Code
      │
      ├──▶ Logs ────────┐
      ├──▶ Traces ──────┤
      └──▶ Metrics ─────┤
                        │
                        ▼
                 OpenTelemetry
                        │
                        ▼
                Aspire Dashboard
```

### Structured Logging

**Provider:** Microsoft.Extensions.Logging
**Sink:** OpenTelemetry
**Format:** Structured JSON

**Log Levels:**
- Trace: Detailed diagnostic information
- Debug: Internal system events
- Information: General flow
- Warning: Unusual events
- Error: Failures
- Critical: Catastrophic failures

### Distributed Tracing

**Standard:** OpenTelemetry
**Trace Context:** W3C Trace Context
**Visualization:** Aspire Dashboard

**Captured:**
- HTTP requests/responses
- Database queries
- Cache operations
- External API calls

### Metrics

**Format:** OpenTelemetry Metrics
**Collection:** Automatic
**Dashboard:** Real-time visualization

**Key Metrics:**
- Request rate
- Response time (p50, p95, p99)
- Error rate
- Cache hit ratio
- Resource utilization

## Health Check Architecture

### Health Check Types

1. **Liveness Probes** (`/alive`)
   - Service is running
   - Can accept requests

2. **Readiness Probes** (`/health`)
   - Service is ready for traffic
   - Dependencies are healthy
   - Can process requests successfully

### Health Check Flow

```
AppHost
   │
   ├──▶ Web Frontend
   │     └──▶ /health → Check API connectivity, Redis connectivity
   │
   ├──▶ API Service
   │     └──▶ /health → Check service status
   │
   └──▶ Redis
         └──▶ TCP check → Port availability
```

### Dependency Health

Services wait for dependencies before starting:

```csharp
.WaitFor(cache)      // Wait for Redis
.WaitFor(apiService) // Wait for API
```

## Deployment Architecture

### Local Development

```
Developer Machine
    │
    ├── Docker Desktop (Redis container)
    ├── AppHost Process
    ├── Web Frontend Process
    ├── API Service Process
    └── Aspire Dashboard (Browser)
```

### Azure Container Apps

```
Azure Subscription
    │
    ├── Container Apps Environment
    │   ├── Web Frontend (Container)
    │   ├── API Service (Container)
    │   └── Ingress Configuration
    │
    ├── Azure Cache for Redis
    │
    └── Application Insights (Telemetry)
```

### Kubernetes

```
Kubernetes Cluster
    │
    ├── Namespace: aspire13app
    │
    ├── Deployments
    │   ├── web-frontend (3 replicas)
    │   ├── api-service (3 replicas)
    │   └── redis (1 replica)
    │
    ├── Services
    │   ├── web-frontend-svc
    │   ├── api-service-svc
    │   └── redis-svc
    │
    └── Ingress
        └── Load Balancer → web-frontend-svc
```

## Data Flow

### Weather Forecast Request

```
1. User Browser
      │ GET /weather
      ▼
2. Web Frontend
      │ HTTP GET https+http://apiservice/weatherforecast
      ▼
3. Service Discovery
      │ Resolve "apiservice" → https://api-service:5001
      ▼
4. API Service
      │ Generate weather data
      │ Add OpenTelemetry trace
      ▼
5. HTTP Response (JSON)
      │
      ▼
6. Web Frontend (caching)
      │ Store in Redis
      │ Render Blazor component
      ▼
7. User Browser
      │ Display weather
```

### Caching Flow

```
1. Request arrives at Web Frontend
      ▼
2. Check Redis cache
      │
      ├──▶ Cache HIT ──▶ Return cached response (fast)
      │
      └──▶ Cache MISS
            │
            ▼
      3. Call API Service
            │
            ▼
      4. Store response in Redis
            │
            ▼
      5. Return response to client
```

## Security Architecture

### Authentication & Authorization

**Current State:** Development mode (no authentication)

**Production Considerations:**
- Add Azure AD / Entra ID integration
- Implement JWT bearer authentication
- Use Aspire parameter resources for secrets
- Configure API scopes and permissions

### Secrets Management

**Development:**
```csharp
var apiKey = builder.AddParameter("api-key", secret: true);
```

**Production:**
- Azure Key Vault integration
- Kubernetes secrets
- Environment variables (encrypted)

### Network Security

**Service-to-Service:**
- HTTPS preferred (TLS 1.2+)
- HTTP fallback for local dev
- Service mesh for production (optional)

**External Access:**
- `.WithExternalHttpEndpoints()` for public access
- Internal services not exposed externally

## Scalability Considerations

### Horizontal Scaling

**Stateless Services:**
- Web Frontend: Scale to N instances
- API Service: Scale to N instances

**Stateful Services:**
- Redis: Single instance (dev) → Redis Cluster (prod)

### Load Balancing

**Azure Container Apps:** Built-in load balancer
**Kubernetes:** Service load balancing
**Local:** Round-robin (Aspire orchestration)

### Caching Strategy

**Benefits:**
- Reduced API calls
- Faster response times
- Lower backend load

**Scaling Redis:**
```csharp
// Development: Container
var cache = builder.AddRedis("cache");

// Production: Azure Cache for Redis
var cache = builder.AddAzureRedis("cache")
    .WithClustering();
```

## Future Architecture Enhancements

### Planned Features

1. **Database Integration**
   - PostgreSQL or SQL Server
   - Entity Framework Core
   - Database migrations

2. **Message Queue**
   - RabbitMQ or Azure Service Bus
   - Asynchronous processing
   - Event-driven architecture

3. **Authentication**
   - Azure AD / Entra ID
   - JWT tokens
   - Role-based access control

4. **API Gateway**
   - YARP reverse proxy
   - Rate limiting
   - Request transformation

5. **Background Workers**
   - Scheduled jobs
   - Long-running tasks
   - Queue processing

## Architecture Decision Records (ADRs)

### ADR-001: Aspire 13 as Orchestration Platform

**Status:** Accepted
**Date:** 2025-11-12

**Context:** Need orchestration for microservices
**Decision:** Use .NET Aspire 13
**Consequences:**
- ✅ Built-in service discovery
- ✅ Excellent observability
- ✅ Simplified local development
- ⚠️ Requires .NET ecosystem knowledge

### ADR-002: Blazor Server for Frontend

**Status:** Accepted
**Date:** 2025-11-12

**Context:** Need interactive web UI
**Decision:** Use Blazor Server with Interactive Components
**Consequences:**
- ✅ Type-safe C# on frontend
- ✅ Real-time updates
- ✅ No JavaScript frameworks needed
- ⚠️ Requires SignalR connection

### ADR-003: Redis for Caching

**Status:** Accepted
**Date:** 2025-11-12

**Context:** Need distributed caching
**Decision:** Use Redis with Aspire integration
**Consequences:**
- ✅ Fast in-memory cache
- ✅ Simple integration
- ✅ Widely supported
- ⚠️ Additional infrastructure

## Monitoring & Alerts

### Key Metrics to Monitor

1. **Availability**
   - Uptime percentage
   - Health check status

2. **Performance**
   - Response time (p95, p99)
   - Throughput (requests/sec)

3. **Errors**
   - Error rate
   - Exception types

4. **Resources**
   - CPU utilization
   - Memory usage
   - Network I/O

### Alert Thresholds

- Response time > 1000ms (warning)
- Error rate > 5% (critical)
- Health check failures > 3 consecutive (critical)
- Memory usage > 85% (warning)

## References

- [Aspire Official Docs](https://learn.microsoft.com/en-us/dotnet/aspire/)
- [Aspire Architecture Guide](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/app-host-overview)
- [OpenTelemetry](https://opentelemetry.io/)
- [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/)

---

**Last Updated:** 2025-11-12
**Version:** 1.0.0
**Author:** Development Team
