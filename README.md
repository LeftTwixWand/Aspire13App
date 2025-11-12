# Aspire13App

## Quick Start for LLMs

Copy and paste this block into your LLM when starting to work on this project:

```text
PROJECT CONTEXT - Aspire13App

This is a .NET Aspire 13 distributed application with microservices architecture.

TECHNOLOGY STACK:
- .NET Aspire: 13.0.0 (orchestration platform)
- .NET SDK: 10.0.100
- Aspire CLI: 9.5.2 (ALWAYS use this for operations)
- Frontend: Blazor Server (Interactive Components)
- Backend: ASP.NET Core Minimal APIs
- Cache: Redis (containerized)
- Observability: OpenTelemetry (built-in)

ARCHITECTURE:
1. Aspire13App.AppHost - Orchestrator (defines topology, service discovery)
2. Aspire13App.Web - Blazor frontend (depends on API + Redis)
3. Aspire13App.ApiService - REST API backend
4. Aspire13App.ServiceDefaults - Shared configurations
5. Redis - Distributed cache (container)

CRITICAL RULES:
✅ ALWAYS use 'aspire run' to start (not 'dotnet run')
✅ ALWAYS use 'aspire add' for integrations (not manual NuGet)
✅ ALWAYS use service discovery (e.g., "https+http://apiservice")
✅ ALWAYS add health checks to new services
✅ ALWAYS update docs/ when making changes
✅ NEVER hardcode URLs or ports
✅ NEVER skip .WaitFor() for dependencies

KEY FILES:
- Aspire13App.AppHost/AppHost.cs - Service orchestration config
- docs/llm/INSTRUCTIONS.md - COMPLETE LLM guidelines (READ THIS FIRST!)
- docs/PROJECT_STATUS.md - Current project state
- docs/architecture/OVERVIEW.md - System architecture
- docs/guides/DEVELOPMENT.md - Development workflows

COMMON COMMANDS:
- aspire run                    # Start application
- aspire add                    # Add integrations (interactive)
- aspire config list            # Show configuration
- dotnet build                  # Build solution
- dotnet test                   # Run tests

BEFORE DOING ANYTHING:
1. Read docs/llm/INSTRUCTIONS.md for complete guidelines
2. Read docs/PROJECT_STATUS.md for current state
3. Use Aspire CLI for all operations
4. Update documentation after changes

GETTING HELP:
- docs/llm/INSTRUCTIONS.md - Comprehensive LLM guidelines
- docs/guides/DEVELOPMENT.md - Development guide
- https://learn.microsoft.com/en-us/dotnet/aspire/ - Official docs
```

---

## What is .NET Aspire?

.NET Aspire is Microsoft's official cloud-native application development stack that simplifies building, deploying, and managing distributed applications. Aspire 13 is the latest major release featuring polyglot support, enhanced AI integrations, and the powerful `aspire do` command for flexible deployment pipelines.

## Project Overview

This project showcases a complete .NET Aspire 13 application with:

- **Blazor Web Frontend** - Interactive server-side UI
- **API Service** - RESTful backend with weather forecast sample
- **Redis Cache** - Distributed caching integration
- **AppHost Orchestration** - Service discovery and health monitoring
- **Service Defaults** - Shared configurations and telemetry

## Architecture

```
Aspire13App/
├── Aspire13App.AppHost/          # Orchestration host (defines app topology)
├── Aspire13App.Web/              # Blazor frontend application
├── Aspire13App.ApiService/       # Backend API service
├── Aspire13App.ServiceDefaults/  # Shared configurations
└── docs/                         # Project documentation
    ├── architecture/             # Architecture diagrams and decisions
    ├── guides/                   # Development and operational guides
    └── llm/                      # Instructions for LLM assistants
```

## Prerequisites

### Required

1. **.NET 10.0 SDK** (or .NET 8.0/9.0)
   - Download: https://dotnet.microsoft.com/download/dotnet/10.0

2. **Docker Desktop** or **Podman** (for Redis container)
   - Docker: https://www.docker.com/products/docker-desktop
   - Podman: https://podman.io/

3. **Aspire CLI** (version 9.5.2+)
   ```bash
   # Install on Linux/macOS
   curl -sSL https://aspire.dev/install.sh | bash

   # Install on Windows (PowerShell)
   iex "& { $(irm https://aspire.dev/install.ps1) }"

   # Verify installation
   aspire --version
   ```

### Optional

- **Visual Studio 2022** (17.9+) with Aspire workload
- **Visual Studio Code** with C# Dev Kit extension
- **JetBrains Rider** with Aspire plugin

## Getting Started

### 1. Clone and Setup

```bash
# Navigate to project directory
cd Aspire13App

# Restore dependencies
dotnet restore
```

### 2. Run the Application

Using Aspire CLI (recommended):

```bash
# Start all services with orchestration
aspire run
```

The Aspire CLI will:
- Install and trust local hosting certificates
- Build all projects
- Start Redis container automatically
- Launch the API service
- Launch the Web frontend
- Open the Aspire Dashboard

Using .NET CLI:

```bash
# Run the AppHost project
dotnet run --project Aspire13App.AppHost
```

### 3. Access the Application

After running, you'll see output like:

```
Dashboard: https://localhost:17178/login?t=<token>
    Logs: ~/.aspire/cli/logs/apphost-<pid>-<timestamp>.log
```

- **Aspire Dashboard**: Real-time monitoring of all resources
- **Web Frontend**: https://localhost:7xxx (check dashboard for exact port)
- **API Service**: https://localhost:5xxx/weatherforecast

## Key Features

### Service Discovery

The frontend automatically discovers the API service using Aspire's built-in service discovery:

```csharp
builder.Services.AddHttpClient<WeatherApiClient>(client =>
    client.BaseAddress = new("https+http://apiservice"));
```

The `apiservice` name is resolved automatically without manual configuration.

### Health Checks

All services include HTTP health probes:

```csharp
builder.AddProject<Projects.Aspire13App_ApiService>("apiservice")
    .WithHttpHealthCheck("/health");
```

Monitor health status in the Aspire Dashboard.

### Output Caching with Redis

The web frontend uses Redis for distributed caching:

```csharp
builder.AddRedisOutputCache("cache");
```

Redis is automatically provisioned as a container.

### Observability

Built-in OpenTelemetry integration provides:
- **Logs**: Structured logging from all services
- **Traces**: Distributed tracing across service calls
- **Metrics**: Performance and health metrics
- **Dashboard**: Real-time visualization

## Development Workflow

### Adding New Services

```bash
# Add a new project service
dotnet new webapi -n Aspire13App.OrderService

# Reference it in AppHost
# Edit Aspire13App.AppHost/AppHost.cs
```

### Adding Integrations

```bash
# Interactive integration browser
aspire add

# Add specific integration
aspire add postgres
aspire add rabbitmq
aspire add azureblob
```

### Running Tests

```bash
# Run all tests
dotnet test

# Run specific test project
dotnet test Aspire13App.Tests
```

### Building for Production

```bash
# Build all projects
dotnet build --configuration Release

# Publish optimized
dotnet publish --configuration Release
```

## Deployment

### Azure Container Apps (Recommended)

```bash
# Enable deploy command
aspire config set features.deployCommandEnabled true

# Authenticate with Azure
az login

# Deploy to Azure
aspire deploy
```

### Kubernetes

```bash
# Generate Kubernetes manifests
aspire publish --output k8s

# Apply to cluster
kubectl apply -f k8s/
```

### Docker Compose

```bash
# Generate docker-compose.yml
aspire publish --output compose

# Run with Docker Compose
docker-compose up
```

## Project Structure

### AppHost Project

The orchestrator that defines how services connect and start:

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

builder.Build().Run();
```

### Service Defaults

Shared configuration for observability and service defaults applied to all projects.

## Documentation

- [Architecture Overview](docs/architecture/OVERVIEW.md)
- [Development Guide](docs/guides/DEVELOPMENT.md)
- [LLM Assistant Instructions](docs/llm/INSTRUCTIONS.md)
- [Project Status](docs/PROJECT_STATUS.md)

## Troubleshooting

### Docker Not Running

```
Error: Cannot connect to Docker daemon
```

Solution: Start Docker Desktop or Podman.

### Port Conflicts

If ports are in use, Aspire will automatically assign different ports. Check the dashboard for actual ports.

### Certificate Trust Issues

```bash
# Re-trust certificates
aspire run
# Follow prompts to trust certificates
```

## Resources

### Official Documentation

- [Aspire Official Site](https://aspire.dev)
- [Microsoft Learn - Aspire Docs](https://learn.microsoft.com/en-us/dotnet/aspire/)
- [Aspire GitHub](https://github.com/dotnet/aspire)

### Getting Help

- [Aspire Discord](https://aka.ms/aspire-discord)
- [GitHub Discussions](https://github.com/dotnet/aspire/discussions)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/.net-aspire)

## Contributing

See [DEVELOPMENT.md](docs/guides/DEVELOPMENT.md) for contribution guidelines.

## License

This project is licensed under the MIT License.

## Release History

See [GitHub Releases](../../releases) for detailed version history, changelogs, and repository snapshots.

---

**Built with .NET Aspire 13** | **Powered by .NET 10.0**
