# OpenFGA Building Blocks

## Table of Contents
1. [Direct Relationships](#direct-relationships)
2. [Concentric Relationships](#concentric-relationships)
3. [Object-to-Object Relationships](#object-to-object-relationships)
4. [Usersets (Groups)](#usersets-groups)
5. [Blocklists (Exclusion)](#blocklists-exclusion)
6. [Public Access](#public-access)
7. [Multiple Restrictions (AND)](#multiple-restrictions-and)

---

## Direct Relationships

User has access without depending on other relationships.

```
type document
  relations
    define viewer: [user]
    define editor: [user]
```

**When to use**: Base-level access grants to individual users.

**Disable direct access** by removing type restrictions to force access through roles only:
```
define can_delete: editor  # No [user], only derived from editor
```

---

## Concentric Relationships

Higher-privilege roles inherit lower-privilege capabilities.

```
type document
  relations
    define viewer: [user] or editor
    define editor: [user] or owner
    define owner: [user]
```

**Chain**: owner → editor → viewer (owners are automatically editors and viewers).

**When to use**: Hierarchical permission levels where higher roles include lower ones.

---

## Object-to-Object Relationships

Objects relate to objects, enabling hierarchical inheritance.

```
type folder
  relations
    define viewer: [user]

type document
  relations
    define parent: [folder]
    define viewer: [user] or viewer from parent
```

**Key syntax**: `viewer from parent` means "anyone who is a viewer of my parent folder".

**Use cases**:
- Folders containing documents
- Organizations owning repositories
- Plans containing features

**Constraint**: `from` only works with concrete objects, not rewrites or wildcards.

---

## Usersets (Groups)

Assign permissions to groups rather than individuals.

**Notation**: `object#relation` represents all users with that relation to that object.
- `team:engineering#member` = all members of engineering team
- `org:acme#admin` = all admins of acme org

```
type team
  relations
    define member: [user]

type document
  relations
    define editor: [user, team#member]
```

**Tuple**: `team:writers#member` as `editor` of `document:roadmap` grants all writers edit access.

---

## Blocklists (Exclusion)

Deny access even when users have permissions through other paths.

```
type document
  relations
    define blocked: [user]
    define editor: [user, team#member] but not blocked
```

**Operator**: `but not` excludes specified userset.

**Use cases**: Block specific users, implement deny lists, override group permissions.

---

## Public Access

Grant access to everyone using type-bound wildcards.

```
type document
  relations
    define viewer: [user, user:*]
```

**Tuple**: `user:*` as `viewer` of `document:public-doc` = anyone can view.

**Constraints**:
- `type:*` only valid in user field, not object field
- Cannot use in usersets: `org:*#member` is invalid

---

## Multiple Restrictions (AND)

Require multiple conditions simultaneously.

```
type document
  relations
    define owner: [organization]
    define writer: [user]
    define can_delete: writer and member from owner
```

**Operator**: `and` creates intersection - user must satisfy ALL conditions.

**Use case**: "Only writers who are also members of the owning organization can delete."
