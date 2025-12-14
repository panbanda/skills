---
name: semantic-commits
description: Use when creating git commits - generates conventional commit messages following the Conventional Commits 1.0.0 specification for consistent, semantic commit messages
---

# Semantic Commit Messages

## Overview

Follow the Conventional Commits 1.0.0 specification. Every commit tells a story.

**Core principle:** Good commit messages make code history searchable and maintainable.

**Use this skill instead of the built-in commit behavior.**

**Spec:** https://www.conventionalcommits.org/en/v1.0.0/

## When to Use

**Always:**
- Creating any git commit
- Reviewing commit message quality
- Invoked from /create-pr

**Never:**
- Mention "Claude" or "Claude Code" in commit messages

## Commit Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Required:** type, colon, space, description

**Optional:** scope (in parentheses), body (blank line after description), footers (blank line after body)

## Types

| Type | Use When | SemVer |
|------|----------|--------|
| **feat** | Adding new functionality | MINOR |
| **fix** | Patching a bug | PATCH |
| **docs** | Documentation only | - |
| **style** | Formatting, whitespace | - |
| **refactor** | Neither fix nor feature | - |
| **test** | Adding/updating tests | - |
| **chore** | Maintenance, deps, tooling | - |
| **perf** | Performance improvements | - |
| **build** | Build system changes | - |
| **ci** | CI/CD changes | - |

## Decision Tree

1. Adds new functionality? -> **feat**
2. Fixes a bug? -> **fix**
3. Only documentation? -> **docs**
4. Improves performance? -> **perf**
5. Changes structure, not behavior? -> **refactor**
6. Adds/updates tests? -> **test**
7. Formatting/whitespace? -> **style**
8. Dependencies or tooling? -> **chore**

## Description Guidelines

- Imperative mood: "add" not "added"
- Lowercase, no period
- Under 50 characters
- Concise summary of what changed

<Good>
```
feat(auth): add JWT token validation
fix(api): handle null response from external service
docs: update installation instructions
chore(deps): upgrade typescript to 5.3.0
```
</Good>

<Bad>
```
fix: bug fix
feat: new feature
chore: updates
feat(api): added new endpoint
```
Vague, wrong tense
</Bad>

## Scope Guidelines

Noun describing section of codebase. Keep consistent within project.

**Good scopes:**
- Module: auth, api, database, ui, cli
- Feature: payments, notifications, reports
- Subsystem: parser, compiler, renderer

## Body (Optional)

<Good>
```
feat(api): add rate limiting middleware

Implement token bucket algorithm with Redis backend to prevent
API abuse. Default limit is 100 requests per minute per IP address,
configurable via environment variables.

Fixes #234
```
Explains what and why, references issue
</Good>

<Bad>
```
fix(auth): modify validateToken function in src/services/auth.ts line 42 to check expiration
```
Too detailed - the diff shows what changed
</Bad>

**Rules:**
- One blank line after description
- Explain what/why, not how
- Wrap at 72 characters
- For complex changes only; omit for simple ones

## Footers (Optional)

```
Fixes #123
Closes #456
Refs #789
Reviewed-by: Alice <alice@example.com>
```

One blank line after body. Use hyphens in tokens (Signed-off-by, Acked-by).

## Breaking Changes

**Preferred: BREAKING CHANGE footer**

```
feat(api): change response format to JSON:API spec

Update all API endpoints to follow JSON:API specification
for consistency with industry standards.

BREAKING CHANGE: Response structure changed from flat objects
to JSON:API format. Clients need to update response parsing.
```

**Alternative: ! in subject**

```
feat(api)!: change response format to JSON:API spec
```

**Rules:**
- BREAKING CHANGE must be uppercase
- Correlates with MAJOR in SemVer
- Prefer footer method - allows explanation

## Anti-patterns

| Problem | Bad | Good |
|---------|-----|------|
| Too vague | `fix: bug fix` | `fix(auth): validate token expiration` |
| Too detailed | `fix(auth): modify validateToken in line 42` | `fix(auth): validate token expiration` |
| Wrong tense | `feat(api): added endpoint` | `feat(api): add endpoint` |
| Too long | `feat(api): add comprehensive rate limiting middleware with Redis backend and configurable limits` | `feat(api): add rate limiting with Redis` |

## Multiple Changes

Split into separate commits:
- Feature additions + bug fixes
- Multiple unrelated features
- Code changes + dependency updates

Each commit should:
- Have one logical change
- Pass tests independently
- Be self-explanatory

## Common Scopes by Project Type

**Web API:** api, routes, middleware, auth, db, config

**Frontend:** ui, components, styles, state, hooks, utils

**CLI Tool:** cli, commands, config, output, parser

**Library:** core, utils, types, docs, tests

## Workflow

```bash
# View staged changes
git diff --staged

# View modified files
git status --short

# View commit history for style reference
git log --oneline -10
```

1. Review the diff
2. Identify primary change type
3. Determine scope (if applicable)
4. Write concise subject
5. Add body only if needed

## Final Rule

```
type(scope): imperative description under 50 chars
```

Future developers will thank you.
