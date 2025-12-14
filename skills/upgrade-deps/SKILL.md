---
name: upgrade-deps
description: Use when upgrading software dependencies - systematic process for safe, thorough upgrades across any language ecosystem with proper research, testing, and documentation
---

# Dependency Upgrade

## Overview

Systematically upgrade software dependencies with safety and thoroughness. Never upgrade blindly - always understand what changed.

## Core Principles

1. Never upgrade blindly - always understand what changed
2. Upgrade incrementally - one major version at a time
3. Check compatibility across the dependency tree
4. Test continuously after each significant change
5. Document breaking changes and migration steps

## Process

### Phase 1: Discovery & Assessment

1. **Identify outdated dependencies**
   ```bash
   npm outdated                    # Node.js
   pip list --outdated             # Python
   cargo outdated                  # Rust
   go list -u -m all               # Go
   ```

2. **Prioritize by risk**
   - Security patches: Immediate priority
   - Patch versions (x.y.Z): Low risk, do first
   - Minor versions (x.Y.z): Medium risk, review changelog
   - Major versions (X.y.z): High risk, treat as dedicated task

3. **Check version constraints**
   ```bash
   npm ls <package>                # Node.js dependency tree
   cargo tree                      # Rust dependency tree
   go mod why <package>            # Go - why dependency exists
   pip show <package>              # Python package info
   ```

### Phase 2: Research & Analysis

For each dependency to upgrade:

1. **Find and read release documentation**
   - Changelog/CHANGELOG.md in repository
   - GitHub releases page
   - Migration guides (UPGRADING.md, MIGRATION.md)
   - Use WebFetch if needed to retrieve documentation

2. **Identify breaking changes**
   - API removals or renames
   - Behavior changes
   - Configuration format changes
   - Minimum runtime version requirements
   - Deprecated features being removed

3. **Check security context**
   - Search for CVEs related to old version
   - Review security advisories
   - Understand what vulnerability is being patched

4. **Determine upgrade path**
   - Can you jump directly to latest?
   - Do intermediate versions exist with migration guides?
   - Check peer dependency compatibility

### Phase 3: Impact Assessment

1. **Search codebase for usage**
   ```bash
   # Find imports and usage of specific APIs mentioned in breaking changes
   rg "import.*<package>" -g "*.{js,ts,py,go,rs}"
   ```

2. **Identify all version references** (critical for runtime upgrades)

   **For language runtimes (go, node, python, ruby, rust):**
   - Version manager configs: `.tool-versions`, `.nvmrc`, `.ruby-version`, `.python-version`
   - GitHub Actions: `.github/workflows/*.yml`
   - Docker: `Dockerfile`, `docker-compose.yml`
   - Package manifests: `package.json` (engines), `go.mod` (go directive)
   - Documentation: `README.md`, `CONTRIBUTING.md`

   ```bash
   # Example: upgrading Go from 1.21 to 1.22
   rg "1\.21" -g "*.{yml,yaml,toml,json,md,Dockerfile}" -g ".tool-versions"

   # Example: upgrading Node from 18 to 20
   rg "(node:18|nodejs.*18|18\\..*\\.)" -g "*.{yml,yaml,json,md,Dockerfile}" -g ".nvmrc"
   ```

3. **Check for deprecation warnings**
   - Run existing tests, capture warnings
   - Search for usage of deprecated APIs

### Phase 4: Execution

1. **Create feature branch**
   ```
   chore/upgrade-<package>-to-<version>
   ```

2. **Update dependency declaration**
   ```bash
   npm install <package>@latest    # Node.js
   pip install --upgrade <package> # Python
   cargo update <package>          # Rust
   go get <package>@latest         # Go
   ```

3. **Review lock file changes**
   - Check for unexpected transitive updates
   - Verify expected packages updated
   - Look for new dependencies introduced

4. **Run tests immediately**
   - Identify compilation/build errors
   - Note test failures
   - Capture error messages for analysis

5. **Apply code changes incrementally**
   - Fix breaking changes one at a time
   - Update deprecated API usage
   - Run relevant test subset after each fix

### Phase 5: Verification

1. **Run full test suite**
   - Unit tests
   - Integration tests
   - End-to-end tests if available

2. **Check for warnings**
   - Deprecation warnings to address
   - Compiler/linter warnings
   - Runtime warnings in output

3. **Performance check (if relevant)**
   - Bundle size changes for frontend deps
   - Build time changes
   - Runtime performance

### Phase 6: Documentation & Commit

1. **Document changes**
   - What was upgraded (from X to Y)
   - Why (security, bug fix, needed features)
   - Breaking changes addressed
   - Testing performed

2. **Commit format**
   - Use `chore(deps):` for routine updates
   - Use `fix(deps):` for security patches
   - Include version numbers in description

## Decision Framework

### When to proceed with caution:
- Multiple major versions behind (e.g., v2 -> v5)
- No changelog available
- Active issues reported about new version
- Breaking changes without migration guide
- Many new transitive dependencies

### When to roll back:
- Tests fail without clear fix path
- Significant performance degradation
- New bugs in production-like environment
- Breaking changes too extensive for timeline

### When to split into multiple commits:
- Multiple independent dependency updates
- Update + code changes to accommodate it
- Security patch + feature version bump
- Different categories (dev deps vs runtime deps)

## Ecosystem-Specific Commands

### Node.js/npm/pnpm/yarn
```bash
npm outdated                    # List outdated
npm ls <package>                # Dependency tree
npm audit                       # Security check
npm update <package>            # Update
```

### Python/pip
```bash
pip list --outdated             # List outdated
pip show <package>              # Package info
pip install --upgrade <package> # Update
```

### Rust/Cargo
```bash
cargo outdated                  # List outdated (requires cargo-outdated)
cargo tree                      # Dependency tree
cargo update --dry-run          # Preview updates
cargo update <package>          # Update
cargo audit                     # Security check (requires cargo-audit)
```

### Go
```bash
go list -u -m all               # List outdated
go mod why <package>            # Why dependency exists
go get <package>@latest         # Update
go mod tidy                     # Clean up
```

## Special Cases

### Security Vulnerabilities
- Prioritize immediately
- Understand the CVE and exploit potential
- Test thoroughly but expedite
- Document security context in commit

### Transitive Dependencies
- Identify why it exists (dependency tree)
- Check if direct dependency update fixes it
- Update direct dependency rather than forcing transitive version

### Deprecated Packages
- Find recommended replacement
- Check migration guides
- Assess if functionality still needed
- Plan phased migration if replacement is significantly different

### Pre-release Versions
- Generally avoid in production
- Useful for testing upcoming changes
- Check stability promises (alpha/beta/rc)
- Pin exact version
- Have rollback plan

### Runtime/Toolchain Upgrades

**Critical locations to update:**
1. Version manager config (`.tool-versions`, `.nvmrc`)
2. CI/CD workflows (GitHub Actions, GitLab CI)
3. Docker base images
4. Package manifest constraints
5. Documentation

**Commit all related version updates together:**
```
chore: upgrade go from 1.21 to 1.22

Update go version across all environments:
- .tool-versions (asdf)
- go.mod go directive
- GitHub Actions workflows
- Dockerfile base image
- Documentation
```

## Red Flags

Stop and reassess if:
- Changelog is missing or vague
- Package was transferred to new owner recently
- Significant increase in package size
- New runtime requirements incompatible with environment
- Breaking changes affect core functionality extensively
- Community reports serious issues

## Output Format

After completing upgrade, provide:

**Summary**: Package upgraded from X to Y, reason for upgrade

**Breaking Changes**: What broke and how it was fixed

**Testing**: Tests run and results

**Risks**: Known issues or concerns

**Rollback**: How to revert if needed (usually `git revert` + restore old version)
