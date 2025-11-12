# MCP Server Recommendations - Aspire13App

This document tracks MCP (Model Context Protocol) servers that are useful for this project.

**Last Updated:** 2025-11-12

---

## Currently Configured MCP Servers

### ✅ Microsoft Learn MCP

**Status:** Active
**Server:** `microsoft-learn`

**Available Tools:**
1. `microsoft_docs_search` - Search official Microsoft/Azure documentation
2. `microsoft_docs_fetch` - Fetch complete documentation pages
3. `microsoft_code_sample_search` - Search for code samples in Microsoft docs

**Use Cases:**
- Researching .NET Aspire features and best practices
- Finding Azure service documentation
- Getting official code samples for integrations
- Staying current with .NET 10 APIs

### ✅ GitHub MCP

**Status:** Configured (Pending Restart)
**Server:** `github`
**Setup:** See [GITHUB_MCP_SETUP.md](GITHUB_MCP_SETUP.md) for activation

**Available Tools:**
1. `create_release` - Create GitHub releases
2. `get_repository` - Get repository information
3. `list_commits` - List commit history
4. `create_issue` - Create issues
5. `create_pull_request` - Create PRs
6. `get_file_contents` - Read repository files
7. `search_code` - Search code in repositories

**Use Cases:**
- Creating releases with rich markdown
- Managing issues and pull requests
- Querying commit history
- Searching repository code
- Automating repository operations

---

## Recommended MCP Servers

### 🔧 GitHub MCP (Highly Recommended)

**Status:** ✅ Configured (Pending Claude Code Restart)
**Priority:** **HIGH**

**Why We Need It:**
This project is hosted on GitHub and we need to:
- Create GitHub releases with detailed markdown
- Manage issues and track bugs
- Handle pull requests
- Tag releases automatically
- Query commit history
- Manage repository settings

**Capabilities:**
- Create and manage releases
- Create issues from code analysis
- Fetch commit history
- Manage pull requests
- Query repository statistics
- Handle GitHub Actions workflows

**Installation:**
GitHub MCP servers are available through the Claude Code MCP ecosystem.

**Configuration Example:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-token>"
      }
    }
  }
}
```

**Benefits for This Project:**
1. **Automated Releases:**
   - Create releases with collapsible sections
   - Include repository tree structure
   - Add detailed changelogs

2. **Issue Management:**
   - Track bugs discovered during development
   - Create feature requests
   - Link commits to issues

3. **PR Automation:**
   - Create pull requests from feature branches
   - Auto-fill PR descriptions
   - Link related issues

4. **Repository Insights:**
   - Query commit history
   - Analyze repository structure
   - Track contributions

### 🔧 Azure MCP (Recommended for Future)

**Status:** ⚠️ Not Configured
**Priority:** Medium (needed when deploying to Azure)

**Why We Need It:**
When deploying to Azure Container Apps, we'll need to:
- Manage Azure resources
- Query deployment status
- Monitor application health
- Configure Azure services

**Capabilities:**
- Create and manage Azure resources
- Deploy applications
- Query resource status
- Configure App Configuration
- Manage Key Vault secrets

**When to Add:**
- Before first Azure deployment
- When setting up staging environment
- For production deployments

### 🔧 Docker MCP (Optional)

**Status:** ⚠️ Not Configured
**Priority:** Low (Aspire handles most Docker operations)

**Why It Could Help:**
- Query container status outside Aspire
- Manage Docker networks
- Inspect container logs
- Handle Docker Compose files

**Note:** Aspire CLI handles most Docker operations, so this is low priority.

### 🔧 PostgreSQL MCP (If Database Added)

**Status:** ⚠️ Not Configured
**Priority:** Medium (if PostgreSQL is added)

**Why We'd Need It:**
If PostgreSQL is added to the project:
- Query database schema
- Run migrations
- Inspect data
- Performance analysis

**When to Add:**
- When PostgreSQL integration is added
- Before implementing database features

---

## MCP Server Evaluation Checklist

Before adding any MCP server, verify:

- [ ] **Usefulness:** Does it provide capabilities we need?
- [ ] **Security:** Can it access sensitive data? How is it secured?
- [ ] **Maintenance:** Is it actively maintained?
- [ ] **Documentation:** Is it well-documented?
- [ ] **Configuration:** How complex is setup?
- [ ] **Performance:** Will it slow down Claude Code?

---

## How to Add MCP Servers

### Option 1: Global Configuration

Edit `~/.claude/config.json`:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "command",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### Option 2: Project-Specific Configuration

Create `.claude/mcp-settings.json` in project:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
  }
}
```

### Option 3: Via Claude Code Settings

1. Open Claude Code
2. Settings → MCP Servers
3. Add server configuration
4. Restart Claude Code

---

## Current Recommendations Summary

### Immediate Action Needed:

**🔧 Add GitHub MCP**
- **Why:** Managing releases, issues, PRs for this project
- **Priority:** HIGH
- **Effort:** 10 minutes (generate token + configure)
- **Benefit:** Automate release creation with tree structure and markdown

### Future Additions:

**🔧 Azure MCP** - When deploying to cloud
**🔧 PostgreSQL MCP** - If database added
**🔧 RabbitMQ MCP** - If message queue added

---

## Questions About MCP Servers?

**Where to find MCP servers:**
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- npm packages: `@modelcontextprotocol/server-*`

**How to create custom MCP:**
- Follow MCP specification
- Use TypeScript SDK
- Document capabilities
- Share with community

---

**Document Owner:** Development Team
**Review Frequency:** Monthly or when adding new integrations
**Last Reviewed:** 2025-11-12
