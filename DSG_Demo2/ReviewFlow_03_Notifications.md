# Task 03 — Notifications (Review Requests inbox)

**Scope:** How reviewers discover review requests assigned to them, and how they enter Review Mode from the inbox.
**Related:** Author-side Send for review and project locking → Task 01. Reviewer picker → Task 02. Review Mode itself → Task 04.

For shared terminology, eligibility rules, and reviewer statuses, see `ReviewFlow_00_Overview.md`.

---

## Goal

Give reviewers a single, persistent place to see all projects assigned to them for review. The page is the entry point into Review Mode (Task 04) and replaces ad-hoc email/inbox dependencies for the in-app experience.

---

## Architecture decision: in-app inbox, not project-scoped panel

The platform has a **project-scoped Notifications panel** (right side panel, accessible inside a project) that shows in-project alerts (validation, conflicts, role changes, etc.). That panel is **not used** for review requests.

Rationale: a reviewer being notified about a new review request is, by definition, **not yet inside that project**. The notification needs to live at the global navigation level, not inside an arbitrary project.

The chosen pattern is a dedicated **left-sidebar item** with a counter badge that takes the user to a list view of all review requests assigned to them.

---

## Sidebar entry

A new item is added to the primary navigation in the left sidebar:

```
…
Dashboard
All projects
Review requests        [N]   ← new item, with counter badge
Master model
Archive
…
```

| Property         | Value                                                                |
|------------------|----------------------------------------------------------------------|
| Label            | `Review requests`                                                    |
| Icon             | `git-pull-request` (Lucide) or design-system equivalent              |
| Counter          | Pill on the right of the row, showing **count of Pending requests for the current user** |
| Counter color    | Warning orange (`#F97316`) — needs attention, not critical           |
| Counter visibility| Hidden when N = 0                                                   |

### What the counter counts

The counter shows the number of review requests where:

- The current user is assigned as a reviewer
- The current user's reviewer status is `Pending`

It excludes:

- Requests the user has already responded to (`Approved` or `Declined` by them)
- Requests cancelled by the author (auto-removed)
- Requests on projects where the current user is not a reviewer

### When the counter updates

The counter decrements when:

- The user gives `Approve` or `Decline` (locally and via row removal)
- The author cancels the review request
- The project's review cycle ends (Approved or Merged across the project)

The counter increments when:

- A new request lists the user as reviewer (new Send for review with this user selected)
- A previously-cancelled request is resubmitted with this user as reviewer

There is no "unread" semantics — visiting the page does **not** decrement the counter. Only acting on a request does.

---

## Page: Review requests

The page opens when the sidebar item is clicked. URL: `/review-requests` (or platform equivalent).

### Layout

```
[Page title: "Review requests"]

[Search ────────────────────]

┌─────────────────────────────────────────────────────────────────────────────────────┐
│ Project          | Sent date ↕ | Changes ↕ | Validation | Sync       | Zones       │
│                  |             |           |            |            |             │
│                  |             |           |            |            |  (cont.)    │
│                  |             |           |            |            |             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│ GridSim West     | 12 min ago  | 12        | ⊘ 21 errors| ● Current  | Downtown LA │
│ TUG · 48291AB    | 11:28 AM    |           |            |            |             │
│ …                                                                                   │
└─────────────────────────────────────────────────────────────────────────────────────┘
                                                  Owner ↕  | Description |
                                                  PP Peter |   📋        | ⋯
```

### Default state (filter, sort)

- **No tabs**: only "Assigned to me". Author-side view of own sent requests is reachable via filter on All projects (out of scope for Task 03).
- **Default sort**: `Sent date` descending (newest at top).
- **Default filter**: only Pending requests for the current user. Already-answered requests do not appear.

### Columns

| # | Column      | Sortable | Content                                                              |
|---|-------------|----------|----------------------------------------------------------------------|
| 1 | Project     | Yes      | Project name (bold) + project ID (e.g., `TUG · 48291AB`) below in mono font |
| 2 | Sent date   | Yes      | Relative time (e.g. `12 min ago`) + absolute time below              |
| 3 | Changes     | Yes      | Number. Cap at `999+` for very large numbers                         |
| 4 | Validation  | No       | Icon + count + label, e.g. `⊘ 21 errors` (red), `⚠ 7 warnings` (orange), `✓ Passed` (green) |
| 5 | Sync        | No       | Colored dot + label: `● Current` (teal), `● 2 conflicts` (red), `● Outdated` (grey) |
| 6 | Zones       | No       | Comma-separated zone names. Overflow as `+N` pill when >2 zones      |
| 7 | Owner       | Yes      | Avatar + first name                                                  |
| 8 | Description | No       | Notepad icon if description exists; empty cell if not                |
| 9 | (Actions)   | —        | `⋯` kebab menu (visible on row hover or when menu is open)           |

**Sortable columns** show a sort indicator (↕ idle, ↑/↓ when active) on the right of the header.

**Validation icons:**

- Errors → filled red circle with exclamation, color `--destructive`
- Warnings → triangle with exclamation, color `--warning`
- Passed → filled green circle with check, color `--success`

**Sync types:**

- `Current` → teal dot
- `N conflicts` → red dot, with conflict count
- `Outdated` → grey dot

### Row interactions

| Action                       | Trigger                                            | Result                                                |
|------------------------------|----------------------------------------------------|-------------------------------------------------------|
| Open in Review mode          | Click anywhere on row (except Description / Kebab buttons) | Opens project in Review Mode (Task 04)        |
| Show description             | Click the notepad icon in Description column      | Opens a tooltip / sidesheet with full description text |
| Open kebab menu              | Click `⋯` in the last column                       | Opens action menu                                     |

### Kebab menu actions

The menu is identical across all rows (action availability may vary by reviewer permissions / project state):

```
👁  Open in Review mode
📋  Show description
─────────────────────────
💬  Reject with comment      ← destructive emphasis (or implied)
✅  Approve
🔀  Merge                   ← disabled (only enabled when project is Approved)
─────────────────────────
ℹ️  Show info
🗂  Activity feed
👥  Manage access
─────────────────────────
🗄  Archive
```

| Action               | Behavior in MVP                                                              |
|----------------------|------------------------------------------------------------------------------|
| Open in Review mode  | Same as row click                                                           |
| Show description     | Same as Description icon click                                              |
| Reject with comment  | Opens a modal: title `Reject review request`, project subtitle, required textarea (`Comment to author`), Cancel + Reject buttons. On confirm: row disappears, author notified, counter decrements. |
| Approve              | Mock-immediate action: row disappears, author notified, counter decrements. (Real flow: see Task 04 / 05 — quick approve from the inbox is an optional shortcut to a full Review Mode action.) |
| Merge                | Disabled in this view by default (becomes enabled only after project is Approved — out of scope for Task 03 to gate this). |
| Show info / Activity feed / Manage access / Archive | Out of scope for Task 03; navigates to the corresponding view if present. |

### Quick approve / decline rationale

Including `Approve` and `Reject with comment` as kebab actions means a reviewer can act on small / familiar / repeat requests **without entering Review Mode**. This is intentional — it shortens the loop for trusted, low-risk changes.

For Task 03 prototype these act on mock data; real implementation in Task 04 / 05 must enforce that a quick-approve still creates the proper TaskPatch / approval signal as if it had been done in Review Mode.

### Empty states

| Situation                              | Display                                                              |
|----------------------------------------|----------------------------------------------------------------------|
| No requests assigned to current user   | Centered: inbox icon + `No review requests` + body `When someone assigns you as a reviewer, requests will show up here.` |
| Search has no matches                  | Centered: search-x icon + `No matches` + body `Nothing matches "<query>". Try a different search.` |

### When the author cancels a review

If the author cancels a review request while the reviewer has the page open, the corresponding row **disappears immediately** on the next render. Counter decrements. No "Cancelled by author" intermediate state is shown — the inbox is for actionable items only.

The user can find audit history in `Activity feed` (separate page, out of scope for Task 03).

---

## Search

A search field above the table filters by:

- Project name
- Project ID
- Owner name
- Zone name

Case-insensitive substring match across these fields. Real-time filtering as the user types.

---

## Out of scope (Task 03)

- Email or push notifications (backend epic)
- Bell icon in top-right header (separate global notification surface, if/when added)
- Right-side project Notifications panel (kept untouched — for in-project alerts)
- "My sent requests" view for authors (covered via filter on All projects)
- Per-row preview of changes (open Review Mode for that)
- Marking as read / unread without acting (counter is action-driven only)
- Bulk actions (select multiple → approve/decline) — not needed at MVP scale
- Saved searches / filters
- Pagination (assume 1 page is enough at MVP scale)
- Real-time push of new requests to the open page (page refresh on next visit is acceptable)

---

## Open questions

None for Task 03. All major decisions resolved:

- Inbox lives in left sidebar, not project Notifications panel ✓
- Label: `Review requests` ✓
- Counter: Pending-for-current-user only, warning-orange (`#F97316`) ✓
- One view: `Assigned to me` only (no tabs) ✓
- Default sort: `Sent date` descending ✓
- Columns: Project / Sent date / Changes / Validation / Sync / Zones / Owner / Description / Actions ✓
- Cancel-by-author = row disappears immediately ✓
- Quick Approve / Reject with comment available from kebab ✓
- Quick Merge present but disabled in this view ✓
- Empty states for both no-requests and no-matches ✓

### Deferred to Task 04

- Behavior of "Open in Review mode" target page (the actual Review Mode UI)
- Whether quick-approve is policy-enforced (e.g. require open + scroll before allowing approve)
- Description sidesheet / modal vs tooltip — Task 03 prototype uses tooltip; final form may live in Task 04

---

## Acceptance criteria

- [ ] `Review requests` item appears in the left sidebar between `All projects` and `Master model`
- [ ] Counter badge shows count of Pending requests for the current user
- [ ] Counter color is warning-orange `#F97316`
- [ ] Counter is hidden when count is 0
- [ ] Page title is `Review requests`
- [ ] Search field filters by project name, project ID, owner, zone (case-insensitive)
- [ ] Default sort: Sent date descending; clicking a sortable header toggles direction
- [ ] Sortable columns: Project, Sent date, Changes, Owner
- [ ] Each row shows all 9 columns including kebab
- [ ] Description column shows icon only when description exists
- [ ] Click on row body opens project in Review Mode (Task 04 entry point)
- [ ] Kebab menu opens on click and contains all listed actions
- [ ] `Merge` action is disabled in the kebab menu
- [ ] `Approve` removes the row, decrements counter, shows confirmation toast
- [ ] `Reject with comment` opens modal with required textarea; Reject button disabled until text entered; on submit removes row, decrements counter
- [ ] Cancel review by author removes the row from this page on next render
- [ ] Empty state for no requests shows inbox icon + helpful message
- [ ] Empty state for no search matches shows distinct message with the query echoed
