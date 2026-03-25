# Identity — Universal User Management

A framework for user identity, authentication, authorization, and entitlement management—domain-agnostic, designed to minimize barrier-to-entry and enable viral uptake.

**Scope:** User, role, permission, entitlement, authentication, authorization.

---

## 1. Universal Identity

### Silos by Default, Discriminators Enable Choice

| Concept | Meaning |
|---------|---------|
| **Logical discriminators (default)** | Discriminators that silo users are introduced by default into a common/shared schema. Users are partitioned logically by discriminator (e.g., `tenant_id`, `app_id`, `scope`). |
| **Shared user schema** | User records live in an application-agnostic schema; app-specific data (roles, entitlements) references this shared identity. Discriminators reside in this schema. |
| **Commuting identity** | One identity works across applications when discriminators are ignored; participation in App A implies recognition in App B where policy allows. |

**Principle:** Identity is owned by the person, not the application. Applications consume identity; they do not create or own it in silos.

### Partition Model: Logical and Physical Silos

Commuting identity is foundational—it enables shared infrastructure and shared identities across domains. Partitioning is achieved in two ways:

| Partition type | Description | When to use |
|----------------|--------------|-------------|
| **Logical silos** | Domain discriminator (e.g. `tenant_id`, `app_id`, `scope`). Same or separate identity runtime instances. Same core libraries, shared or partitioned schema. | **Default.** |
| **Physical silos** | Independent datastore; same core libraries; separate runtime instances. | Regulation/policy motivated (multitenant). Opt-in. |

**Commuting across domains:** To share identities across domains, **white-list domain discriminators** for specific domains. When configured, a principal in domain A is recognized in domain B. The default is strict tenant scope (no sharing); white-listing enables commuting where policy allows.

Both choices are strategic, not technical. The default is logical silos; physical isolation is opt-in for regulation.

### Identity vs. Participation

| Entity | Purpose |
|--------|---------|
| **User** | The person. Identifiers (email, phone). Shared schema. Reusable across apps. |
| **Participant / Role** | What the user *does* in a context (contributor, viewer, member). One user, many participation roles. |
| **Entitlement** | What the user or participant may do or access. Derived from role, subscription, or policy. |

Identity is who you are. Participation and entitlement are what you can do.

---

## 2. Roundtrip Authentication

**Roundtrip authentication:** User proves control of an identifier (email or phone) by completing a roundtrip—receive a one-time code or magic link, enter it, session created. No password. No sign-up form.

| Step | Action |
|------|--------|
| 1. Provide identifier | User enters email or phone |
| 2. Request code | System sends OTP or magic link |
| 3. Enter code | User receives and enters code (or clicks link) |
| 4. Session created | User is recognized |

Effectively: passwordless authentication; the second factor of 2FA without a first factor.

---

## 3. Participation-First: No Sign-Up

A person is a **user** when they participate—not when they "sign up" or "register."

| Principle | Meaning |
|-----------|---------|
| **Participation creates identity** | The first participation act (identifier + roundtrip) establishes the user. No separate "create account" flow. |
| **Identity is retrofitted** | User records are created or augmented as the person engages. Attributes attached when provided—not required upfront. |
| **Augmenting is optional** | Additional attributes (name, preferences) can be requested over time; never prerequisite for participation. |

**Principle:** Value first, identity second. Deliver something useful immediately; collect and refine identity through engagement.

### Flow

1. User has reason to engage (invited, shared link, organic encounter).
2. User provides identifier and completes OTP roundtrip.
3. User is recognized. Session established. Participation begins.
4. Optional: augment profile, link accounts—after initial participation.

No sign-up, no registration, no "join" action required.

---

## 4. Barrier-to-Entry and Viral Mechanics

**Strategy:** Remove the sign-up wall. The first value-delivering action is the identity-establishing action.

| Mechanic | Description |
|----------|-------------|
| **Share-to-participate** | A shares with B. B completes roundtrip. B is in. |
| **Organic onboarding** | Real-world encounter; to complete in product, one roundtrip. No "join" feeling. |
| **Install-to-act** | Value gated by light action (install, identifier, verify). Minimal friction, maximum intent. |
| **Network visibility** | New users see broader network (inviter's connections, other participants). Pool compounds; value is immediate. |

Each new user expands the visible pool for all participants. Reciprocal visibility reduces "empty network" perception. See [[VIRAL]] for the transactional invite pattern that enables share-to-participate flows.

---

## 5. Sessions, Tokens, Persistence

| Requirement | Description |
|-------------|-------------|
| **Bearer tokens** | Session represented by token in `Authorization` header. Stateless from client perspective. |
| **Server-side validation** | Each request validates token. Session store holds `user_id`, scopes, metadata. |
| **Persistence across restarts** | Sessions survive server restarts. Valid until logout or expiry. |
| **Validation endpoint** | `GET /session` (or equivalent) validates token and returns metadata (user_id, roles, entitlements). |

Client stores token securely; on load, validates with server before treating as authenticated. Explicit logout invalidates server-side; client clears token.

---

## 6. Roles, Permissions, Entitlements

- **Roles:** What the user does in a context. Implicit (from participation) or explicit (admin/policy assigned). Scoped to app, tenant, or context.
- **Permissions:** Fine-grained capabilities (edit, delete, invite). Derived from roles or assigned directly.
- **Entitlements:** What the user may access—features, quotas, tiers. From role, subscription, policy, or explicit grant.

**Principle:** Entitlements are computed at decision time. Avoid hardcoding "user type" in business logic; use roles and entitlements.

### Authorization Layers

| Layer | Responsibility |
|-------|-----------------|
| **Authentication** | Who is this? (Identity + session.) |
| **Authorization** | What may they do? (Roles, permissions, entitlements.) |
| **Enforcement** | Check at each protected resource. Deny by default. |

---

## 7. Implementation

- **Identifiers:** Email and/or phone. OTP or magic link. Federation (OAuth/OpenID) when integrating existing providers.
- **OTP:** Short-lived (e.g., 10 min). Single-use. Rate-limited.
- **Schema:** User table (`user_id`, `email`, `phone_number`, `created_at`, `updated_at`, optional `external_ids`). Application tables reference `user_id`.
- **Session schema:** `session_id`, `user_id`, `created_at`, `expires_at`, optional `tenant_id`, `app_id`, `scope`.

---

## 8. Requirements (UU-01 through UU-08)

| ID | Requirement |
|----|-------------|
| UU-01 | Identity SHALL commute across applications; partitioning SHALL be opt-in, not default. |
| UU-02 | Users SHALL be modeled in a separate, application-agnostic schema. |
| UU-03 | Authentication SHALL use identifier + OTP (or magic link) roundtrip; no password required. |
| UU-04 | A person SHALL be a user when they participate; identity SHALL be retrofitted, not required upfront. |
| UU-05 | Sign-up, registration, and join SHALL NOT be required; roundtrip SHALL establish recognition. |
| UU-06 | Sessions SHALL persist across restarts; token validation SHALL be supported. |
| UU-07 | Roles, permissions, entitlements SHALL be modeled separately from identity; authorization SHALL enforce at decision time. |
| UU-08 | Design SHALL minimize barrier-to-entry; value delivery SHALL precede identity augmentation. |
