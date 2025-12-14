# OpenFGA Advanced Patterns

## Table of Contents
1. [Roles vs Permissions](#roles-vs-permissions)
2. [Custom Roles](#custom-roles)
3. [Conditions (ABAC)](#conditions-abac)
4. [Contextual Tuples](#contextual-tuples)
5. [Entitlements](#entitlements)
6. [Modular Models](#modular-models)

---

## Roles vs Permissions

**Roles** are assigned (editor, owner). **Permissions** are derived (can_view, can_delete).

```
type trip
  relations
    define owner: [user]
    define viewer: [user]
    define booking_adder: owner
    define booking_viewer: viewer or owner
```

**Benefit**: Application checks permissions, not roles. Role changes don't require code changes.

**Pattern**: Always check fine-grained permissions in your application, not roles directly.

---

## Custom Roles

Allow users to create arbitrary roles at runtime (not baked into the model).

```
type role
  relations
    define assignee: [user]

type asset
  relations
    define viewer: [user, role#assignee] or editor
    define editor: [user, role#assignee]
```

**How it works**:
1. Create role object: `role:content-manager`
2. Assign users: `user:anne` as `assignee` of `role:content-manager`
3. Grant role permissions: `role:content-manager#assignee` as `editor` of `asset:images`

**When to use**: Multi-tenant apps where each tenant defines their own roles.

---

## Conditions (ABAC)

Add attribute-based conditions to relationships.

```
model
  schema 1.1

type user

type document
  relations
    define viewer: [user with non_expired_grant]

condition non_expired_grant(current_time: timestamp, grant_time: timestamp, grant_duration: duration) {
  current_time < grant_time + grant_duration
}
```

**Tuple with condition**:
```json
{
  "user": "user:anne",
  "relation": "viewer",
  "object": "document:roadmap",
  "condition": {
    "name": "non_expired_grant",
    "context": {"grant_time": "2024-01-01T00:00:00Z", "grant_duration": "720h"}
  }
}
```

**Check with context**:
```json
{"context": {"current_time": "2024-01-15T00:00:00Z"}}
```

**Use cases**: Time-based access, IP restrictions, feature quotas.

**Limits**: 32KB tuple context, 512KB request limit.

---

## Contextual Tuples

Temporary tuples sent with requests, not persisted.

**Use case**: Pass JWT claims as authorization context without syncing to OpenFGA.

```json
{
  "contextual_tuples": [
    {"user": "user:anne", "relation": "member", "object": "group:marketing"}
  ]
}
```

**Benefits**:
- No write/read/delete cycles
- Handles parallel requests without conflicts
- Models truly dynamic conditions

**Constraints**: Only work with Check and ListObjects APIs.

**Pattern for org context**:
```
type organization
  relations
    define member: [user]
    define user_in_context: [user]
    define project_editor: member and user_in_context
```

Pass `user_in_context` as contextual tuple based on user's current org selection.

---

## Entitlements

Model feature access through subscription tiers.

```
type feature
  relations
    define associated_plan: [plan]
    define access: subscriber_member from associated_plan

type plan
  relations
    define subscriber: [organization]
    define subscriber_member: member from subscriber

type organization
  relations
    define member: [user]
```

**Access chain**: user → org member → org subscribes to plan → plan has feature → user can access feature.

**Benefit**: No manual user-feature mappings. Subscription changes automatically propagate.

---

## Modular Models

Split large models across multiple files for team ownership.

**fga.mod** (manifest):
```yaml
schema: '1.2'
contents:
  - core.fga
  - issue-tracker/projects.fga
  - issue-tracker/tickets.fga
  - wiki/wiki.fga
```

**Type Extensions**: Add relations to existing types from other modules:
```
module wiki

extend type organization
  relations
    define can_create_space: member
```

**Benefits**:
- Clear ownership boundaries
- Enables code review with CODEOWNERS
- Scalable for large organizations

**Deployment**: `fga model write --file fga.mod`
