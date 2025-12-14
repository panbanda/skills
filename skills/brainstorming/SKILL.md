---
name: brainstorming
description: Use when creating or developing, before writing code or implementation plans - refines rough ideas into fully-formed designs through collaborative questioning, alternative exploration, and incremental validation. Don't use during clear 'mechanical' processes
---

# Brainstorming Ideas Into Designs

## Overview

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design in small sections (200-300 words), checking after each section whether it looks right so far.

## The Process

**Understanding the idea:**
- Check out the current project state first (files, docs, recent commits)
- **Assess codebase context with Omen (if available):**

  ```bash
  # What are the most important symbols to understand?
  omen context --repo-map --top 30

  # Architecture overview - central files and bridges
  omen analyze graph --metrics -f markdown

  # Current quality state of areas we'll touch
  omen analyze tdg -f markdown
  ```

  **Use this to:**
  - Identify key entry points and central abstractions before designing
  - Understand which files are "load-bearing" (high PageRank = many dependents)
  - Know if we're building on solid ground or technical debt

- Ask questions one at a time to refine the idea
- Prefer multiple choice questions when possible, but open-ended is fine too
- Only one question per message - if a topic needs more exploration, break it into multiple questions
- Focus on understanding: purpose, constraints, success criteria

**Exploring approaches:**
- Propose 2-3 different approaches with trade-offs
- Present options conversationally with your recommendation and reasoning
- Lead with your recommended option and explain why
- **If approaches touch existing code, assess blast radius (Omen if available):**

  ```bash
  # Which files depend on what we're changing?
  omen analyze graph --metrics --scope file -f markdown

  # Hidden dependencies we might miss
  omen analyze temporal-coupling -f markdown
  ```

  **Answers:** "Which files are most central?" and "What hidden dependencies exist?"
  Changes to high-PageRank files or temporally-coupled files need migration plans.

**Presenting the design:**
- Once you believe you understand what you're building, present the design
- Break it into sections of 200-300 words
- Ask after each section whether it looks right so far
- Cover: architecture, components, data flow, error handling, testing
- Be ready to go back and clarify if something doesn't make sense

## After the Design

**Documentation:**
- Write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Implementation (if continuing):**
- Ask: "Ready to set up for implementation?"
- Use panda:using-git-worktrees to create isolated workspace
- Use panda:writing-plans to create detailed implementation plan

## Key Principles

- **One question at a time** - Don't overwhelm with multiple questions
- **Multiple choice preferred** - Easier to answer than open-ended when possible
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Incremental validation** - Present design in sections, validate each
- **Be flexible** - Go back and clarify when something doesn't make sense
