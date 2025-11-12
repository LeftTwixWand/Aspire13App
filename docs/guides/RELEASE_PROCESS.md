# Release Process - Aspire13App

Complete guide for creating releases with detailed GitHub release notes.

**Last Updated:** 2025-11-12

---

## Release Philosophy

**GitHub is the source of truth** for version history:
- ❌ No version history in README.md
- ❌ No changelogs in documentation files
- ✅ All releases on GitHub with rich markdown
- ✅ Collapsible sections for organization
- ✅ Repository snapshots for each release

---

## Semantic Versioning

We follow [Semantic Versioning 2.0.0](https://semver.org/):

```
MAJOR.MINOR.PATCH

Example: v2.3.1
```

**Version Components:**
- **MAJOR:** Breaking changes, incompatible API changes
- **MINOR:** New features, backwards-compatible additions
- **PATCH:** Bug fixes, backwards-compatible patches

**Pre-release Labels:**
- `v1.0.0-alpha.1` - Alpha release
- `v1.0.0-beta.2` - Beta release
- `v1.0.0-rc.1` - Release candidate

---

## Release Checklist

### Phase 1: Preparation

- [ ] All planned features completed
- [ ] All tests passing: `dotnet test`
- [ ] Code formatted: `dotnet format`
- [ ] Documentation updated
- [ ] `docs/PROJECT_STATUS.md` reflects current state
- [ ] No known critical bugs

### Phase 2: Version Update

- [ ] Update version numbers in:
  - Project files (if using `<Version>` property)
  - `docs/PROJECT_STATUS.md`
  - Any version constants

### Phase 3: Testing

- [ ] Build release configuration:
  ```bash
  dotnet build --configuration Release
  ```
- [ ] Run all tests:
  ```bash
  dotnet test --configuration Release
  ```
- [ ] Test locally with `aspire run`
- [ ] Smoke test all major features
- [ ] Verify no errors in logs

### Phase 4: Git Tagging

- [ ] Commit all changes
- [ ] Create annotated tag:
  ```bash
  git tag -a v1.0.0 -m "Release v1.0.0: Feature Name"
  ```
- [ ] Push tag:
  ```bash
  git push origin v1.0.0
  ```

### Phase 5: GitHub Release

- [ ] Create GitHub release (use template below)
- [ ] Include all sections
- [ ] Add repository tree structure
- [ ] Mark as pre-release if applicable
- [ ] Publish release

### Phase 6: Post-Release

- [ ] Update `docs/PROJECT_STATUS.md` with release info
- [ ] Announce release (if applicable)
- [ ] Monitor for issues
- [ ] Plan next release

---

## GitHub Release Template

Copy this template for each release:

````markdown
# v1.0.0 - Release Title

Brief description of this release (2-3 sentences).

---

<details>
<summary>📋 Full Changelog</summary>

## Added
- **Feature Name** - Description of what was added
  - Details about the feature
  - Related files: `path/to/file.cs`
- **Another Feature** - Description

## Changed
- **Updated Component** - What changed and why
  - Migration notes if applicable
  - Impact on existing functionality

## Fixed
- **Bug Description** - What was fixed
  - Issue: #123
  - Affected: Component X

## Removed
- **Deprecated Feature** - What was removed
  - Reason for removal
  - Alternative: Use X instead

## Security
- **Security Fix** - Description of security improvement
  - Severity: Critical/High/Medium/Low
  - Affected versions: v0.x.x - v0.9.9

</details>

<details>
<summary>⚠️ Breaking Changes</summary>

## Breaking Change 1: Title

**What Changed:**
Description of the breaking change.

**Migration Guide:**
```csharp
// Before
OldCode();

// After
NewCode();
```

**Affected:**
- Component X
- API endpoint Y

---

## Breaking Change 2: Title

**What Changed:**
...

</details>

<details>
<summary>🌳 Repository Structure</summary>

```
Aspire13App/
├── Aspire13App.AppHost/           # Orchestration host
│   ├── AppHost.cs                 # Main configuration
│   └── Aspire13App.AppHost.csproj
├── Aspire13App.Web/               # Blazor frontend
│   ├── Components/
│   ├── Program.cs
│   └── Aspire13App.Web.csproj
├── Aspire13App.ApiService/        # REST API backend
│   ├── Program.cs
│   └── Aspire13App.ApiService.csproj
├── Aspire13App.ServiceDefaults/   # Shared configurations
│   ├── Extensions.cs
│   └── Aspire13App.ServiceDefaults.csproj
├── docs/                          # Documentation
│   ├── architecture/
│   │   └── OVERVIEW.md
│   ├── guides/
│   │   ├── DEVELOPMENT.md
│   │   └── RELEASE_PROCESS.md
│   ├── llm/
│   │   └── INSTRUCTIONS.md
│   ├── MCP_RECOMMENDATIONS.md
│   ├── PROJECT_STATUS.md
│   └── WORKING_RULES.md
├── README.md
└── Aspire13App.sln
```

**Statistics:**
- Total Projects: 4
- Total Files: ~X
- Lines of Code: ~X
- Test Coverage: X%

</details>

<details>
<summary>📦 Dependencies</summary>

## NuGet Packages

### AppHost
- `Aspire.AppHost.Sdk` v13.0.0
- `Aspire.Hosting.Redis` v13.0.0

### Frontend (Web)
- Microsoft.AspNetCore.Components.* (various)
- Redis output cache integration

### Backend (ApiService)
- Microsoft.AspNetCore.OpenApi

### ServiceDefaults
- Microsoft.Extensions.* (various)
- OpenTelemetry packages

## Runtime Requirements
- .NET SDK: 10.0.100 (or .NET 8.0/9.0)
- Docker Desktop or Podman
- Aspire CLI: 9.5.2+

</details>

<details>
<summary>🚀 New Features</summary>

## Feature 1: Title

**Description:**
Detailed description of the feature.

**Usage:**
```csharp
// Example code showing how to use the feature
var example = new Feature();
example.DoSomething();
```

**Benefits:**
- Benefit 1
- Benefit 2

**Documentation:** [Link to docs](path/to/docs)

---

## Feature 2: Title

...

</details>

<details>
<summary>🐛 Bug Fixes</summary>

## Fixed: Bug Description

**Issue:** #123
**Severity:** Critical/High/Medium/Low

**Problem:**
Description of what was broken.

**Solution:**
How it was fixed.

**Files Changed:**
- `path/to/file1.cs`
- `path/to/file2.cs`

---

</details>

<details>
<summary>📚 Documentation</summary>

## Documentation Updates

- ✅ Updated architecture diagrams
- ✅ Added new API documentation
- ✅ Updated getting started guide
- ✅ Added troubleshooting section

## New Documentation
- Guide: [Title](link)
- Tutorial: [Title](link)

</details>

<details>
<summary>⚙️ Configuration Changes</summary>

## New Configuration Options

**AppHost:**
```csharp
// New configuration example
builder.AddNewFeature("name")
    .WithOption(value);
```

**Environment Variables:**
- `NEW_VARIABLE` - Description
- `UPDATED_VARIABLE` - What changed

</details>

<details>
<summary>🔒 Security</summary>

## Security Improvements

- Added authentication support
- Improved input validation
- Updated dependencies for security patches

## CVEs Addressed
- CVE-2024-XXXXX - Description

</details>

<details>
<summary>⚡ Performance</summary>

## Performance Improvements

- **API Response Time:** 120ms → 45ms (62% faster)
- **Startup Time:** 5s → 3s (40% faster)
- **Memory Usage:** -15% reduction

## Optimizations
- Implemented caching for X
- Optimized database queries
- Reduced allocations in Y

</details>

<details>
<summary>🧪 Testing</summary>

## Test Coverage

- **Total Tests:** X
- **Coverage:** X%
- **New Tests:** X

## Test Improvements
- Added integration tests for X
- Improved test performance
- Added load tests

</details>

<details>
<summary>⚠️ Known Issues</summary>

## Known Limitations

1. **Issue Title**
   - Description
   - Workaround: ...
   - Tracking: #XXX

2. **Another Issue**
   - Description
   - Expected fix: vX.X.X

</details>

<details>
<summary>🔄 Migration Guide</summary>

## Upgrading from v0.9.x to v1.0.0

### Step 1: Update Dependencies

```bash
dotnet restore
aspire update
```

### Step 2: Update Code

**Change 1:**
```csharp
// Before
OldAPI.DoSomething();

// After
NewAPI.DoSomething();
```

### Step 3: Update Configuration

Update `AppHost.cs`:
```csharp
// Add new configuration
builder.AddNewFeature();
```

### Step 4: Test

```bash
aspire run
# Verify all services start
# Test functionality
```

</details>

<details>
<summary>👥 Contributors</summary>

## Contributors to This Release

- @username1 - Feature X, Bug fix Y
- @username2 - Documentation updates
- @username3 - Testing improvements

**Special Thanks:**
Community members who reported issues and provided feedback.

</details>

---

## Installation

```bash
git clone https://github.com/YourOrg/Aspire13App.git
cd Aspire13App
git checkout v1.0.0
dotnet restore
aspire run
```

## Quick Start

See [Getting Started Guide](../../README.md) for detailed instructions.

## Support

- Documentation: [Link to docs](https://...)
- Issues: [GitHub Issues](../../issues)
- Discussions: [GitHub Discussions](../../discussions)

---

**Full Changelog:** [v0.9.0...v1.0.0](../../compare/v0.9.0...v1.0.0)
````

---

## Creating Release with GitHub CLI

If you have GitHub MCP or CLI configured:

```bash
# Create release with markdown file
gh release create v1.0.0 \
  --title "v1.0.0 - Release Title" \
  --notes-file RELEASE_NOTES.md \
  --draft  # Remove --draft when ready

# Or create directly
gh release create v1.0.0 \
  --title "v1.0.0 - Release Title" \
  --notes "Release notes here"
```

---

## Automated Release Script

Create `.github/scripts/create-release.sh`:

```bash
#!/bin/bash

VERSION=$1
TITLE=$2

if [ -z "$VERSION" ] || [ -z "$TITLE" ]; then
    echo "Usage: ./create-release.sh v1.0.0 'Release Title'"
    exit 1
fi

# Generate tree structure
echo "Generating repository structure..."
tree -L 3 -I 'bin|obj|node_modules' > /tmp/tree.txt

# Create release notes
cat > /tmp/release-notes.md << EOF
# $VERSION - $TITLE

<details>
<summary>🌳 Repository Structure</summary>

\`\`\`
$(cat /tmp/tree.txt)
\`\`\`

</details>

<!-- Add more sections -->

EOF

# Create release
gh release create $VERSION \
  --title "$VERSION - $TITLE" \
  --notes-file /tmp/release-notes.md \
  --draft

echo "Release draft created: $VERSION"
echo "Edit on GitHub and publish when ready"
```

Usage:
```bash
chmod +x .github/scripts/create-release.sh
./.github/scripts/create-release.sh v1.0.0 "Initial Release"
```

---

## Release Frequency

**Recommended Schedule:**

- **Major (X.0.0):** Yearly or when breaking changes accumulated
- **Minor (x.X.0):** Monthly or when features complete
- **Patch (x.x.X):** As needed for bug fixes

**Emergency Releases:**
- Security fixes: Immediate
- Critical bugs: Within 24-48 hours

---

## Pre-Release Testing

### Alpha Release (v1.0.0-alpha.1)
- Internal testing only
- May have known bugs
- APIs may change

### Beta Release (v1.0.0-beta.1)
- Feature complete
- External testing
- API mostly stable
- Bug fixes only

### Release Candidate (v1.0.0-rc.1)
- Production ready
- Final testing
- No new features
- Critical bugs only

---

## Post-Release Checklist

After publishing release:

- [ ] Update `docs/PROJECT_STATUS.md`
- [ ] Verify release on GitHub
- [ ] Test download and installation
- [ ] Monitor for issues
- [ ] Respond to feedback
- [ ] Plan next release

---

**Document Owner:** Development Team
**Last Reviewed:** 2025-11-12
