# LLM Assistant Instructions for Aspire13App

This document provides guidance for Large Language Model (LLM) assistants working with this .NET Aspire 13 project. Following these instructions ensures consistent, high-quality assistance while adhering to Aspire best practices.

## Table of Contents

1. [Project Context](#project-context)
2. [Critical Rules](#critical-rules)
3. [Aspire CLI Usage](#aspire-cli-usage)
4. [Project Structure](#project-structure)
5. [Common Tasks](#common-tasks)
6. [Code Patterns](#code-patterns)
7. [Documentation Maintenance](#documentation-maintenance)
8. [Troubleshooting](#troubleshooting)

## Project Context

### Technology Stack

- **.NET Aspire**: 13.0.0 (latest)
- **.NET SDK**: 10.0.100 (supports .NET 8.0, 9.0, 10.0)
- **Frontend**: Blazor Server (Interactive Server Components)
- **Backend**: ASP.NET Core Minimal APIs
- **Cache**: Redis (containerized)
- **Observability**: OpenTelemetry (built-in)

### Project Philosophy

This is a **cloud-native, microservices-based application** built with Aspire's orchestration capabilities. All services are:
- Observable by default (logs, traces, metrics)
- Discoverable (automatic service discovery)
- Resilient (health checks, dependency management)
- Deployable anywhere (Azure, Kubernetes, Docker)

## Critical Rules

### Rule 1: Always Use Aspire CLI

**DO:**
```bash
aspire run           # Start the application
aspire add redis     # Add integrations
aspire new          # Create new projects
aspire deploy       # Deploy to cloud
```

**DO NOT:**
```bash
dotnet run          # Unless specifically requested
dotnet new aspire-* # Use aspire CLI instead
```

### Rule 2: Maintain Documentation

**ALWAYS** update these files when making changes:
- `README.md` - Project overview and getting started
- `docs/PROJECT_STATUS.md` - Current state and recent changes
- `docs/architecture/OVERVIEW.md` - When adding/removing services
- `docs/guides/DEVELOPMENT.md` - When adding new workflows

### Rule 3: Follow Aspire Patterns

**AppHost Configuration:**
```csharp
// CORRECT: Proper dependency management
var cache = builder.AddRedis("cache");
var api = builder.AddProject<Projects.Api>("api")
    .WithReference(cache)
    .WaitFor(cache);

// WRONG: No dependency management
var cache = builder.AddRedis("cache");
var api = builder.AddProject<Projects.Api>("api");
```

### Rule 4: Service Discovery Over Hard-Coded URLs

**CORRECT:**
```csharp
builder.Services.AddHttpClient<WeatherApiClient>(client =>
    client.BaseAddress = new("https+http://apiservice"));
```

**WRONG:**
```csharp
builder.Services.AddHttpClient<WeatherApiClient>(client =>
    client.BaseAddress = new("https://localhost:5001"));
```

### Rule 5: Health Checks Are Mandatory

**ALWAYS** add health checks to new services:
```csharp
builder.AddProject<Projects.NewService>("newservice")
    .WithHttpHealthCheck("/health");
```

## Aspire CLI Usage

### Project Initialization

```bash
# Check Aspire version
aspire --version

# Update Aspire CLI
curl -sSL https://aspire.dev/install.sh | bash  # Linux/macOS
iex "& { $(irm https://aspire.dev/install.ps1) }"  # Windows

# Create new Aspire project
aspire new
```

### Running and Debugging

```bash
# Run in development mode
aspire run

# Run with specific project
aspire run --project ./Aspire13App.AppHost/Aspire13App.AppHost.csproj

# Enable experimental features
aspire config set features.execCommandEnabled true
aspire exec --resource api -- dotnet ef migrations add Init
```

### Adding Integrations

```bash
# Interactive integration picker
aspire add

# Add specific integrations
aspire add Aspire.Hosting.PostgreSQL
aspire add Aspire.Hosting.RabbitMQ
aspire add Aspire.Hosting.Azure.Storage

# Search for integrations
aspire add postgres  # Fuzzy search
```

### Configuration Management

```bash
# List all configuration
aspire config list

# Set configuration
aspire config set features.deployCommandEnabled true

# Get specific config value
aspire config get features.deployCommandEnabled
```

### Deployment

```bash
# Generate deployment artifacts
aspire publish --output ./artifacts

# Deploy to Azure (requires feature flag)
aspire config set features.deployCommandEnabled true
aspire deploy
```

## Project Structure

### Directory Layout

```
Aspire13App/
├── Aspire13App.AppHost/           # Orchestration project (START HERE)
│   ├── AppHost.cs                 # Main orchestration configuration
│   └── Aspire13App.AppHost.csproj # Aspire.AppHost.Sdk/13.0.0
│
├── Aspire13App.Web/               # Blazor frontend
│   ├── Components/                # Razor components
│   ├── Program.cs                 # Service configuration
│   └── WeatherApiClient.cs        # HTTP client for API
│
├── Aspire13App.ApiService/        # Backend API
│   ├── Program.cs                 # API endpoints & health checks
│   └── Aspire13App.ApiService.csproj
│
├── Aspire13App.ServiceDefaults/   # Shared configurations
│   ├── Extensions.cs              # Service defaults extension methods
│   └── Aspire13App.ServiceDefaults.csproj
│
└── docs/                          # KEEP CURRENT
    ├── architecture/              # Architecture documentation
    ├── guides/                    # How-to guides
    ├── llm/                       # LLM instructions (this file)
    └── PROJECT_STATUS.md          # Current state tracking
```

### Key Files to Understand

1. **`Aspire13App.AppHost/AppHost.cs`**
   - Defines the entire application topology
   - Configures service dependencies and startup order
   - Manages integrations (Redis, databases, etc.)

2. **`Aspire13App.ServiceDefaults/Extensions.cs`**
   - Shared configurations (telemetry, health checks)
   - Applied to all projects via `builder.AddServiceDefaults()`

3. **`Aspire13App.Web/Program.cs`**
   - Blazor frontend startup
   - Configures Redis output caching
   - Sets up HTTP client with service discovery

4. **`Aspire13App.ApiService/Program.cs`**
   - API endpoints definition
   - Health check endpoints
   - OpenAPI configuration

## Common Tasks

### Adding a New Service

1. **Create the project:**
   ```bash
   dotnet new webapi -n Aspire13App.NewService
   dotnet sln add Aspire13App.NewService
   ```

2. **Add ServiceDefaults reference:**
   ```bash
   dotnet add Aspire13App.NewService reference Aspire13App.ServiceDefaults
   ```

3. **Configure Program.cs:**
   ```csharp
   var builder = WebApplication.CreateBuilder(args);

   builder.AddServiceDefaults();  // Add this!

   var app = builder.Build();
   app.MapDefaultEndpoints();     // Add this!
   app.Run();
   ```

4. **Register in AppHost:**
   ```csharp
   var newService = builder.AddProject<Projects.Aspire13App_NewService>("newservice")
       .WithHttpHealthCheck("/health");
   ```

5. **Update documentation:**
   - Update `README.md` architecture section
   - Update `docs/architecture/OVERVIEW.md`
   - Update `docs/PROJECT_STATUS.md`

### Adding a Database

1. **Add via Aspire CLI:**
   ```bash
   aspire add postgres  # or sql-server, mongodb, etc.
   ```

2. **Configure in AppHost:**
   ```csharp
   var postgres = builder.AddPostgres("postgres")
       .WithPgAdmin();  // Optional: Add management UI

   var database = postgres.AddDatabase("mydb");

   var api = builder.AddProject<Projects.Aspire13App_ApiService>("api")
       .WithReference(database)
       .WaitFor(database);
   ```

3. **Use in service:**
   ```csharp
   builder.AddNpgsqlDbContext<MyDbContext>("mydb");
   ```

### Adding Azure Services

1. **Add Azure hosting package:**
   ```bash
   aspire add Aspire.Hosting.Azure.Storage
   ```

2. **Configure in AppHost:**
   ```csharp
   var storage = builder.AddAzureStorage("storage");
   var blobs = storage.AddBlobs("blobs");

   builder.AddProject<Projects.Aspire13App_ApiService>("api")
       .WithReference(blobs);
   ```

### Adding Message Queues

```bash
# Add RabbitMQ
aspire add rabbitmq

# Or Azure Service Bus
aspire add Aspire.Hosting.Azure.ServiceBus
```

```csharp
// AppHost configuration
var messaging = builder.AddRabbitMQ("messaging")
    .WithManagementPlugin();

var api = builder.AddProject<Projects.Aspire13App_ApiService>("api")
    .WithReference(messaging);
```

## Code Patterns

### Service Discovery Pattern

```csharp
// In consuming service
builder.Services.AddHttpClient<IMyClient, MyClient>(client =>
{
    // Service name matches AppHost registration
    client.BaseAddress = new("https+http://servicename");
});
```

### Health Check Pattern

```csharp
// In Program.cs
var app = builder.Build();

// Default endpoints include /health and /alive
app.MapDefaultEndpoints();
```

### Caching Pattern

```csharp
// In AppHost
var cache = builder.AddRedis("cache");

// In service Program.cs
builder.AddRedisOutputCache("cache");
// or
builder.AddRedisDistributedCache("cache");
```

### Configuration Pattern

```csharp
// In AppHost - use parameters for secrets
var apiKey = builder.AddParameter("api-key", secret: true);

builder.AddProject<Projects.Api>("api")
    .WithEnvironment("API_KEY", apiKey);
```

### Dependency Management Pattern

```csharp
// Ensure startup order
var database = builder.AddPostgres("pg").AddDatabase("db");
var cache = builder.AddRedis("cache");

var api = builder.AddProject<Projects.Api>("api")
    .WaitFor(database)    // Wait for database
    .WaitFor(cache)       // Wait for cache
    .WithReference(database)
    .WithReference(cache);

var web = builder.AddProject<Projects.Web>("web")
    .WaitFor(api)        // Wait for API
    .WithReference(api);
```

## Documentation Maintenance

### When to Update Documentation

**Always update `docs/PROJECT_STATUS.md` when:**
- Adding or removing services
- Changing dependencies
- Modifying integrations
- Deploying to new environments
- Completing major features

**Always update `README.md` when:**
- Changing prerequisites
- Adding new workflows
- Modifying getting started steps
- Changing deployment processes

**Always update `docs/architecture/OVERVIEW.md` when:**
- Adding or removing services
- Changing service communication patterns
- Modifying data flow
- Changing deployment architecture

**Always update `docs/guides/DEVELOPMENT.md` when:**
- Adding new development workflows
- Changing build processes
- Adding new tools or dependencies
- Modifying testing procedures

### Documentation Format Standards

#### Dates
Use ISO 8601 format: `2025-11-12`

#### Code Examples
Always include language identifiers:
```bash
aspire run
```

```csharp
var builder = DistributedApplication.CreateBuilder(args);
```

#### File Paths
Use relative paths from project root:
- ✅ `Aspire13App.AppHost/AppHost.cs`
- ❌ `/Users/john/projects/Aspire13App/Aspire13App.AppHost/AppHost.cs`

#### Version Numbers
Always specify versions:
- ✅ `.NET Aspire 13.0.0`
- ❌ `.NET Aspire`

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Docker is not running"

**Solution:**
```bash
# Verify Docker is running
docker ps

# Start Docker Desktop or Podman
# Then restart Aspire
aspire run
```

#### Issue: "AppHost not found"

**Solution:**
```bash
# Aspire CLI searches for AppHost automatically
# Ensure you're in the project root or subdirectory
cd Aspire13App
aspire run

# Or specify project explicitly
aspire run --project ./Aspire13App.AppHost/Aspire13App.AppHost.csproj
```

#### Issue: "Port already in use"

**Solution:**
Aspire automatically assigns different ports. Check the dashboard for actual ports. Don't hardcode ports in service discovery.

#### Issue: "Certificate trust errors"

**Solution:**
```bash
# Aspire CLI manages certificates automatically
aspire run
# Follow prompts to trust certificates
```

#### Issue: "Service discovery not working"

**Check:**
1. Service name in `AddProject<>()` matches HTTP client base address
2. `AddServiceDefaults()` is called in both services
3. `MapDefaultEndpoints()` is called in target service

```csharp
// AppHost
var api = builder.AddProject<Projects.Api>("apiservice");

// Consuming service
builder.Services.AddHttpClient<Client>(c =>
    c.BaseAddress = new("https+http://apiservice")); // Must match
```

## Best Practices Summary

### DO

✅ Use `aspire run` to start the application
✅ Use service discovery for inter-service communication
✅ Add health checks to all services
✅ Use `.WaitFor()` for dependencies
✅ Keep documentation current
✅ Use `builder.AddServiceDefaults()` in all services
✅ Use parameters for secrets
✅ Add observability from the start
✅ Test locally before deployment

### DO NOT

❌ Hardcode URLs or ports
❌ Skip health checks
❌ Ignore dependency order
❌ Forget to update documentation
❌ Commit secrets to version control
❌ Skip service defaults
❌ Use blocking calls without timeouts
❌ Deploy without testing locally

## Integration with AI Tools

### Aspire MCP (Model Context Protocol)

Aspire 13 includes an MCP server in the dashboard. LLM assistants can:

1. **Query Resources:**
   ```
   "List all resources with their health status"
   "Show me the logs for the API service"
   "What's the current state of the Redis cache?"
   ```

2. **Execute Commands:**
   ```
   "Run Entity Framework migrations on the database"
   "Restart the API service"
   ```

3. **Analyze Traces:**
   ```
   "Show me traces for failed requests"
   "Analyze the performance of the weather endpoint"
   ```

### Working with This Project

When assisting with this project:

1. **Always check** `docs/PROJECT_STATUS.md` first
2. **Reference** `docs/architecture/OVERVIEW.md` for system design
3. **Update documentation** after making changes
4. **Use Aspire CLI** for all operations
5. **Follow patterns** from existing code
6. **Maintain consistency** with project structure

## Questions?

For questions about this project:
1. Check `docs/` directory first
2. Review Aspire official docs: https://learn.microsoft.com/en-us/dotnet/aspire/
3. Check Aspire GitHub: https://github.com/dotnet/aspire

---

**Last Updated:** 2025-11-12
**Aspire Version:** 13.0.0
**Maintained By:** Project Team
