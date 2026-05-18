# Task 05 — Validation in Review Mode

**Scope:** How the existing Validation panel surfaces inside Review Mode. Project-scoped run history, auto-run on Send for review dialog open, read-only access for reviewers, and the relationship between validation results and the Approve action.

**Related:** Review Mode surface and status popover → Task 04. Send-for-review entry → Task 01. Reviews list inbox → Task 03. Project-vs-master sync state → Task 06.

**Design:** [Figma — Validation in Review Mode](https://www.figma.com/design/0FW49cYCouwjKw3YzqQwcI/Grid-Model-Work-in-Progress?node-id=3283-113999)

For shared terminology, state model, and the validation policy at Send for Review, see `ReviewFlow_00_Overview.md`. For the panel design itself (three views, section grouping, severity treatment, cross-highlight), see `UI-VAL-ValidationResultsPanel.md`.

---

## Goal

Give reviewers a confident read on the technical health of the project they're reviewing:

- A fresh validation snapshot captured at the moment of Send for review
- Full project history of validation runs leading up to that snapshot
- Direct inspection of each finding (cross-highlight to Map / SLD, Info Panel in read-only)

Approving with outstanding errors is allowed — the reviewer carries that decision, not the system.

---

## Architectural framing

Review Mode reuses the existing Validation panel without structural changes: same right drawer, same three views (Validation history / Results collapsed / Results expanded), same grouping by validation stage, same per-row severity, same click-to-cross-highlight.

Task 05 defines wrapper behavior around that panel inside Review Mode:

1. **Run triggers** — when validation runs happen across the lifecycle
2. **History scope** — where runs are stored and who can see them
3. **Access in Review Mode** — what users can do once the project is locked
4. **Approve relationship** — how results connect (or don't connect) to the Approve action

Entry point is unchanged across modes: the Validation icon (with severity badge) in the project header opens the panel.

---

## Run triggers

Validation runs come from two paths:

| Event                              | Validation behavior                                                          |
|------------------------------------|------------------------------------------------------------------------------|
| Author works in Draft              | Manual `[Verify]` adds an entry to project history                           |
| Author opens Send for review dialog | Backend runs validation in the background; result populates the dialog's warning banner (when errors exist) and is committed to history on send as the review snapshot |
| Reviewer or watcher in Review Mode | Read-only access to project history; no new runs                             |

The backend run triggered by opening the Send for review dialog guarantees that the snapshot reviewers receive is fresh against the current master state at the moment of handoff. The author sees the same fresh result inside the dialog (validation banner with error count) so they make the send decision with current information, not against whatever the author last validated.

If validation is still in progress when the author clicks Send for review, the send action waits for the run to complete before committing. The dialog's primary button stays enabled but reflects the in-progress state until ready.

If the author cancels the dialog, the run is discarded — it does not pollute the project history.

### Stale state

The project model is locked while in Review Mode, so runs cannot become stale from within the project. The single trigger for staleness is **master moving** between the snapshot moment and the reviewer's current session — i.e., another project merged in the meantime.

When this happens, the Validation panel header shows `Results may be outdated — master updated`. The indicator is informational. Reviewers cannot refresh from inside Review Mode. The path to a fresh snapshot is `Request changes` → project returns to Draft → author opens Send for review again and the dialog triggers a new validation run.

---

## History scope

Validation history is **scoped to the project**, not to individual users. Every user with access to the project — author, reviewers, watchers — sees the same Validation history ordered by recency.

Each entry in the list shows trigger context:

```
Run #6 · 12:40 · Auto on Send for review
Run #5 · 12:34 · P. Petrygin (manual)
Run #4 · 11:20 · P. Petrygin (manual)
```

The auto-run committed at Send for review is the snapshot. Older entries are the author's pre-send manual runs and provide context on how validation evolved during the project's preparation.

---

## Access in Review Mode

The project model is locked in Review Mode; the Validation panel follows the same constraint — no new runs can be triggered.

The **`[Verify]` button inside the panel is hidden** in Review Mode. The **Validation entry point in the project header** (icon + severity badge) remains visible and clickable for every user — it opens the panel in read-only so the history is accessible.

Per role:

- **Author** can run `[Verify]` in Draft. Once the project is sent and switches to Review Mode, the author sees the panel in read-only — same view as the reviewer.
- **Reviewer** has read-only access: the Validation history is fully inspectable, but the `[Verify]` button is hidden.
- **Watcher** has read-only access: same as reviewer.

### Cross-highlight

Cross-highlight (clicking a check row → component highlights on Map and SLD + opens Info Panel in read-only) works for the most recent run in history — identical to existing panel behavior. Older runs are inspectable but not re-highlightable.

---

## Approve relationship

Validation results inform the Approve action; they do not gate it. The Approve button in the status popover remains enabled regardless of validation severity counts.

### Why no gate

The reviewer is the responsible party at the approval moment. A hard gate would remove the option to approve with known caveats — e.g., load-flow study attached separately, downstream remediation tracked in another project, or junior-engineer work that the senior is choosing to merge with follow-ups. When the reviewer wants the author to fix issues before merge, the appropriate action is `Request changes` with comment (Task 04), not a blocked button.

### No validation summary in the Approve popover

The validation summary already surfaces in two places ahead of the Approve action:

1. **Reviews list** (Task 03) — the `Validation` column gives the reviewer the project-level signal before they ever open the project
2. **Validation panel** — full detail once inside Review Mode

Adding the same summary into the Approve popover would duplicate signal already visible upstream.

---

## Out of scope

- Changes to the Validation panel itself (locked design — `UI-VAL-ValidationResultsPanel.md`)
- Reviewer-initiated validation runs in Review Mode — deferred for MVP per PM decision; revisited if reviewers find the read-only constraint blocking in practice
- Refresh of stale results from inside Review Mode — the path is `Request changes` → Draft → re-open Send for review
- Cross-project conflict detection — Task 06; the `Sync` column on the Reviews list (`Current`, `2 conflicts`) is the upstream signal
- Per-issue reviewer comments — general review comments are referenced via `Add comment` in Task 04
- Validation-driven automation (auto-assignment of fixers, suggested fixes, etc.)
- External notifications about validation results (email, webhook)
- Validation diff between two runs

---

## Forward references

- **Task 06**: `Sync` column and `N conflicts` indicator on the Reviews list reflect project-vs-master sync state, the entry into conflict resolution
- **Comments thread**: Channel for reviewer-to-author signal about specific validation findings — currently surfaced as `Request changes` comment (Task 04), with finer-grained per-component comments planned separately

---

## Open questions

None for Task 05 — all decisions resolved in discussion.
