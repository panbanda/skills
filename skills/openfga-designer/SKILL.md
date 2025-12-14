---
name: openfga-designer
description: Use when designing, reviewing, or evolving OpenFGA authorization models - guides requirements gathering, pattern selection, and model structure through questioning and iteration
---

# OpenFGA Authorization Model Designer

## Core Philosophy

OpenFGA uses **Relationship-Based Access Control (ReBAC)**. The fundamental question is not "can user U perform action A on object O?" but rather "does user U have relation R with object O?"

**Key principles**:
- Start with resources, not roles
- Relations describe connections, permissions derive from relations
- Keep models simple - complexity is the enemy of security
- Plan for growth but don't over-engineer

## Design Workflow

### Phase 1: Requirements Discovery

Ask these questions to understand the domain:

**1. Core Resources**
- "What are the main things users interact with?" (documents, projects, devices, etc.)
- "Do these resources have hierarchies?" (folders contain documents, orgs own repos)

**2. Access Patterns**
- "Describe ONE critical permission in plain language: 'A user can [action] a [resource] IF [conditions]'"
- "What are the different levels of access?" (view, edit, admin, etc.)
- "Do higher access levels include lower ones?" (editors can also view)

**3. User Organization**
- "How are users organized?" (teams, departments, organizations)
- "Can users belong to multiple groups?"
- "Do groups nest?" (team within department within org)

**4. Special Cases**
- "Can access be shared publicly?"
- "Do you need to block specific users?"
- "Is access time-limited or conditional?"
- "Do you need dynamic/custom roles defined at runtime?"

**5. Multi-tenancy**
- "Is this multi-tenant?" (multiple orgs, each with own data)
- "Can users belong to multiple tenants?"

### Phase 2: Model Construction

Follow the six-step process for EACH major feature:

**Step 1**: Document feature in plain language
```
"A user can view a document IF they are a viewer, editor, or owner of the document,
OR if they are a viewer of the parent folder"
```

**Step 2**: Extract object types (nouns)
```
document, folder, user
```

**Step 3**: Extract relations (verbs/connections)
```
viewer, editor, owner, parent
```

**Step 4**: Define in DSL
```
type document
  relations
    define parent: [folder]
    define owner: [user]
    define editor: [user] or owner
    define viewer: [user] or editor or viewer from parent
```

**Step 5**: Write test assertions

**Step 6**: Iterate for next feature

### Phase 3: Pattern Selection

Select patterns based on requirements:

| Requirement | Pattern | Reference |
|-------------|---------|-----------|
| Basic user access | Direct relationships | building-blocks.md |
| Permission hierarchy (editor includes viewer) | Concentric relationships | building-blocks.md |
| Group-based access | Usersets | building-blocks.md |
| Folder/container inheritance | Object-to-object | building-blocks.md |
| Deny specific users | Blocklists (but not) | building-blocks.md |
| Public/anyone access | Public access (user:*) | building-blocks.md |
| Must satisfy multiple conditions | Multiple restrictions (and) | building-blocks.md |
| Separate roles from permissions | Roles vs Permissions | advanced-patterns.md |
| Tenant-defined roles at runtime | Custom Roles | advanced-patterns.md |
| Time/IP/attribute-based access | Conditions (ABAC) | advanced-patterns.md |
| JWT claims, request context | Contextual Tuples | advanced-patterns.md |
| Subscription/feature access | Entitlements | advanced-patterns.md |
| Large multi-team models | Modular Models | advanced-patterns.md |

For complete examples: see examples.md (Google Drive, GitHub, IoT, Slack, SaaS).

### Phase 4: Validation Checklist

Before finalizing, verify:

- [ ] **Concentric relations**: Higher roles include lower ones (owner -> editor -> viewer)
- [ ] **Derived permissions**: App checks permissions, not roles (check `can_delete`, not `admin`)
- [ ] **No redundant tuples**: If editor implies viewer, don't write both tuples
- [ ] **Unique IDs**: Use real IDs, not display names (`user:uuid` not `user:bob`)
- [ ] **Test coverage**: Assertions for each relation and object type
- [ ] **Simplicity**: Can the model be simplified without losing capability?

## Anti-Patterns

**1. Role explosion**: Creating roles like `project_a_editor`, `project_b_editor`. Use usersets instead.

**2. Checking roles, not permissions**: Application code checks `is_admin` instead of `can_delete`. Roles change; permissions are stable.

**3. Missing concentric relations**: Forcing duplicate tuple writes because `editor` doesn't imply `viewer`.

**4. Over-reliance on conditions**: If a relationship is mostly static, store it as a tuple rather than computing via conditions.

**5. Flat hierarchies**: Modeling everything with direct relationships when parent-child (`from`) would be cleaner.

**6. Premature custom roles**: Using dynamic custom roles when fixed roles would suffice.

## Growth and Evolution

**Adding new permission levels**:
```
# Before
define viewer: [user] or editor

# After - add commenter between viewer and editor
define viewer: [user] or commenter
define commenter: [user] or editor
```

**Adding new object types**: Add new type, define relations, add parent relations to existing types if needed.

**Migrating relations**: Never remove relations that have tuples - deprecate first, migrate data, then remove.

**Modular growth**: When model exceeds ~500 lines or multiple teams contribute, split into modules with `fga.mod`.

## Output Format

When presenting a model design:

```
## Authorization Model

### Object Types
- [list types with brief description]

### Key Relations
- [explain important relations and why]

### Design Decisions
- [explain tradeoffs made]

### Model Definition
[DSL code block]

### Example Tuples
[sample relationship tuples]

### Test Cases
[key assertions to verify]
```

## References

- **Building Blocks**: references/building-blocks.md - Core patterns (direct, concentric, object-to-object, usersets, blocklists, public access, AND logic)
- **Advanced Patterns**: references/advanced-patterns.md - Roles vs permissions, custom roles, conditions, contextual tuples, entitlements, modular models
- **Examples**: references/examples.md - Complete models for Google Drive, GitHub, IoT, Slack, SaaS entitlements
