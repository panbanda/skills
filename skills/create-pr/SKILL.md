---
name: create-pr
description: Use when committing changes and creating pull requests - guides through branch management, semantic commits, and comprehensive PR descriptions
---

# Git PR Workflow

## Overview

Guide through committing changes and creating a pull request with proper git workflow practices.

**Use the `semantic-commits` skill for commit messages.**

## Process

### Step 1: Verify and Prepare Branch

1. Check current branch name
2. If on `main` branch:
   - Pull latest from origin/main
   - Review uncommitted changes (show diff summary)
   - Create feature branch: `feature/brief-description` or `fix/brief-description`
   - Switch to new branch

### Step 2: Commit Changes

1. Display summary of modified files and key changes
2. Use `semantic-commits` skill for commit message
3. Commit all staged changes
4. Push branch to remote origin

### Step 3: Create Pull Request

1. **Check for PR template:**
   - `.github/pull_request_template.md`
   - `.github/PULL_REQUEST_TEMPLATE.md`
   - `docs/pull_request_template.md`

2. **Generate PR title** (semantic format: `type(scope): description`):
   - Single commit: Use commit message as title
   - Multiple commits, same type/scope: Use that type/scope with summary
   - Multiple commits, different types: Use most significant (feat > fix > others)
   - Keep concise

3. **Create PR description:**

   If template exists: Fill in sections based on changes

   If no template:
   - **Summary**: Brief overview of changes and impact
   - **Changes Made**: Bullet points of key modifications
   - **Testing**: Relevant testing performed
   - **Notes**: Additional context for reviewers

4. Create PR against main using `gh pr create`
5. Provide link to created PR

## Output Guidelines

- Start with impact summary when changes affect multiple files
- Use clear, actionable language
- Highlight breaking changes or dependencies
- Focus on what reviewers need to know
- Never reference AI assistants in commits or PRs
- No emojis
