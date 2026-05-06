# Task 03 — Notifications (Review Requests inbox + Notifications panel)

**Scope:** Two complementary surfaces for surfacing review-flow events to users:
1. **Review requests page** — action-oriented inbox in left sidebar (primary triage destination)
2. **Notifications panel** — passive pulse of all project events, including review-flow

**Related:** Author-side Send for review and project locking → Task 01. Reviewer picker → Task 02. Review Mode itself → Task 04.

For shared terminology, eligibility rules, and reviewer statuses, see `ReviewFlow_00_Overview.md`.

---

## Goal

Give every participant in the review flow a clear and predictable place to discover events relevant to them:

- **Reviewers** see their pending assignments in one centralized inbox
- **Authors** track progress of their submitted reviews via passive notifications
- Both surfaces work together — they don't duplicate, they complement each other

---

## Architecture decision: two surfaces, distinct purposes

The platform has a **right-side Notifications panel** (accessible inside any project) showing in-project alerts (validation, conflicts, role changes, lifecycle events).

The platform also gets a new **Review requests page** in the left sidebar — a global inbox for review assignments.

| Surface | Purpose | When used |
|---------|---------|-----------|
| Review requests page | Triage pending reviews assigned to me | I want to actively work through my queue |
| Notifications panel | Pulse of all project events (review + others) | I want to stay aware while doing other work |

Notifications panel is global — it works the same whether I'm inside a project or on Dashboard / All projects. Card actions adapt to context (covered later).

---

# Part 1 — Review requests page (sidebar inbox)

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
| Counter          | Pill on the right of the row, count of Pending requests for the current user |
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

- The user gives `Approve` or `Decline`
- The author cancels the review request
- The project's review cycle ends (Approved or Merged across the project)

The counter increments when a new request lists the user as reviewer, or a previously-cancelled request is resubmitted with this user as reviewer.

There is no "unread" semantics — visiting the page does **not** decrement the counter. Only acting on a request does.

---

## Page: Review requests

The page opens when the sidebar item is clicked. URL: `/review-requests` (or platform equivalent).

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

### Row interactions

| Action                       | Trigger                                            | Result                                                |
|------------------------------|----------------------------------------------------|-------------------------------------------------------|
| Open in Review mode          | Click anywhere on row (except Description / Kebab) | Opens project in Review Mode (Task 04)                |
| Show description             | Click the notepad icon in Description column      | Opens a tooltip / sidesheet with full description text |
| Open kebab menu              | Click `⋯` in the last column                       | Opens action menu                                     |

### Kebab menu actions

```
👁  Open in Review mode
📋  Show description
─────────────────────────
💬  Reject with comment
✅  Approve
🔀  Merge                   ← disabled until project is Approved
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
| Approve              | Mock-immediate action: row disappears, author notified, counter decrements. |
| Merge                | Disabled in this view by default (becomes enabled only after project is Approved — out of scope for Task 03 to gate this). |
| Show info / Activity feed / Manage access / Archive | Out of scope for Task 03; navigates to the corresponding view if present. |

### Empty states

| Situation                              | Display                                                              |
|----------------------------------------|----------------------------------------------------------------------|
| No requests assigned to current user   | Centered: inbox icon + `No review requests` + body `When someone assigns you as a reviewer, requests will show up here.` |
| Search has no matches                  | Centered: search-x icon + `No matches` + body `Nothing matches "<query>". Try a different search.` |

### When the author cancels a review

Row **disappears immediately** on the next render. Counter decrements. No "Cancelled by author" intermediate state. Audit history lives in `Activity feed` (separate page, out of scope).

### Search

Search field above the table. Filters by project name, project ID, owner name, zone name. Case-insensitive substring match. Real-time filtering.

---

# Part 2 — Notifications panel (review events)

The right-side Notifications panel surfaces all project events, including review-flow events. Below is the spec for the **review-related cards** only — other categories (Verification, Document Gates, Role & Permissions, etc.) are out of scope here.

## Where review events live in the panel

Review-flow notifications are grouped under the existing **`Project Lifecycle`** category. No new top-level group is introduced — review/approval/merge transitions are part of the project's lifecycle.

## Card visual structure

All review-event cards share a single uniform layout:

```
┌──────────────────────────────────────────┐
│ [Pill]                  10h ago [☐]      │   ← category pill + time + read-marker
│                                           │
│ Body text — single sentence describing    │
│ what happened (≤ ~140 chars).             │
│                                           │
│ [↗ Open project]   [Details ▾]            │   ← single CTA + optional details toggle
│                                           │
│ (expanded details if present)             │
└──────────────────────────────────────────┘
```

### Universal rules

- **Single CTA** on every card: `Open project`. No semantic variants (`Review project`, `Merge to master`) — clicking always lands the user in the project where the actual action happens. Action by user must be deliberate, not driven by a notification button.
- **Pill** carries the category name + color. Category color = semantic meaning (success / warning / info / neutral), not severity.
- **Body text** ends with a period. Single sentence, project name in-flow.
- **Details toggle** appears only when expanded content exists. Cards without expansion data (`Review cancelled`, `Archived`) show only the CTA.
- **Time format**: relative (`10h ago`, `Pending 3d`, `just now`).
- **Read-marker** (`[☐]`) toggles read/unread per card.

## Pill colors — 4 semantic tints

| Pill | Tint              | Used for                                            |
|------|-------------------|-----------------------------------------------------|
| Warning | Peach (warm)   | Action needed: `Review request`, `Changes requested` |
| Success | Mint (green)   | Positive milestones: `Approved N/M`, `Ready to merge`, `Merged` |
| Info | Light blue        | Neutral / FYI: `Review cancelled`                    |
| Neutral | Light grey     | Terminal / muted: `Archived`                         |

Card background uses a lighter tone of the same tint. Pill is the more saturated accent within the card.

## Event copy — collapsed and expanded

### Review request *(warning · peach)*

**Collapsed:**
```
[Review request]                    10h ago [☐]
Peter G. asked you to review GridSim West
Upgrade 2025.
[↗ Open project]    [Details ▾]
```

**Details:**
```
12 changes · 5 added · 4 modified · 3 removed

Voltage regulator upgrade across HV substations.
Coordinate with Sarah for relay settings before
approving — non-standard config in TM-133-T1.
```

The author's description (if provided in Send for Review modal) appears in muted style. No quotes, no `— Peter G.` attribution — context already implies it.

---

### Review cancelled *(info · light blue)* — **no Details**

```
[Review cancelled]                  10h ago [☐]
Peter G. cancelled the review request on
NorthWind Transmission Model.
[↗ Open project]
```

No expansion. The card itself is self-sufficient.

---

### Approved N/M *(success · mint)*

**Collapsed:**
```
[Approved 1/3]                      10h ago [☐]
Sarah Chen approved your review request on
GridSim West Upgrade 2025.
[↗ Open project]    [Details ▾]
```

The pill includes the running tally `N/M` (e.g., `Approved 1/3`) so the author sees progress at a glance.

**Details:**
```
Sarah Chen — Approved
Liam Harper — Pending
Maya Bennett — Pending

13 Comments
```

`13 Comments` is a counter referencing the review thread (Comments thread — separate scope, TBD).

---

### Changes requested *(warning · peach)*

**Collapsed:**
```
[Changes requested]                 10h ago [☐]
Liam Foster requested changes on EastRiver
Substation Replica.
[↗ Open project]    [Details ▾]
```

**Details:**
```
Liam F.: The transformer rating in section 3 doesn't 
match the panel spec on page 17. T-204 is listed as 
50 kVA but the load calc requires 75 kVA. Please 
update or explain why.

Project returned to Draft. Resubmit when changes 
are ready.
```

Reviewer comment is prefixed with `First-name F.:` (no quotes). Closing line in muted style describes the resulting project state.

---

### Ready to merge *(success · mint)*

**Collapsed:**
```
[Ready to merge]                    10h ago [☐]
All reviewers approved GridSim West Upgrade
2025.
[↗ Open project]    [Details ▾]
```

CTA stays `Open project` — merge is a deliberate action, taken inside the project, not from the notification.

**Details:**
```
Sarah Chen — Approved
Liam Harper — Approved
Maya Bennett — Approved

Validation: Passed
Sync: Current — no conflicts with master

24 Comments
```

---

### Merged *(success · mint)*

**Collapsed:**
```
[Merged]                            10h ago [☐]
Sarah Chen merged your project GridSim West
Upgrade 2025.
[↗ Open project]    [Details ▾]
```

**Details:**
```
Merged by Sarah Chen on Mar 14, 11:42 AM
12 changes are now in master
```

---

### Archived *(neutral · light grey)* — **no Details**

```
[Archived]                          10h ago [☐]
Peter G. archived GridSim West Upgrade 2025.
[↗ Open project]
```

No expansion. Read-only project view is reachable through CTA.

---

## Copy conventions (single source of truth)

- **Body text** is one sentence ending in a period. Project name flows inline.
- **Details lines** do **not** end in periods — they are metadata, not narrative.
- **Reviewer status list** uses `Full Name — Status` format (e.g., `Sarah Chen — Approved`). No icons, no pills inside the list.
- **Reviewer comment inline** uses `First-name F.: text` format (short attribution prefix, no quotes).
- **Author description** in Review request expansion appears in muted style without attribution — context implies authorship.
- **Comments counter** appears as `N Comments` in muted style at the end of Details where applicable. Refers to the review thread (separate scope, TBD).
- **CTA** is universally `Open project`. No alternates.

## In-project vs out-of-project context

The Notifications panel is global. The same review-event card looks identical whether the user is on Dashboard, in All projects, or inside another project. The CTA `Open project` always navigates to the project the notification is about.

When the notification refers to the **same project** the user is already inside, the CTA still says `Open project` but acts as a no-op or refresh (it just stays put). This is an edge case — review notifications most often refer to projects the user is not currently in.

---

## Out of scope (Task 03)

- Email or push notifications (backend epic)
- Bell icon in top-right header (separate global notification surface, if/when added)
- Other notification categories (Verification, Document Gates, Role & Permissions, Model State) — review events only
- Reminder notifications for stale Pending reviews — author can see staleness on the Reviews page (sorted by Sent date desc, oldest at top)
- Comments thread inside review (referenced as `N Comments` counter only)
- "My sent requests" view for authors (covered via filter on All projects)
- Per-row preview of changes inside notifications (open the project for that)
- Marking as read / unread (counter is action-driven only on Reviews page; the panel `[☐]` toggle is just a per-card state)
- Bulk actions (select multiple → approve/decline)
- Saved searches / filters
- Pagination (assume 1 page is enough at MVP scale)
- Real-time push of new requests to the open page

## Forward references

- `Comments thread` — referenced via `N Comments` counter in Approved / Ready to merge Details. The actual comments UX (where lives, how to write/reply, mentions) is out of scope for Task 03 and will be specified in Task 04 or a dedicated Comments spec.
- Behavior of "Open project" target page when notification refers to a project in `In review` state — covered by Task 04 (Review Mode).

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
- Notifications panel: review events live under `Project Lifecycle` ✓
- Pill carries the category name + tint; tint = semantic, not severity ✓
- 4 pill tints: warning peach, success mint, info blue, neutral grey ✓
- 7 review events with finalized copy and Details where applicable ✓
- Universal CTA `Open project` on every card ✓
- 2 cards have no Details: `Review cancelled`, `Archived` ✓
- Reminder notifications removed from MVP ✓

---

## Acceptance criteria

### Reviews page

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
- [ ] `Reject with comment` opens modal with required textarea; Reject button disabled until text entered
- [ ] Cancel review by author removes the row from this page on next render
- [ ] Empty state for no requests shows inbox icon + helpful message
- [ ] Empty state for no search matches shows distinct message with the query echoed

### Notifications panel — review events

- [ ] Review-event cards render under the `Project Lifecycle` group
- [ ] All 7 event types render with correct pill, tint, and copy
- [ ] `Review cancelled` and `Archived` cards have no Details toggle
- [ ] Other 5 cards have `Details ▾` toggle that expands inline
- [ ] CTA on every card is `Open project` (single button, no semantic variants)
- [ ] Pill colors match the 4-tint mapping (warning / success / info / neutral)
- [ ] Body text ends with period; Details lines do not
- [ ] Reviewer status list uses `Full Name — Status` format (no icons)
- [ ] Reviewer comment uses `First-name F.: text` prefix
- [ ] Comments counter appears as `N Comments` in muted style
- [ ] Read-marker `[☐]` toggles per-card read state
- [ ] Card layout is identical regardless of user's current location (in-project / out-of-project)
