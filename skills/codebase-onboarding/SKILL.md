---
name: codebase-onboarding
description: Use when starting work on an unfamiliar codebase - generates AI-friendly context using Omen's PageRank-ranked symbols, architecture overview, and health assessment
---

# Codebase Onboarding

## Overview

Get oriented in an unfamiliar codebase quickly. Generates context optimized for AI consumption using structural importance rather than alphabetical listing.

**Requires:** Omen CLI installed (`omen --version` to check)

## When to Use

- First time working in a repository
- Returning to a project after extended absence
- Preparing codebase context for AI assistants
- Onboarding new team members

## The Process

### 1. Generate Repository Map

```bash
omen context --repo-map --top 50
```

**Answers:** "What are the most important functions and types?"

PageRank-ranked symbols show what matters structurally, not just what's largest. Start understanding here.

### 2. Assess Overall Health

```bash
omen score -f markdown
```

**Answers:** "What's the overall codebase health?"

Sets expectations. Score <60 means expect friction. Score >80 means well-maintained.

### 3. Understand Architecture

```bash
omen analyze graph --metrics -f markdown
```

**Answers:** "Which files are central? What are the bridges?"

- **High PageRank** = load-bearing files, changes here ripple everywhere
- **High betweenness** = bridges connecting modules, integration points
- **Cycles** = architectural problems, tread carefully

### 4. Identify Known Problems

```bash
# Where are the hotspots?
omen analyze hotspot -f markdown

# Acknowledged debt
omen analyze satd -f markdown
```

**Answers:**
- Hotspots: "Which complex files change constantly?" - Avoid if possible, extra care if not
- SATD: "Where have developers marked problems?" - Known landmines

### 5. Summarize Findings

After running commands, synthesize into a brief orientation:

```markdown
## Codebase Orientation: [project-name]

**Health:** X/100 - [interpretation]

**Key Entry Points:**
- [top 3-5 symbols from repo-map with one-line descriptions]

**Architecture:**
- [2-3 sentences about structure from graph analysis]

**Watch Out For:**
- [hotspots to avoid or handle carefully]
- [significant SATD clusters]
```

## Quick Version

For rapid orientation, run just:

```bash
omen context --repo-map --top 20 --include-metrics
```

This gives PageRank-ranked symbols with complexity metrics in one command.
