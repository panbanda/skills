---
name: code-review
description: Use when reviewing pull requests, code changes, or merges - runs Omen analysis for metrics and risk, applies industry best practices from Google, Microsoft, and OWASP, provides deterministic approval status with actionable feedback
---

# Code Review

Structured code review process combining automated metrics analysis with manual inspection across security, performance, quality, complexity, and maintainability.

## Review Process

1. **Gather context** - Run Omen analysis and read changed files
2. **Analyze changes** - Review each category systematically
3. **Document findings** - Record issues with severity and location
4. **Determine status** - Apply decision criteria for approval status
5. **Write review** - Output structured review with next steps

## Step 1: Gather Context

Run Omen commands to get objective metrics:

```bash
# PR/merge risk assessment (required - run first)
omen analyze diff -f markdown              # JIT defect prediction for this PR
omen score -f markdown                     # Baseline repo health (compare before/after)

# Additional context (run as needed)
omen analyze complexity --format markdown  # Cyclomatic/cognitive complexity
omen analyze tdg --format markdown         # Technical debt scores
omen analyze hotspot --format markdown     # High-risk files (churn + complexity)
omen analyze satd --format markdown        # TODOs, FIXMEs, HACKs
omen analyze defect --format markdown      # Per-file defect probability
omen analyze smells --format markdown      # Architectural issues

# Blast radius assessment (run for shared/core module changes)
omen analyze graph --metrics --scope file  # Dependencies, PageRank, coupling
```

**PR risk factors (from `omen analyze diff`):**
- Lines added/deleted - larger changes = higher risk
- Files touched - more files = higher risk
- Developer experience with files - unfamiliar code = higher risk
- File age - new files vs established code
- Historical defect density of touched files

**Graph metrics indicate blast radius:**
- High PageRank = central file, changes affect many dependents
- High in-degree = many files depend on this, breaking changes risky
- High betweenness = bridge file connecting modules, integration risk

**Use graph edges to identify files to review:**
- The Mermaid output shows `A --> B` meaning A depends on B
- For each changed file, check files with arrows pointing TO it (dependents)
- If interface/signature changed, review dependent files for compatibility
- Run `omen analyze graph --metrics --scope file` for file-level dependencies

Read the changed files to understand the actual code changes.

## Step 2: Analyze Changes

### Requirements Check (if ticket/plan available)

If requirements or ticket not provided, ask: "Is there a ticket or requirements doc for this change?"

When available, verify:
- [ ] All requirements met?
- [ ] Implementation matches spec?
- [ ] No scope creep?
- [ ] Breaking changes documented?

### Code Review Categories

Review each category. See `references/checklists.md` for detailed checklists.

| Category | Focus Areas |
|----------|-------------|
| **Security** | Input validation, auth, injection, XSS, secrets, access control |
| **Performance** | Complexity, N+1 queries, caching, memory, concurrency |
| **Quality** | Tests, error handling, logging, edge cases |
| **Complexity** | Cyclomatic <15, cognitive <15, function length, nesting depth |
| **Maintainability** | Naming, single responsibility, comments explain why not what |
| **Design** | Coupling, cohesion, SOLID principles, API design |

## Step 3: Document Findings

Record each issue with:
- **Severity**: `blocking`, `should-fix`, `nitpick`
- **Category**: security, performance, quality, complexity, maintainability, design
- **Location**: `file:line`
- **Description**: What's wrong and why
- **Suggestion**: How to fix it

## Step 4: Determine Approval Status

Apply these decision criteria:

```
HAS blocking issues? → CHANGES_REQUESTED
HAS architectural/design concerns needing human input? → NEEDS_DISCUSSION
HAS only should-fix or nitpick issues? → APPROVED (with comments)
NO issues? → APPROVED
```

**Blocking issues include:**
- Security vulnerabilities (injection, XSS, auth bypass, secrets exposed)
- Data loss or corruption risk
- Breaking changes without migration
- Missing tests for critical paths
- Cyclomatic complexity >25 or cognitive complexity >30
- Defect prediction score >70% on changed files
- PR risk score >70 without mitigating factors (tests, small scope)

**NEEDS_DISCUSSION triggers:**
- Significant architectural changes
- New dependencies with security/license implications
- Performance tradeoffs requiring product input
- Ambiguous requirements
- Changes to high-PageRank files (top 10% centrality) without migration plan

## Step 5: Write Review

Output format:

```markdown
## Code Review Summary

**Ready to merge:** [Yes | No | With fixes]

### Omen Analysis

**Merge Risk:** X/100 [Low|Medium|High]
- Repo health: X/100 (before) -> Y/100 (after)
- Files changed: N
- Lines added/deleted: +X/-Y

**Risk Factors:**
- Complexity hotspots: [list if any]
- High-centrality files changed: [list if any PageRank >0.1]
- Dependent files to verify: [files that import changed files]
- Defect probability: [files with >30% probability]

### Strengths
[What's well done - be specific with file:line references]

### Issues

#### Blocking (must fix before merge)
[List with file:line, what's wrong, why it matters, how to fix]

#### Should Fix
[List with file:line, what's wrong, why it matters]

#### Nitpicks
[List minor improvements]

### Recommendations
[Forward-looking suggestions for code quality, architecture, or process]

### Verdict

**Ready to merge:** [Yes | No | With fixes]

**Reasoning:** [1-2 sentence technical assessment]

**Next steps:**
- Yes: "Merge when ready."
- With fixes: "Address [specific issues], then merge."
- No: "Fix blocking issues and re-request review."
```

## Review Standards

### From Google
- Seek continuous improvement, not perfection
- Small changes reviewed quickly (target <400 LOC)
- Comments explain "why" not "what"
- Style guide is authority on style matters

### From Microsoft
- Correctness of business logic first
- Readability and maintainability matter
- Methods with >3 arguments may be overly complex
- "A bug caught matters - not who made it, found it, or fixed it"

### From Meta
- Every change must have a test plan
- Balance review speed with quality (avoid rubber-stamping)
- 75% of review comments affect maintainability, not functionality

## Common Mistakes to Catch

- Missing input validation at system boundaries
- Error messages exposing sensitive information
- N+1 query patterns in loops
- Hardcoded secrets or credentials
- Missing null/undefined checks
- Unbounded loops or recursion
- Race conditions in async code
- Missing cleanup in error paths

## Reviewer Rules

**DO:**
- Run Omen analysis before reviewing code
- Categorize by actual severity (not everything is blocking)
- Be specific with file:line references
- Explain WHY issues matter, not just what's wrong
- Acknowledge strengths - what's well done
- Give a clear verdict with reasoning

**DON'T:**
- Say "looks good" without actually reviewing
- Mark nitpicks as blocking issues
- Give vague feedback ("improve error handling")
- Skip the Omen metrics
- Avoid giving a clear verdict
- Review code you haven't read
