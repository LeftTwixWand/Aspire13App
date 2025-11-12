# Development Guide - Aspire13App

Complete guide for developers working on the Aspire13App project.

## Table of Contents

1. [Development Setup](#development-setup)
2. [Daily Workflow](#daily-workflow)
3. [Aspire CLI Commands](#aspire-cli-commands)
4. [Adding Features](#adding-features)
5. [Testing](#testing)
6. [Debugging](#debugging)
7. [Code Standards](#code-standards)
8. [Git Workflow](#git-workflow)
9. [Troubleshooting](#troubleshooting)

## Development Setup

### Initial Setup

1. **Install Prerequisites:**
   ```bash
   # Verify .NET SDK
   dotnet --version  # Should be 10.0.100+

   # Verify Docker
   docker --version

   # Install Aspire CLI
   curl -sSL https://aspire.dev/install.sh | bash  # Linux/macOS
   # or
   iex "& { $(irm https://aspire.dev/install.ps1) }"  # Windows

   # Verify Aspire CLI
   aspire --version  # Should be 9.5.2+
   ```

2. **Clone Repository:**
   ```bash
   git clone <repository-url>
   cd Aspire13App
   ```

3. **Restore Dependencies:**
   ```bash
   dotnet restore
   ```

4. **Verify Build:**
   ```bash
   dotnet build
   ```

5. **Run Application:**
   ```bash
   aspire run
   ```

### IDE Configuration

#### Visual Studio 2022

1. Open `Aspire13App.sln`
2. Install Aspire workload if prompted
3. Set `Aspire13App.AppHost` as startup project
4. Press F5 to run

#### Visual Studio Code

1. Install extensions:
   - C# Dev Kit
   - Docker

2. Open folder in VS Code
3. Trust workspace
4. Use integrated terminal: `aspire run`

#### JetBrains Rider

1. Open `Aspire13App.sln`
2. Install Aspire plugin
3. Configure run configuration for AppHost
4. Click Run

## Daily Workflow

### Starting Development

```bash
# 1. Pull latest changes
git pull origin main

# 2. Restore dependencies
dotnet restore

# 3. Start Docker
# Ensure Docker Desktop is running

# 4. Run application
aspire run

# 5. Access dashboard
# Open URL from terminal output (e.g., https://localhost:17178/login?t=...)
```

### During Development

```bash
# Hot reload is enabled - just save files

# Restart if needed
# Ctrl+C to stop
aspire run

# Run tests
dotnet test

# Check code style
dotnet format --verify-no-changes
```

### Ending Development

```bash
# Stop application
# Ctrl+C in terminal

# Commit changes
git add .
git commit -m "feat: add new feature"
git push origin feature-branch
```

## Aspire CLI Commands

### Essential Commands

#### Run Application

```bash
# Start with auto-discovery
aspire run

# Specify AppHost project
aspire run --project ./Aspire13App.AppHost/Aspire13App.AppHost.csproj

# Run with additional args
aspire run -- --environment Staging
```

#### Create Projects

```bash
# Interactive template picker
aspire new

# Create specific template
aspire new aspire-starter --name MyNewApp
```

#### Add Integrations

```bash
# Interactive browser
aspire add

# Search for integration
aspire add postgres

# Add specific package
aspire add Aspire.Hosting.PostgreSQL
```

#### Configuration

```bash
# List all config
aspire config list

# Get specific value
aspire config get features.deployCommandEnabled

# Set value
aspire config set features.execCommandEnabled true

# Delete value
aspire config delete features.execCommandEnabled
```

#### Deployment

```bash
# Generate deployment artifacts
aspire publish --output ./dist

# Deploy to Azure (requires feature flag)
aspire config set features.deployCommandEnabled true
az login
aspire deploy
```

### Advanced Commands

#### Execute in Resource Context

```bash
# Enable feature
aspire config set features.execCommandEnabled true

# Run EF migrations
aspire exec --resource api -- dotnet ef migrations add Initial

# Run custom script
aspire exec --resource api --workdir /app/scripts -- ./setup.sh
```

#### Update Aspire

```bash
# Update CLI
curl -sSL https://aspire.dev/install.sh | bash

# Update project packages (preview)
aspire update
```

## Adding Features

### Adding a New Service

#### 1. Create Service Project

```bash
# Create new project
dotnet new webapi -n Aspire13App.OrderService

# Add to solution
dotnet sln add Aspire13App.OrderService

# Add ServiceDefaults reference
dotnet add Aspire13App.OrderService reference Aspire13App.ServiceDefaults
```

#### 2. Configure Service

Edit `Aspire13App.OrderService/Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add service defaults (required)
builder.AddServiceDefaults();

// Add your services
builder.Services.AddControllers();

var app = builder.Build();

// Map default endpoints (required for health checks)
app.MapDefaultEndpoints();

// Map your endpoints
app.MapControllers();

app.Run();
```

#### 3. Register in AppHost

Edit `Aspire13App.AppHost/AppHost.cs`:

```csharp
var orderService = builder.AddProject<Projects.Aspire13App_OrderService>("orderservice")
    .WithHttpHealthCheck("/health");

// Add to existing services
builder.AddProject<Projects.Aspire13App_Web>("webfrontend")
    .WithReference(orderService)
    .WaitFor(orderService);
```

#### 4. Update Documentation

- [ ] Update `README.md` architecture section
- [ ] Update `docs/architecture/OVERVIEW.md`
- [ ] Update `docs/PROJECT_STATUS.md`
- [ ] Commit changes

### Adding Database Support

#### PostgreSQL Example

```bash
# Add hosting integration
aspire add Aspire.Hosting.PostgreSQL

# Add client integration
dotnet add Aspire13App.ApiService package Aspire.Npgsql.EntityFrameworkCore.PostgreSQL
```

Configure in AppHost:

```csharp
var postgres = builder.AddPostgres("postgres")
    .WithPgAdmin();  // Optional: Add management UI

var database = postgres.AddDatabase("appdb");

var api = builder.AddProject<Projects.Aspire13App_ApiService>("api")
    .WithReference(database)
    .WaitFor(database);
```

Use in service:

```csharp
builder.AddNpgsqlDbContext<AppDbContext>("appdb");

// Use in endpoints
app.MapGet("/orders", async (AppDbContext db) =>
    await db.Orders.ToListAsync());
```

### Adding Message Queue

#### RabbitMQ Example

```bash
# Add integration
aspire add rabbitmq
```

Configure in AppHost:

```csharp
var messaging = builder.AddRabbitMQ("messaging")
    .WithManagementPlugin();

var api = builder.AddProject<Projects.Aspire13App_ApiService>("api")
    .WithReference(messaging);
```

Use in service:

```csharp
builder.AddRabbitMQClient("messaging");

// Publish message
services.AddSingleton<IMessagePublisher, RabbitMQPublisher>();
```

### Adding Azure Services

#### Azure Blob Storage Example

```bash
# Add integration
aspire add Aspire.Hosting.Azure.Storage
```

Configure in AppHost:

```csharp
var storage = builder.AddAzureStorage("storage");

if (builder.ExecutionContext.IsRunMode)
{
    storage.RunAsEmulator();  // Local development
}

var blobs = storage.AddBlobs("blobs");

var api = builder.AddProject<Projects.Aspire13App_ApiService>("api")
    .WithReference(blobs);
```

## Testing

### Unit Tests

```bash
# Create test project
dotnet new xunit -n Aspire13App.Tests
dotnet sln add Aspire13App.Tests

# Add Aspire testing package
dotnet add Aspire13App.Tests package Aspire.Hosting.Testing

# Run tests
dotnet test
```

Example test:

```csharp
public class ApiTests
{
    [Fact]
    public async Task GetWeatherReturnsData()
    {
        var builder = await DistributedApplicationTestingBuilder
            .CreateAsync<Projects.Aspire13App_AppHost>();

        await using var app = await builder.BuildAsync();
        await app.StartAsync();

        var httpClient = app.CreateHttpClient("apiservice");

        var response = await httpClient.GetAsync("/weatherforecast");

        response.EnsureSuccessStatusCode();
    }
}
```

### Integration Tests

```bash
# Run integration tests
dotnet test --filter Category=Integration
```

### Load Testing

Use k6 or Apache JMeter:

```javascript
// k6 example
import http from 'k6/http';

export default function () {
  http.get('https://localhost:7xxx/weatherforecast');
}
```

## Debugging

### Debug in Visual Studio

1. Set breakpoints
2. F5 to start debugging
3. Debug → Attach to Process for running services

### Debug in VS Code

1. Set breakpoints
2. Run → Start Debugging (F5)
3. Select ".NET Core Attach" for running processes

### Debug with Aspire CLI

```bash
# Run with debug output
aspire run --verbosity debug

# Check logs
cat ~/.aspire/cli/logs/apphost-*.log

# Use aspire exec for debugging
aspire config set features.execCommandEnabled true
aspire exec --resource api -- dotnet watch
```

### View Telemetry

1. Open Aspire Dashboard
2. Navigate to:
   - **Resources**: Service status and health
   - **Logs**: Structured logs from all services
   - **Traces**: Distributed tracing
   - **Metrics**: Performance metrics

### Common Debug Scenarios

#### Service Not Starting

```bash
# Check service logs
# In Aspire Dashboard → Resources → [Service] → View Logs

# Check health endpoint
curl https://localhost:<port>/health

# Verify dependencies
# In AppHost.cs, check WaitFor() calls
```

#### Service Discovery Failing

```csharp
// Verify service name matches
// AppHost:
var api = builder.AddProject<Projects.Api>("apiservice");

// Consuming service:
builder.Services.AddHttpClient<Client>(c =>
    c.BaseAddress = new("https+http://apiservice"));  // Must match
```

#### Database Connection Issues

```bash
# Check connection string
aspire exec --resource api -- dotnet user-secrets list

# Test connection
aspire exec --resource api -- dotnet ef database update
```

## Code Standards

### C# Coding Conventions

- Follow [Microsoft C# Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- Use .editorconfig (included in project)
- Run formatter before committing:

```bash
dotnet format
```

### File Organization

```
Aspire13App.ServiceName/
├── Controllers/          # API controllers
├── Models/              # Data models
├── Services/            # Business logic
├── Program.cs           # Startup configuration
└── appsettings.json     # Configuration
```

### Naming Conventions

- **Classes**: PascalCase (e.g., `OrderService`)
- **Methods**: PascalCase (e.g., `GetOrders`)
- **Variables**: camelCase (e.g., `orderList`)
- **Constants**: PascalCase (e.g., `MaxRetries`)
- **Private fields**: _camelCase (e.g., `_dbContext`)

### Aspire Conventions

- Service names: lowercase (e.g., `"apiservice"`)
- Resource names: lowercase (e.g., `"cache"`, `"postgres"`)
- Always use `AddServiceDefaults()`
- Always use `MapDefaultEndpoints()`

## Git Workflow

### Branch Strategy

```
main
├── develop
    ├── feature/add-orders-api
    ├── feature/add-authentication
    └── bugfix/fix-caching-issue
```

### Creating Feature Branch

```bash
git checkout develop
git pull origin develop
git checkout -b feature/add-orders-api
```

### Committing Changes

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```bash
git add .
git commit -m "feat: add orders API endpoint"
git commit -m "fix: resolve caching timeout issue"
git commit -m "docs: update architecture documentation"
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance

### Pull Request Process

1. **Create PR** on GitHub/Azure DevOps
2. **Fill template** (description, testing notes)
3. **Request review** from team
4. **Address feedback**
5. **Merge** after approval

## Troubleshooting

### Build Errors

#### "Cannot find project reference"

```bash
# Verify project exists
dotnet sln list

# Add if missing
dotnet sln add Aspire13App.NewService
```

#### "Package version conflict"

```bash
# Clear NuGet cache
dotnet nuget locals all --clear

# Restore
dotnet restore
```

### Runtime Errors

#### "Docker is not available"

**Solution:**
```bash
# Start Docker Desktop
# Verify: docker ps

# Restart Aspire
aspire run
```

#### "Port already in use"

**Solution:** Aspire assigns ports dynamically. Check dashboard for actual ports.

#### "Service discovery timeout"

**Solution:**
```csharp
// Increase timeout
builder.Services.AddHttpClient<Client>(c =>
    c.BaseAddress = new("https+http://api"))
    .ConfigureHttpClient(c => c.Timeout = TimeSpan.FromSeconds(30));
```

### Performance Issues

#### Slow startup

```bash
# Check Docker resources
# Docker Desktop → Settings → Resources

# Increase limits if needed
```

#### High memory usage

```bash
# Profile application
dotnet-trace collect --process-id <PID>

# Analyze with PerfView or dotnet-trace
```

## Best Practices

### Development

- ✅ Use `aspire run` for local development
- ✅ Enable hot reload for faster iteration
- ✅ Write tests for new features
- ✅ Update documentation when changing architecture
- ✅ Use service discovery, not hardcoded URLs
- ✅ Add health checks to all services
- ✅ Use structured logging

### Performance

- ✅ Use caching for expensive operations
- ✅ Implement async/await properly
- ✅ Use connection pooling
- ✅ Monitor with OpenTelemetry
- ✅ Set appropriate timeouts

### Security

- ✅ Use parameters for secrets
- ✅ Never commit secrets to git
- ✅ Use HTTPS in production
- ✅ Validate input data
- ✅ Implement authentication/authorization

## Learning Resources

### Official Documentation

- [Aspire Docs](https://learn.microsoft.com/en-us/dotnet/aspire/)
- [Aspire GitHub](https://github.com/dotnet/aspire)
- [Aspire Samples](https://github.com/dotnet/aspire-samples)

### Video Tutorials

- [Aspire YouTube Channel](https://www.youtube.com/@dotnet)
- [.NET Community Standup](https://www.youtube.com/playlist?list=PLdo4fOcmZ0oX-DBuRG4u58ZTAJgBAeQ-t)

### Community

- [Aspire Discord](https://aka.ms/aspire-discord)
- [GitHub Discussions](https://github.com/dotnet/aspire/discussions)

---

**Last Updated:** 2025-11-12
**Maintained By:** Development Team
