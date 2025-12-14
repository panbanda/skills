---
name: security-research
description: Use when conducting security audits, vulnerability assessments, pre-deployment security reviews, analyzing for OWASP Top 10 vulnerabilities, or generating security compliance reports
---

# Security Research and Analysis

## Overview

Comprehensive codebase security analysis using industry-standard tools and methodologies. Outputs security brief with findings categorized by severity.

## When to Use

- Security audit, scan, or vulnerability assessment
- Pre-deployment security review
- Compliance or security report generation
- Investigation of potential security issues
- Security posture assessment for a codebase

## Methodology

### Phase 1: Reconnaissance and Context Gathering

**Objective**: Understand the codebase architecture and identify attack surfaces.

```bash
# Identify entry points and external interfaces
rg -t py -t js -t go -t java "http\.|listen\(|bind\(|accept\(" --no-heading
rg -t py -t js -t go -t java "os\.system|exec\(|eval\(|subprocess" --no-heading
```

**Key Areas to Identify**:
- Web servers and API endpoints
- Database connections and queries
- File system operations
- External process execution
- Authentication and authorization mechanisms
- Input validation points
- Data serialization/deserialization
- Cryptographic operations

### Phase 2: Local Security Tooling Discovery and Execution

**Objective**: Discover and run any security tools, scripts, or checks built into the repository.

Many projects have custom security tooling integrated into their build process, CI/CD pipeline, or development workflow. This phase discovers and executes all local security checks.

#### Step 1: Discover Build System Security Targets

```bash
# Check for Makefile/Taskfile/Justfile security targets
for build_file in Makefile Taskfile.yml Taskfile.yaml justfile Justfile; do
  if [ -f "$build_file" ]; then
    echo "Checking $build_file for security targets..."
    rg "^[a-z-]*?(security|audit|scan|vuln)" "$build_file" || true
  fi
done

# Try common security target names
for target in security audit security-scan security-check vuln-check; do
  make "$target" 2>/dev/null || task "$target" 2>/dev/null || just "$target" 2>/dev/null || true
done
```

#### Step 2: Language-Specific Security Tool Discovery

Automatically detect project languages and run available security tools:

```bash
# Detect languages and run their security tools
detect_and_run_security_tools() {
  # Node.js/JavaScript
  if [ -f package.json ]; then
    jq -r '.scripts | keys[] | select(test("security|audit|scan"))' package.json 2>/dev/null
    npm audit 2>/dev/null || yarn audit 2>/dev/null || pnpm audit 2>/dev/null || true
  fi

  # Python
  if [ -f requirements.txt ] || [ -f pyproject.toml ]; then
    pip-audit 2>/dev/null || safety check 2>/dev/null || bandit -r . 2>/dev/null || true
  fi

  # Rust
  if [ -f Cargo.toml ]; then
    cargo audit 2>/dev/null || cargo deny check 2>/dev/null || true
  fi

  # Go
  if [ -f go.mod ]; then
    govulncheck ./... 2>/dev/null || gosec ./... 2>/dev/null || true
  fi

  # Java
  if [ -f pom.xml ]; then
    mvn dependency-check:check 2>/dev/null || true
  fi

  # Ruby
  if [ -f Gemfile ]; then
    bundle-audit check 2>/dev/null || brakeman 2>/dev/null || true
  fi

  # PHP
  if [ -f composer.json ]; then
    composer audit 2>/dev/null || true
  fi
}

detect_and_run_security_tools
```

#### Step 3: Container and Infrastructure Security

```bash
# Docker/Container security
if [ -f Dockerfile ] || [ -f docker-compose.yml ]; then
  trivy fs . 2>/dev/null || hadolint Dockerfile 2>/dev/null || true
fi

# Kubernetes/Helm security
if [ -d k8s ] || [ -d kubernetes ] || [ -f Chart.yaml ]; then
  kubesec scan k8s/*.yaml 2>/dev/null || kube-bench run 2>/dev/null || true
fi

# Terraform/IaC security
if fd -e tf | grep -q .; then
  tfsec . 2>/dev/null || checkov -d . 2>/dev/null || terrascan scan 2>/dev/null || true
fi
```

#### Step 4: CI/CD and Custom Security Scripts

```bash
# Check CI/CD for security jobs
for ci_file in .github/workflows/*.yml .gitlab-ci.yml .circleci/config.yml Jenkinsfile; do
  [ -f "$ci_file" ] && rg "security|audit|scan|sast|dast" "$ci_file" 2>/dev/null || true
done

# Find and list custom security scripts
for dir in scripts bin tools .github/scripts; do
  [ -d "$dir" ] && fd -e sh -e py "security|audit|scan" "$dir" 2>/dev/null || true
done

# Run pre-commit security hooks if configured
[ -f .pre-commit-config.yaml ] && pre-commit run --all-files 2>/dev/null || true
```

#### Step 5: Third-Party Security Tools

```bash
# Run third-party security tools if configured
snyk test 2>/dev/null || true
gitleaks detect 2>/dev/null || true
trufflehog filesystem . 2>/dev/null || true
git-secrets --scan 2>/dev/null || true
```

#### Step 6: Consolidate Local Tool Results

```bash
# Generate summary of all local security tool outputs
echo "=== Local Security Tools Summary ===" > local_security_summary.txt
fd -e json -e txt "audit|security|scan|vuln" --max-depth 1 >> local_security_summary.txt
```

**Integration**: Include all local tool findings in the final security report under "Local Security Tool Findings", deduplicated and cross-referenced with OWASP/CWE classifications.

### Phase 3: Semgrep Static Analysis

**Objective**: Detect specific vulnerabilities using rule-based static analysis.

```bash
# Run semgrep with security-focused rulesets
semgrep --config=auto --json --output=semgrep_results.json .

# Run OWASP Top 10 rules specifically
semgrep --config="p/owasp-top-ten" --json --output=semgrep_owasp.json .

# Run language-specific security rules
semgrep --config="p/security-audit" --json --output=semgrep_audit.json .

# Additional security rulesets
semgrep --config="p/cwe-top-25" --json --output=semgrep_cwe.json .
semgrep --config="p/secrets" --json --output=semgrep_secrets.json .
```

**Key Semgrep Rulesets**:
- `p/owasp-top-ten`: OWASP Top 10 vulnerabilities
- `p/security-audit`: Comprehensive security rules
- `p/cwe-top-25`: CWE Top 25 Most Dangerous Software Weaknesses
- `p/secrets`: Hardcoded secrets and credentials
- `p/jwt`: JWT security issues
- `p/sql-injection`: SQL injection patterns
- `p/xss`: Cross-site scripting vulnerabilities
- `p/command-injection`: OS command injection
- `p/path-traversal`: Path traversal vulnerabilities

### Phase 4: Manual Security Review

**Objective**: Analyze code for security issues that automated tools may miss.

**Authentication and Authorization**:
```bash
# Find authentication mechanisms
rg "auth|login|password|credential" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# Authorization checks
rg "permission|authorize|access_control|role|admin" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

**Input Validation**:
```bash
# Input sanitization and validation
rg "sanitize|validate|filter|escape|clean" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# Dangerous input functions
rg "eval|exec|system|shell|query" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

**Cryptography**:
```bash
# Cryptographic operations
rg "encrypt|decrypt|hash|hmac|cipher|crypto" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# Weak cryptography patterns
rg "md5|sha1|des|rc4|random\(\)" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

**Data Exposure**:
```bash
# Logging sensitive data
rg "log|print|console\.|debug|error" --type-add 'code:*.{py,js,go,java,rb,php}' -t code | rg "password|token|secret|key|credit"

# Hardcoded secrets
rg "api_key|secret|password|token" --type-add 'config:*.{json,yaml,yml,env,ini,conf}' -t config
```

**Dependencies and Third-Party Code**:
```bash
# List dependency files
fd "package.json|requirements.txt|Gemfile|go.mod|pom.xml|Cargo.toml"

# Check for known vulnerable dependencies (if applicable tools are available)
# npm audit (for Node.js)
# pip-audit (for Python)
# cargo audit (for Rust)
```

### Phase 5: Analysis and Report Generation

**Objective**: Consolidate findings into industry-standard security brief.

## Security Report Format

Generate a comprehensive security report with the following structure:

```markdown
# Security Analysis Report

**Project**: <project_name>
**Date**: <date>
**Analyst**: Security Research Skill
**Scope**: <full_codebase|specific_components>

---

## Executive Summary

[2-3 paragraphs summarizing key findings, overall security posture, and critical recommendations]

**Overall Risk Rating**: <CRITICAL|HIGH|MEDIUM|LOW>

**Key Metrics**:
- Total Vulnerabilities: X
- Critical: X
- High: X
- Medium: X
- Low: X
- Informational: X

---

## Findings

### CRITICAL Severity Issues

#### [CWE-XXX] Vulnerability Title
- **Severity**: CRITICAL
- **CWE**: CWE-XXX (link)
- **OWASP**: A01:2021 Category
- **Location**: `path/to/file.ext:line_number`
- **Description**: Detailed description of the vulnerability
- **Impact**: What an attacker could achieve
- **Exploit Scenario**: How the vulnerability could be exploited
- **Remediation**: Specific steps to fix the issue
- **References**: Links to relevant documentation/CVEs

[Repeat for each critical finding]

### HIGH Severity Issues

[Follow same structure as critical]

### MEDIUM Severity Issues

[Follow same structure]

### LOW Severity Issues

[Follow same structure]

### Informational Findings

[Security improvements and best practices recommendations]

---

## Attack Surface Analysis

### Entry Points
1. **HTTP Endpoints**: List all API routes and web endpoints
2. **File Operations**: File upload, download, and processing capabilities
3. **External Integrations**: Third-party API calls and webhooks
4. **Database Access**: SQL/NoSQL query interfaces

### Trust Boundaries
- External user input
- Inter-service communication
- Third-party libraries
- Configuration files

---

## Security Hotspots

**High-Risk Files** (by complexity and security impact):
1. `path/to/file.ext` - Risk Score: X/100
   - Complexity: HIGH
   - Security-sensitive operations: auth, data access, file handling
   - Recommendation: Priority refactoring

[Continue for top 10 hotspots]

---

## Compliance Assessment

### OWASP Top 10 (2021)

- **A01:2021 - Broken Access Control**: <PASS|FAIL> - [Findings]
- **A02:2021 - Cryptographic Failures**: <PASS|FAIL> - [Findings]
- **A03:2021 - Injection**: <PASS|FAIL> - [Findings]
- **A04:2021 - Insecure Design**: <PASS|FAIL> - [Findings]
- **A05:2021 - Security Misconfiguration**: <PASS|FAIL> - [Findings]
- **A06:2021 - Vulnerable Components**: <PASS|FAIL> - [Findings]
- **A07:2021 - Identification and Authentication Failures**: <PASS|FAIL> - [Findings]
- **A08:2021 - Software and Data Integrity Failures**: <PASS|FAIL> - [Findings]
- **A09:2021 - Security Logging and Monitoring Failures**: <PASS|FAIL> - [Findings]
- **A10:2021 - Server-Side Request Forgery**: <PASS|FAIL> - [Findings]

### CWE Top 25

List any CWE Top 25 vulnerabilities found with references.

---

## Remediation Roadmap

### Immediate Actions (Critical - within 24-48 hours)
1. [Action item with specific file/line references]
2. [Action item]

### Short-term (High - within 1-2 weeks)
1. [Action item]
2. [Action item]

### Medium-term (Medium - within 1 month)
1. [Action item]
2. [Action item]

### Long-term (Low/Informational - within 3 months)
1. [Action item]
2. [Action item]

---

## Security Best Practices Recommendations

### Authentication and Authorization
- [Specific recommendations based on findings]

### Input Validation
- [Specific recommendations]

### Cryptography
- [Specific recommendations]

### Error Handling and Logging
- [Specific recommendations]

### Dependency Management
- [Specific recommendations]

### Configuration and Deployment
- [Specific recommendations]

---

## Tool Output Summary

### Local Security Tools
- Tools discovered: X
- Tools executed: X
- Findings: X

### Semgrep Static Analysis
- Rules executed: X
- Findings: X
- False positive rate: ~X%

---

## Appendices

### A. Detailed Tool Output
- Link to full semgrep JSON output
- Link to local tool outputs

### B. References
- OWASP Top 10: https://owasp.org/Top10/
- CWE Top 25: https://cwe.mitre.org/top25/
- [Additional references]

---

**Report Generated**: <timestamp>
```

## Severity Classification

Use this standard severity classification:

### CRITICAL
- **Definition**: Immediate and severe risk to confidentiality, integrity, or availability
- **Examples**:
  - Remote code execution (RCE)
  - SQL injection in authentication
  - Hard-coded credentials in production
  - Authentication bypass
  - Direct object reference with admin access
- **CVSS Score**: 9.0-10.0
- **Remediation**: Immediate (within 24 hours)

### HIGH
- **Definition**: Significant risk that could lead to major security incidents
- **Examples**:
  - Cross-site scripting (XSS) in critical flows
  - Insecure deserialization
  - Weak cryptography for sensitive data
  - Missing authorization checks
  - Path traversal vulnerabilities
- **CVSS Score**: 7.0-8.9
- **Remediation**: Urgent (within 1 week)

### MEDIUM
- **Definition**: Moderate risk that could impact security posture
- **Examples**:
  - Information disclosure (non-sensitive)
  - Missing security headers
  - Insufficient logging
  - Outdated dependencies (no known exploits)
  - Weak session management
- **CVSS Score**: 4.0-6.9
- **Remediation**: Important (within 1 month)

### LOW
- **Definition**: Minor security concerns with limited impact
- **Examples**:
  - Verbose error messages
  - Missing rate limiting
  - Non-exploitable code quality issues
  - Minor configuration improvements
- **CVSS Score**: 0.1-3.9
- **Remediation**: Optional (within 3 months)

### INFORMATIONAL
- **Definition**: Security best practices and hardening recommendations
- **Examples**:
  - Security header recommendations
  - Code organization suggestions
  - Documentation improvements
  - Proactive security enhancements
- **CVSS Score**: 0.0
- **Remediation**: As time permits

## Common Vulnerability Patterns

### Injection Attacks

**SQL Injection**:
```bash
# Search for unsafe SQL construction
rg "execute\(.*\+|query\(.*\+|raw\(.*\+" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
rg "SELECT.*\%s|INSERT.*\%s|UPDATE.*\%s" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

**Command Injection**:
```bash
# Search for shell command construction
rg "system\(|exec\(|popen\(|shell_exec\(|eval\(" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
rg "Runtime\.exec|ProcessBuilder" -t java
```

**NoSQL Injection**:
```bash
# MongoDB and NoSQL patterns
rg "\$where|\$regex|\$ne\[" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

### Authentication and Authorization

**Broken Authentication**:
```bash
# Weak password requirements
rg "password.*length.*[0-6]" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# Missing authentication
rg "@app\.route|@router\.|app\.get|app\.post" --type-add 'code:*.{py,js,go,java,rb,php}' -t code | rg -v "auth|login|permission"
```

**Authorization Bypass**:
```bash
# Direct object references
rg "params\[.id.\]|request\.params|getId\(\)" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# Missing authorization checks
rg "delete|update|modify" --type-add 'code:*.{py,js,go,java,rb,php}' -t code | rg -v "authorize|permission|check"
```

### Sensitive Data Exposure

**Weak Cryptography**:
```bash
# Deprecated algorithms
rg "MD5|SHA1|DES|RC4|ECB" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# Hardcoded keys and secrets
rg "api_key\s*=\s*['\"]|secret\s*=\s*['\"]|password\s*=\s*['\"]" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

**Information Leakage**:
```bash
# Debug mode in production
rg "DEBUG\s*=\s*True|debug:\s*true|development" --type-add 'config:*.{py,js,json,yaml,yml,env}' -t config

# Verbose error messages
rg "printStackTrace|console\.error|raise.*Exception" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

### Cross-Site Scripting (XSS)

```bash
# Unsafe HTML rendering
rg "innerHTML|dangerouslySetInnerHTML|html\(|render_template_string" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# Missing output encoding
rg "print|echo|write" --type-add 'code:*.{py,js,go,java,rb,php}' -t code | rg -v "escape|encode|sanitize"
```

### Security Misconfiguration

```bash
# Default credentials
rg "admin:admin|root:root|password:password" --type-add 'config:*' -t config

# Exposed debug endpoints
rg "/debug|/test|/admin" --type-add 'code:*.{py,js,go,java,rb,php}' -t code

# CORS misconfiguration
rg "Access-Control-Allow-Origin.*\*|cors.*allow_all" --type-add 'code:*.{py,js,go,java,rb,php}' -t code
```

## Integration Workflow

### Step 1: Initial Scan
```bash
# Quick security triage
semgrep --config=auto --severity ERROR --severity WARNING .
```

### Step 2: Deep Analysis
```bash
# Full semgrep analysis
semgrep --config="p/owasp-top-ten" --config="p/cwe-top-25" --config="p/secrets" --json .
```

### Step 3: Manual Review
Focus on:
- High-complexity files
- Files with multiple semgrep findings
- Authentication and authorization logic
- Input handling and validation
- Cryptographic operations

### Step 4: Report Generation
Consolidate all findings into the standard security report format.

### Step 5: Remediation Tracking
Create actionable tickets for each finding with:
- Clear description and reproduction steps
- Severity classification
- Remediation guidance
- Code references

## False Positive Management

**Common False Positives**:
1. **Safe SQL with ORMs**: Parameterized queries flagged as injection
2. **Internal Admin Tools**: Admin endpoints in development/internal tools
3. **Test Code**: Security issues in test fixtures and mocks
4. **Generated Code**: Auto-generated files with security patterns

**Verification Steps**:
1. Examine the actual code implementation
2. Check for sanitization/validation before the flagged line
3. Understand the data flow and trust boundaries
4. Consider the deployment context

## Tool Limitations

### Semgrep Limitations
- Pattern-based detection (may miss complex logic flaws)
- False positives in complex codebases
- Limited data flow analysis
- May not understand business logic context

**Best Practice**: Combine automated tools with manual security review for comprehensive coverage.

## Continuous Security

### Pre-commit Hooks
```bash
# Add semgrep to pre-commit
semgrep --config=auto --severity ERROR .
```

### CI/CD Integration
```bash
# Add to CI pipeline
semgrep --config="p/security-audit" --error --json .
```

## Example Usage

### Example 1: Full Security Audit
```bash
# User: "Run a complete security audit of this codebase"

# Step 1: Context gathering
# Explore codebase structure and identify key files

# Step 2: Local security tools
# Run all discovered security tools

# Step 3: Semgrep analysis
semgrep --config="p/owasp-top-ten" --config="p/cwe-top-25" --config="p/secrets" --json --output=semgrep_full.json .

# Step 4: Manual review of findings
bat semgrep_full.json | jq '.results[] | select(.extra.severity == "ERROR")'

# Step 5: Generate comprehensive security report
# [Consolidate findings into report format]
```

### Example 2: Pre-deployment Security Check
```bash
# User: "Quick security check before we deploy"

# Step 1: Critical issues only
semgrep --config="p/security-audit" --severity ERROR --severity WARNING .

# Step 2: Secrets scan
semgrep --config="p/secrets" .

# Step 3: OWASP Top 10 check
semgrep --config="p/owasp-top-ten" .

# Step 4: Generate quick summary
# [Provide executive summary of critical/high findings]
```

### Example 3: Targeted Security Review
```bash
# User: "Review the authentication module for security issues"

# Step 1: Find auth-related files
fd "auth|login" --type f

# Step 2: Run semgrep on specific paths
semgrep --config="p/security-audit" src/auth/

# Step 3: Manual review
rg "password|token|session" src/auth/ --context 5

# Step 4: Generate focused report
# [Provide auth-specific security analysis]
```

## References and Resources

### Standards and Guidelines
- **OWASP Top 10**: https://owasp.org/Top10/
- **CWE Top 25**: https://cwe.mitre.org/top25/
- **SANS Top 25**: https://www.sans.org/top25-software-errors/
- **NIST Guidelines**: https://www.nist.gov/cybersecurity
- **OWASP ASVS**: https://owasp.org/www-project-application-security-verification-standard/

### Tools Documentation
- **Semgrep Rules**: https://semgrep.dev/explore
- **CVSS Calculator**: https://www.first.org/cvss/calculator/3.1

### Training and Learning
- **OWASP Web Security Testing Guide**: https://owasp.org/www-project-web-security-testing-guide/
- **PortSwigger Web Security Academy**: https://portswigger.net/web-security
- **CWE Database**: https://cwe.mitre.org/

---

**Remember**: Security is not a one-time activity. Regular security assessments, continuous monitoring, and proactive vulnerability management are essential for maintaining a strong security posture.
