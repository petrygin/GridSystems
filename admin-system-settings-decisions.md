# Admin · System Settings — Design Decisions

**Task:** GRDM-120 — `[DSGN] Admin: System Settings Design`
**Scope:** The three System Settings tabs of the Admin App — **Company**, **Infrastructure**, **RBAC**.
**Prototype:** `admin-settings.html` (single file, three tabs, built on the `gridmodel-ui` design system).
**Sources of truth:** `05_Features/Admin/Spec.md` (overview) + the three feature specs under `05_Features/Admin/System/{Company,Infrastructure,RBAC}/Spec.md`.

---

## 0. How scope was read (methodology)

These decisions follow a fixed rule for resolving conflicts between documents, because the specs went through several reconciliations during the task:

- The **feature-level Spec.md** is authoritative for a feature's *behaviour and model*. The **Admin overview Spec.md** owns the *field catalogue* (the field tables and business data types). Where they disagree, the feature spec wins for that feature.
- Each feature spec's **"Not in scope" table** is the real boundary. It also names where each excluded concern is tracked.
- The **Jira ticket narrative is secondary** to the specs linked in the ticket header. The BA links the canonical feature specs in the GRDM-120 header; those links — not the ticket body — define scope.
- Spec deviations are **flagged explicitly**, never applied silently. Where the design departs from the spec's *presentation* (not its data), this is called out below and marked for BA confirmation.

---

## 1. Global patterns (cross-cutting all three tabs)

**Single-screen, view-and-save (Company, Infrastructure).** Each tab is one configuration record. One Read opens the screen; one Update saves it. All blocks on the tab are read together and saved together — there is no per-block save. Configuration is seeded at deploy; only Read and Update are exposed (no create/delete).

**Contextual save bar (Company, Infrastructure).** A single screen-level save control replaces per-card Save/Cancel buttons. It is a fixed bar pinned to the bottom of the content, hidden while the form matches server state, and it slides up the moment any field on any block diverges (disabled-until-dirty). Cancel re-fetches server state and hides the bar. The slide-up animation is to be finalised in the implementation pass.

**Editing in place.** Most fields are editable directly; some tables enter editing from an Edit or row control before becoming inline. "Inline everywhere" is the default rather than a strict rule.

**Secrets model.** A secret (SSO client secret, map/LLM API keys, MCP host secrets) is entered once and masked thereafter. What the configuration persists is a **reference** (e.g. a vault path), never the clear-text value. The value is never fetched or shown after entry and never returned to the client. The Admin App holds only the masked placeholder and the reference; storage/rotation/resolution live outside it.

**Layout vocabulary.**
- **Card per block** — each configuration concern is one card.
- **Drawer (side panel)** — for editing a complex sub-item in place (file-attachment rule, role assignment).
- **Modal** — for transactional one-time actions (issuing an MCP secret) and for the role editor (800 px).
- **Structure editor (`claim → target` rows)** — the reusable "map-row" list pattern for key→value structures (SSO role/attribute mapping, per-attribute unit exceptions). Rows are addable and removable.

**RBAC uses a different save model.** Unlike Company/Infrastructure, RBAC is not a single full-tab save. Roles are saved per role (in the role modal); assignments are written per grant. The contextual-save-bar/disabled-until-dirty logic is intentionally not applied to RBAC.

---

## 2. Page — Company

The Company tab is the installation's organisation context. Three blocks, single seeded record, Read + Update only.

### 2.1 Company Identity & branding

Fields exactly per spec: **Company name** (required), **Email**, **Website URL**, **Location** (country), **Timezone**, **Logo** (PNG/SVG, ≤ 2 MB, stored as a file reference handled by the File Attachments service). All inline-editable.

*Decision:* the earlier prototype's invented blocks (Company Metadata, Account Information) are removed — they are not in the spec. Identity is reduced to the six spec fields.

### 2.2 Display units

- **Display system** — single-select Metric / Imperial. Values are always stored in SI; this preference affects display only.
- **Per-attribute exceptions** — a **Structure**, not a boolean. A toggle enables the section and reveals an editable list of `attribute → display unit` overrides (rows with add/remove). This replaces the earlier bare on/off toggle, which could not hold per-attribute overrides.

### 2.3 Workflow (DocFlow) — read-only

A locked, view-only presentation of the object lifecycle: object-type tabs (Project / Task / Component), the default lifecycle chain, and the allowed transitions. Editing statuses/transitions is owned by the **Model · Workflow Statuses** feature and is out of scope here.

*Note:* the statuses shown in the prototype are illustrative pending the canonical lifecycle from `StatusModel.md`.

### 2.4 Save model

A single contextual save bar covers Identity + Display units. Workflow is read-only and not part of the save.

---

## 3. Page — Infrastructure

Infrastructure is the platform-level configuration of the installation. It is **exactly nine blocks** (the interim "five plumbing cards" model — Database/Cache/Storage/Messaging — was superseded and removed). The overview owns the field tables; the feature spec adds the behaviour. Single seeded config; Read + Update only; all nine blocks saved together.

### 3.1 Platform domain
**Platform domain** (required, URL) and **API base URL** (redirect URLs, email links, API callbacks). Non-secret.

### 3.2 Authentication / SSO
One identity provider per installation; SSO is the only authentication method (there is password-free login). Fields: **Provider** (Okta / Google Workspace / Microsoft Entra ID / Custom SAML, plus OIDC/LDAP), **Client ID**, **Client secret** (masked Secret), **Provider domain**, **Authorization server**, **Redirect URI** (read-only, generated, copyable), **Allowed logout URL**, and **Role & Attribute Mapping** (structure: `provider claim → GridModel target` rows, addable/removable).

**Actions:** configure, enable/disable, **Test connection**.

**Test connection — result states (designed):**
- *Loading* — "Testing connection…".
- *Success (positive)* — green: provider responded, redirect URI verified, claims received.
- *Failure (negative)* — red: failure reason (e.g. provider returned 401 — invalid client secret) with a corrective hint.

### 3.3 VPN / Network access
**Allowed IP ranges** (CIDR, one per line) and **Allow admin bypass from any IP**.

**Current-IP status (designed, live against the CIDR list):**
- *Positive* — green dot: "Your current IP `203.0.113.42` — inside the allowed ranges."
- *Negative* — amber dot: "… not covered by these ranges", plus a **lockout warning** ("Saving now could lock you out of the admin app from this network") and an **Add my IP** quick action that appends the current IP and returns the status to positive.

This is the self-lockout safeguard made visible at the point of editing.

### 3.4 Map tiles source
**Map provider** (OpenStreetMap default / Mapbox / Custom URL), **Tile URL template**, **API key** (masked Secret), **Coordinate scheme** (XYZ / TMS), **Min / Max zoom**. Actions: configure, **Preview map**, enable/disable. Map rendering itself is owned by the Map / SLD Editor feature.

### 3.5 Global platform settings
**Session timeout** (minutes), **Audit logging** (Yes/No), **Maintenance mode** (blocks non-admin access during maintenance).

### 3.6 AI / LLM provider
**Provider** (Anthropic / OpenAI / Gemini / vLLM / GLM), **Default model**, **API key** (masked Secret), **Base URL** (for self-hosted providers), **Active**. The AI panel behaviour is owned by its own feature.

### 3.7 AI / MCP host secrets
A table of credentials MCP hosts present to the system: **Label**, **Binding** (the user or role the secret acts as), **Status** (Active / Revoked), **Expiry**. **Issue** and **Revoke** are the only lifecycle actions. Issue uses a two-step modal (configure → copy now); the value is shown once and never stored or displayed afterward.

### 3.8 File attachments
Rules per entity type (Project / Task / Component / Zone): **File count cap**, **Document categories** (multi-select), **Category required**, **Max file size**, **Allowed file types** (multi-select). Edited in a side drawer.

### 3.9 Seed data import
Bootstraps reference data/libraries from prepared files: **Seed data set** (file), **Data type**, **Source format** (SAP / MS Project / XLS-CSV / AutoCAD / CIM — read as a file format, not an online connector), and **Preview import**. Preview is a dry-run that opens an **Import preview** modal with Created / Updated / Skipped / Issues tiles, severity-tagged (Error/Warning) rows, Download report, and Apply. Run mechanics are owned by the Dataset Seed Runner feature.

### 3.10 Save model
A single contextual save bar saves the whole screen. Block-level actions (Test connection, Preview map, Issue secret, Preview import) are distinct from save and remain on their cards.

---

## 4. Page — RBAC

Role-based access control. **Three blocks:** Roles & permissions, User assignment, Notification defaults per role. The scope model is **allowed zones + allowed tags** (flat lists) per the feature spec — the overview's older "aggregation level / regional scope" model is not used.

### 4.1 Roles & permissions — list

A table with columns **Role · Description · Zones · Tags**. The Type (System/Custom) column was removed; the system distinction is shown inside the role editor instead. Four seeded roles: **System Administrator** (system role, cannot be deleted), **Planning Engineer**, **Planning Manager**, **Model Manager** (the canonical matrices/scopes are owned by GRDM-750).

### 4.2 Role editor — 800 px modal

The role editor is a centred **modal, 800 px wide**, with a pinned header (editable role name + System badge + Delete + close) and a scrollable body. Delete is hidden for system roles. The body has four sections:

**Description** — name, description, and a system/custom note.

**Permission matrix — split into two tables** so every cell is meaningful:
- *Object permissions* — object types × **Create / Read / Update / Delete**. Per a PM requirement, with per-zone / per-tag granularity: data objects (Project, Component, Connection, Verification, File Attachment) **expand into a Zones / Tags tree** and then into individual zones/tags, where CRUD is set per item. Global objects (Model, Library, User, Role, Zone, Tag) stay flat — they have no zones/tags, so a tree would be recursive.
  - The object row is the default baseline; per-zone/tag rows are overrides on top of it.
  - The group-level checkbox (Zones / Tags) is a **rollup + bulk control**: it shows an indeterminate "—" when children differ, and clicking it **sets the value for all zones/tags at once** (e.g. "grant Update to every zone").
  - The zones/tags in the tree are the allowed universe from the Scope dropdowns; the tree is an optional narrowing on top of it.
  - *Complexity note:* the model is heavy (three dimensions: object × action × zone/tag) and is recorded as a PM requirement; the base spec assumed a flat per-role scope.
- *Workflow actions* — **Project and Task only** × **Approve / Merge / Force Approve**. Workflow actions apply only to objects with a review lifecycle, so Tag/Role/Library etc. no longer carry meaningless approve/merge cells.

Three rules are surfaced as hints: Approve applies only where the role's holder is named on the project's review; Merge applies only to an APPROVED project; **Force Approve** is an admin override that approves bypassing the normal review gates and is granted sparingly.

**Scope** — **Allowed zones** + **Allowed tags** as two chip lists with add. Each dropdown carries a description spelling out the universe→narrow relationship (permissions apply across all selected by default; expand an object in the matrix to grant different rights per zone/tag). An empty list carries the per-role meaning defined in the seed (GRDM-750).

**Notifications — group table inside the role.** Notification defaults live inside the role editor, not as a separate page-level block. A single **Scope** dropdown sets the breadth for the role (All / Own region / Own project / Own task / Own data / Own account / None). Below it, a table has one row per domain group: **Project flow, Validations, Comments, Access & roles, Permissions, System** (sample, pending the notifications spec). Columns: **Active**, **Mandatory**, and the channels **In-app / Email / Push**.
- Channel cells are tri-state: a "—" indicates the events inside that group differ; a check means all on. Per-event detail was collapsed to group rows for now to keep the surface simple.
- **Active is the row master:** turning a group's Active off disables and dims its Mandatory and channels.
- *Field semantics:* *Mandatory* = the user cannot disable it in personal preferences; *Active* = whether the group's rules are in effect.

Footer: Cancel / Save.

### 4.3 User assignment

A table of **User** (avatar + name + email) and **Roles**. Effective access is the **union** of the assigned roles, each within its own scope. There is no regional restriction on the assignment — scope is a property of the role.

**Provisioning is via SSO.** Users are not created here: they are provisioned by the identity provider (JIT on first SSO login, or directory sync). Therefore:
- Editing an existing assignment shows a **read-only SSO identity** (avatar/name/email) with a "managed by your identity provider" note; only roles are editable.
- **+ Assign user** opens a **picker over existing provisioned users** (search by name/email), instead of a free email field.

### 4.4 ProjectShare

ProjectShare exists in the model but the `enableProjectSharingCheck` flag is disabled (open question, GRDM-302); it is not surfaced as a feature in this UI.

---

## 5. Out of scope / deferred (per specs)

- Per-block save; configuration revision history; configuration create/delete (single seeded config per installation).
- Secret storage, rotation, and resolution (held outside the Admin App; the app keeps only the masked value and its reference).
- Editable workflow statuses/transitions (owned by Model · Workflow Statuses; here view-only).
- Notification message templates (the message bodies per category).
- Live import connectors to external systems (seed import is file-based only).
- The full RBAC object/action catalogue and the concrete seeded role matrices (RBAC epic + GRDM-750).
- In-app log viewing (delegated to an external system).
- Endpoint paths, request/response shapes, and storage layout (each block's API contracts).

---

## 6. Prototype reference

`admin-settings.html` — one file, three tabs (Company / Infrastructure / RBAC), opening on Company, built with the `gridmodel-ui` system (Geist, Lucide icons, dense light theme, tokens only). Interactive demos included for review: contextual save bar (Company/Infrastructure), SSO Test connection (success/failure), VPN current-IP status (positive/negative + Add my IP), add-exception / add-mapping structure rows, the 800 px role modal, and the notifications group table with Active master dimming.
