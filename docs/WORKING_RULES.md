# Working Rules - Aspire13App

This document contains mandatory rules and principles for LLM assistants working with this project. These rules must be followed consistently.

**Last Updated:** 2025-11-12
**Status:** Active

---

## Critical Rules

### 1. Version Control & History

#### ❌ DO NOT:
- Store version history in README.md
- Maintain changelogs in documentation files
- Track releases in project documentation

#### ✅ DO:
- Use GitHub releases for all version history
- Include detailed release notes in GitHub releases with:
  - **Collapsible sections** for organization
  - **Repository tree structure** at release moment
  - **Full changelog** with all changes
  - **Breaking changes** clearly marked
  - **Migration guides** when applicable
  - **Known issues** for that release
- Direct users to GitHub releases in README
- Use semantic versioning (e.g., v1.0.0)

**Example Release Structure:**
```markdown
## v1.0.0 - Release Name

<details>
<summary>📋 Full Changelog</summary>

### Added
- Feature 1
- Feature 2

### Changed
- Update 1

### Fixed
- Bug fix 1

</details>

<details>
<summary>🌳 Repository Structure</summary>

\`\`\`
Project/
├── src/
├── docs/
└── tests/
\`\`\`

</details>

<details>
<summary>⚠️ Breaking Changes</summary>

- Breaking change 1
- Migration: Steps to migrate

</details>
```

### 2. MCP Server Integration

#### Always Check for MCP Servers

Before implementing integration with any service or tool, **ALWAYS**:

1. **Check if MCP server exists** for that service
2. **Evaluate usefulness** for the project
3. **Ask user for permission** to add MCP to configuration
4. **Document MCP servers** in use

#### Current MCP Servers

**Active:**
- ✅ `microsoft-learn` - Microsoft/Azure documentation access
  - Tools: `microsoft_docs_search`, `microsoft_docs_fetch`, `microsoft_code_sample_search`
  - Use Case: Aspire documentation, Azure services, .NET APIs

**Available But Not Configured:**
- ⏸️ GitHub MCP - Repository management, issues, PRs
  - Would enable: Creating releases, managing issues, PR operations
  - **Recommendation:** Should be added for this project

#### When to Suggest MCP Addition

Ask user: "I notice we're working with [SERVICE]. There's an MCP server available that would let me [CAPABILITIES]. Would you like me to add it to the Claude Code configuration?"

**Examples:**
- GitHub operations → Suggest GitHub MCP
- Database work → Suggest database-specific MCP
- Cloud deployment → Suggest cloud provider MCP

### 3. Aspire CLI First

#### ❌ NEVER:
- Use `dotnet run` unless explicitly requested
- Use `dotnet new aspire-*` templates directly
- Manually add NuGet packages for Aspire integrations
- Skip Aspire CLI for standard operations

#### ✅ ALWAYS:
- Use `aspire run` to start applications
- Use `aspire new` for project creation
- Use `aspire add` for integrations
- Use `aspire config` for configuration
- Use `aspire deploy` for deployment (when feature enabled)

### 4. Service Discovery Over Hardcoding

#### ❌ NEVER:
- Hardcode URLs: `https://localhost:5001`
- Hardcode ports: `http://localhost:8080`
- Use IP addresses: `http://192.168.1.100`

#### ✅ ALWAYS:
- Use service names: `https+http://apiservice`
- Let Aspire assign ports dynamically
- Use service discovery for all inter-service communication

### 5. Documentation Maintenance

#### Update Documentation When:

1. **Adding/Removing Services:**
   - Update `README.md` architecture section
   - Update `docs/architecture/OVERVIEW.md`
   - Update `docs/PROJECT_STATUS.md`

2. **Changing Architecture:**
   - Update `docs/architecture/OVERVIEW.md`
   - Add Architecture Decision Record (ADR)
   - Update diagrams if applicable

3. **Adding Features:**
   - Update `docs/PROJECT_STATUS.md` (move from planned to completed)
   - Update relevant guides in `docs/guides/`

4. **Changing Workflows:**
   - Update `docs/guides/DEVELOPMENT.md`
   - Add examples if complex

5. **Creating Releases:**
   - Create GitHub release with full details
   - Update `docs/PROJECT_STATUS.md` with release number
   - Tag commit appropriately

#### Documentation Standards

**File Headers:**
```markdown
# Document Title

Brief description.

**Last Updated:** YYYY-MM-DD
**Status:** Active/Draft/Deprecated
```

**Code Blocks:**
- Always specify language
- Include comments for complex code
- Show both correct and incorrect examples when relevant

**Links:**
- Use relative paths for internal docs
- Full URLs for external resources
- Check links work before committing

### 6. Health Checks Are Mandatory

#### ✅ ALWAYS:
- Add health checks to new services:
  ```csharp
  builder.AddProject<Projects.NewService>("newservice")
      .WithHttpHealthCheck("/health");
  ```
- Implement `/health` endpoint in services
- Monitor health in Aspire Dashboard

### 7. Dependency Management

#### ✅ ALWAYS:
- Use `.WaitFor()` for service dependencies:
  ```csharp
  var api = builder.AddProject<Projects.Api>("api")
      .WaitFor(database)
      .WaitFor(cache);
  ```
- Specify startup order explicitly
- Document dependencies in architecture docs

### 8. Security Best Practices

#### ❌ NEVER:
- Commit secrets to git
- Hardcode API keys
- Store passwords in code
- Skip authentication in production

#### ✅ ALWAYS:
- Use parameter resources for secrets:
  ```csharp
  var apiKey = builder.AddParameter("api-key", secret: true);
  ```
- Use Azure Key Vault for production
- Validate all inputs
- Use HTTPS in production

### 9. Observability

#### ✅ ALWAYS:
- Add `builder.AddServiceDefaults()` to all services
- Call `app.MapDefaultEndpoints()` in all services
- Use structured logging
- Include trace context in logs
- Monitor via Aspire Dashboard

### 10. Testing

#### ✅ ALWAYS:
- Write tests for new features
- Update tests when changing behavior
- Run tests before committing: `dotnet test`
- Include integration tests for service interactions

### 11. Code Quality

#### ✅ ALWAYS:
- Run formatter before committing: `dotnet format`
- Follow .editorconfig settings
- Use meaningful variable names
- Add comments for complex logic
- Keep methods focused and small

### 12. Git Workflow

#### Commit Messages:
Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <description>

[optional body]

[optional footer]
```

**Types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `refactor:` - Code restructuring
- `test:` - Adding/updating tests
- `chore:` - Maintenance tasks

**Examples:**
```bash
git commit -m "feat: add PostgreSQL integration with EF Core"
git commit -m "fix: resolve Redis connection timeout"
git commit -m "docs: update architecture diagram"
```

#### Branch Strategy:
```
main/master
├── develop
│   ├── feature/feature-name
│   ├── bugfix/bug-name
│   └── chore/task-name
└── hotfix/critical-fix
```

### 13. Release Process

#### When Creating Release:

1. **Prepare Release:**
   ```bash
   # Update version in project files
   # Run all tests
   dotnet test
   # Build release
   dotnet build --configuration Release
   ```

2. **Create Git Tag:**
   ```bash
   git tag -a v1.0.0 -m "Release v1.0.0: Description"
   git push origin v1.0.0
   ```

3. **Create GitHub Release:**
   - Use GitHub CLI or MCP
   - Include all collapsible sections
   - Add tree structure
   - List all changes
   - Mark breaking changes
   - Add migration guide if needed

4. **Update Documentation:**
   - Update `docs/PROJECT_STATUS.md` with new version
   - Update README if needed
   - Archive old documentation if major version

### 14. Proactive MCP Suggestions

#### Check MCP Availability For:

**Services:**
- GitHub (repos, issues, PRs) → GitHub MCP
- Azure (resources, deployments) → Azure MCP
- AWS (services, deployment) → AWS MCP
- PostgreSQL/MySQL → Database MCPs
- Redis → Redis MCP
- Docker → Docker MCP

**Workflows:**
- CI/CD → GitHub Actions MCP
- Monitoring → Observability MCPs
- Documentation → Docs platform MCPs

#### Suggestion Template:

```
🔧 MCP Server Suggestion

I notice we're working with [SERVICE/TOOL].

There's an MCP server available: [MCP_NAME]

Capabilities:
- [Feature 1]
- [Feature 2]
- [Feature 3]

Benefits for this project:
- [Benefit 1]
- [Benefit 2]

Would you like me to add this MCP to the Claude Code configuration?
```

### 15. Performance Considerations

#### ✅ ALWAYS:
- Use caching for expensive operations
- Implement async/await properly
- Use connection pooling
- Set appropriate timeouts
- Monitor performance metrics
- Profile before optimizing

### 16. Error Handling

#### ✅ ALWAYS:
- Use try-catch for expected failures
- Log errors with context
- Return appropriate HTTP status codes
- Include error details in development
- Use generic messages in production
- Implement retry logic for transient failures

## Enforcement

These rules are **MANDATORY** for all LLM assistants working on this project.

### Before Any Change:
1. ✅ Check if rules apply
2. ✅ Review relevant documentation
3. ✅ Verify no hardcoded values
4. ✅ Ensure documentation will be updated
5. ✅ Plan commit message

### After Any Change:
1. ✅ Update documentation
2. ✅ Run tests
3. ✅ Format code
4. ✅ Commit with proper message
5. ✅ Verify build succeeds

## Rule Violations

If a rule must be broken:
1. **Document why** in code comments
2. **Note in commit message**
3. **Add to PROJECT_STATUS.md** known issues
4. **Plan remediation** with timeline

## Adding New Rules

When adding rules:
1. Update this document
2. Update `docs/llm/INSTRUCTIONS.md` if relevant
3. Commit with `docs:` prefix
4. Announce to team

## Questions?

If rules conflict or are unclear:
1. Check `docs/llm/INSTRUCTIONS.md` for details
2. Review official Aspire docs
3. Ask for clarification
4. Document decision as ADR

---

**Rule Owner:** Development Team
**Review Frequency:** With each major release
**Last Reviewed:** 2025-11-12
