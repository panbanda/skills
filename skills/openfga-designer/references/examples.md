# OpenFGA Real-World Examples

## Table of Contents
1. [Google Drive](#google-drive)
2. [GitHub](#github)
3. [IoT Security System](#iot-security-system)
4. [Slack Workspace](#slack-workspace)
5. [SaaS Entitlements](#saas-entitlements)

---

## Google Drive

**Patterns used**: Concentric relationships, object-to-object (parent), public access, usersets (domains).

```
type user

type domain
  relations
    define member: [user]

type folder
  relations
    define owner: [user, domain#member]
    define writer: [user, domain#member] or owner
    define commenter: [user, domain#member] or writer
    define viewer: [user, user:*, domain#member] or commenter

type document
  relations
    define parent: [folder]
    define owner: [user, domain#member] or owner from parent
    define writer: [user, domain#member] or owner or writer from parent
    define commenter: [user, domain#member] or writer or commenter from parent
    define viewer: [user, user:*, domain#member] or commenter or viewer from parent
```

**Key features**:
- `user:*` enables public sharing
- `domain#member` enables org-wide sharing
- `from parent` cascades folder permissions to documents

---

## GitHub

**Patterns used**: Concentric relationships, usersets (teams), nested teams, org base permissions.

```
type user

type team
  relations
    define member: [user, team#member]

type organization
  relations
    define member: [user, organization#member]
    define repo_admin: [user]
    define repo_writer: [user]
    define repo_reader: [user]

type repository
  relations
    define owner: [organization]
    define admin: [user, team#member, organization#member] or repo_admin from owner
    define maintainer: [user, team#member, organization#member] or admin
    define writer: [user, team#member, organization#member] or maintainer or repo_writer from owner
    define triager: [user, team#member, organization#member] or writer
    define reader: [user, team#member, organization#member] or triager or repo_reader from owner
```

**Key features**:
- `team#member` with nested team membership (`team#member` in team's member relation)
- `repo_admin from owner` implements org-level base permissions
- Concentric: admin → maintainer → writer → triager → reader

---

## IoT Security System

**Patterns used**: Role-based, concentric, device groups, disabled direct access.

```
type user

type device_group
  relations
    define it_admin: [user]
    define security_guard: [user]

type device
  relations
    define device_group: [device_group]
    define it_admin: [user] or it_admin from device_group
    define security_guard: [user] or security_guard from device_group or it_admin
    define live_video_viewer: security_guard
    define recorded_video_viewer: security_guard
    define device_renamer: it_admin
```

**Key features**:
- Permissions (`live_video_viewer`) have no direct type restrictions - access only through roles
- IT admins inherit security guard capabilities
- Device groups cascade permissions to all contained devices

---

## Slack Workspace

**Patterns used**: Workspace roles, channel permissions, guest restrictions.

```
type user

type workspace
  relations
    define legacy_admin: [user]
    define channels_admin: [user] or legacy_admin
    define member: [user] or channels_admin
    define guest: [user]

type channel
  relations
    define workspace: [workspace]
    define writer: [user, workspace#member, workspace#legacy_admin]
    define viewer: [user, workspace#member, workspace#legacy_admin, workspace#guest] or writer
```

**Key features**:
- Role hierarchy: legacy_admin → channels_admin → member
- Guests are separate from members (not in hierarchy)
- Channels inherit from workspace membership

---

## SaaS Entitlements

**Patterns used**: Object-to-object, derived access through subscription chain.

```
type user

type organization
  relations
    define member: [user]
    define admin: [user]

type plan
  relations
    define subscriber: [organization]
    define subscriber_member: member from subscriber
    define subscriber_admin: admin from subscriber

type feature
  relations
    define associated_plan: [plan]
    define access: subscriber_member from associated_plan
    define admin_access: subscriber_admin from associated_plan
```

**Access resolution**:
1. Is user a member of an organization?
2. Does that organization subscribe to a plan?
3. Is this feature associated with that plan?
4. If yes to all: user has access

**Key features**:
- No direct user-feature relationships
- Subscription changes automatically propagate
- Clean separation: orgs subscribe to plans, plans include features
