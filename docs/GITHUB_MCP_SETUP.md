# GitHub MCP Setup Instructions

This document contains the complete setup instructions for GitHub MCP integration with this project.

**Status:** Configuration files created, waiting for Claude Code restart

**Your Token:** Already securely stored in `.claude/mcp-config.json` (gitignored)

---

## What Was Configured

✅ **Project Configuration:**
- Added `.claude/` to `.gitignore` (keeps tokens secure)
- Created `.claude/mcp-config.json` with GitHub MCP settings
- Updated `.claude/settings.local.json` to allow GitHub MCP tools
- GitHub Personal Access Token securely stored

✅ **Permissions Added:**
- `mcp__github__*` - All GitHub MCP operations
- `Bash(gh:*)` - GitHub CLI commands
- `Bash(npm:*)` and `Bash(npx:*)` - For running GitHub MCP server

---

## Next Steps: Activate GitHub MCP

GitHub MCP needs to be configured in your **global** Claude Code settings. Here's how:

### Option 1: Manual Configuration (Recommended)

1. **Locate your Claude Code config file:**

   **Windows:**
   ```
   %USERPROFILE%\.claude\config.json
   ```

   **macOS/Linux:**
   ```
   ~/.claude/config.json
   ```

2. **Open the file** in a text editor

3. **Add GitHub MCP server** to the `mcpServers` section:

   **IMPORTANT:** Your token is already stored in `.claude/mcp-config.json`.
   You can either:

   **A) Copy from project config:**
   ```bash
   # View your token (in project directory)
   cat .claude/mcp-config.json
   ```

   **B) Use this template** (replace `<YOUR_TOKEN>` with your actual token):
   ```json
   {
     "mcpServers": {
       "github": {
         "command": "npx",
         "args": ["-y", "@modelcontextprotocol/server-github"],
         "env": {
           "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_GITHUB_TOKEN_HERE>"
         }
       },
       "microsoft-learn": {
         ... existing config ...
       }
     }
   }
   ```

4. **Save the file**

5. **Restart Claude Code**

### Option 2: Using Claude Code UI (If Available)

1. Open Claude Code Settings
2. Navigate to MCP Servers section
3. Click "Add Server"
4. Enter:
   - **Name:** github
   - **Command:** npx
   - **Args:** -y, @modelcontextprotocol/server-github
   - **Environment Variables:**
     - Key: `GITHUB_PERSONAL_ACCESS_TOKEN`
     - Value: `<YOUR_GITHUB_TOKEN>` (get from `.claude/mcp-config.json`)
5. Save and restart

---

## Verification

After restart, verify GitHub MCP is working:

1. **Check available MCP servers:**
   - I should be able to see GitHub MCP tools
   - Tools like: create_release, create_issue, get_repository, etc.

2. **Test with a simple query:**
   Ask me: "List the recent commits in this repository"

---

## GitHub MCP Capabilities

Once configured, I can:

### Repository Operations
- `get_repository` - Get repository details
- `list_commits` - List commit history
- `get_file_contents` - Read files from repo
- `search_repositories` - Search GitHub repos
- `list_issues` - List repository issues
- `create_or_update_file` - Create/update files

### Issue Management
- `create_issue` - Create new issues
- `get_issue` - Get issue details
- `update_issue` - Update issue status/labels
- `list_issues` - Query issues

### Pull Request Operations
- `create_pull_request` - Create PRs
- `get_pull_request` - Get PR details
- `list_pull_requests` - List PRs

### Release Management
- `create_release` - **Create GitHub releases with markdown**
- `get_release` - Get release details
- `list_releases` - List all releases

### Other Operations
- `fork_repository` - Fork repos
- `create_branch` - Create branches
- `search_code` - Search code in repo
- `search_issues` - Search issues

---

## Security Notes

### Token Security
- ✅ Token stored in global config (not in repository)
- ✅ `.claude/` added to `.gitignore`
- ✅ Project MCP config excluded from git
- ✅ Token has appropriate scopes (repo, workflow)

### Token Scopes
Your token has these permissions:
- ✅ `repo` - Full repository access
- ✅ `workflow` - GitHub Actions access
- ✅ `read:org` - Organization read access

### Token Management
- **Expiration:** Check your GitHub settings
- **Rotation:** Generate new token when expired
- **Revocation:** Revoke from GitHub settings if compromised

---

## Troubleshooting

### "GitHub MCP not found"

**Solution:**
- Ensure you added config to **global** config file
- Restart Claude Code completely
- Verify npx is available: `npx --version`

### "Authentication failed"

**Solution:**
- Verify token is correct in config
- Check token hasn't expired
- Ensure token has required scopes

### "Command not found: npx"

**Solution:**
Install Node.js:
- Windows: https://nodejs.org/
- macOS: `brew install node`
- Linux: `sudo apt install nodejs npm`

### "Permission denied"

**Solution:**
- Check `.claude/settings.local.json` has `mcp__github__*` permission
- Restart Claude Code

---

## Using GitHub MCP

### Example: Create a Release

After configuration, you can ask me:

```
Create a GitHub release for v1.0.0 using the content from RELEASE_v1.0.0.md
```

And I'll use the GitHub MCP to:
1. Read the release notes file
2. Create the release on GitHub
3. Add all the markdown with collapsible sections
4. Tag the release properly

### Example: Query Repository

```
Show me the last 5 commits to this repository
```

### Example: Create an Issue

```
Create an issue to track the PostgreSQL integration feature
```

---

## Configuration Files Reference

### `.claude/mcp-config.json`
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_GITHUB_TOKEN>"
      }
    }
  }
}
```

### `.claude/settings.local.json` (excerpt)
```json
{
  "permissions": {
    "allow": [
      "mcp__github__*",
      "Bash(gh:*)",
      "Bash(npm:*)",
      "Bash(npx:*)",
      ...
    ]
  }
}
```

---

## Next Actions

**Immediate:**
1. ✅ Configuration files created
2. ⏸️ **YOU:** Add GitHub MCP to global Claude config
3. ⏸️ **YOU:** Restart Claude Code
4. ⏸️ **ME:** Test GitHub MCP and create v1.0.0 release

**After Setup:**
- Create v1.0.0 GitHub release with full markdown
- Test other GitHub MCP features
- Document workflow in project docs

---

## Questions?

If you encounter issues:
1. Check this document's troubleshooting section
2. Verify all prerequisites are installed
3. Check Claude Code logs for errors
4. Ask me for help after restart

---

**Created:** 2025-11-12
**Token Stored:** Securely in .claude/mcp-config.json (gitignored)
**Status:** Ready for global configuration
