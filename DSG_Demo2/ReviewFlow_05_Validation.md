# Task 05 — Validation in Review Mode

**Scope:** How the existing Validation panel surfaces inside Review Mode. Project-scoped run history, manual triggers only, equal access for all users, and the relationship between validation results and the Approve action.

**Related:** Review Mode surface and status popover → Task 04. Send-for-review entry → Task 01. Reviews list inbox → Task 03. Project-vs-master sync state → Task 06.

**Design:** [Figma — Validation in Review Mode](https://www.figma.com/design/0FW49cYCouwjKw3YzqQwcI/Grid-Model-Work-in-Progress?node-id=3283-113999)

For shared terminology, state model, and the validation policy at Send for Review, see `ReviewFlow_00_Overview.md`. For the panel design itself (three views, section grouping, severity treatment, cross-highlight), see `UI-VAL-ValidationResultsPanel.md`.

---

## Goal

Give reviewers a confident read on the technical health of the project they're reviewing:

- Project-scoped history of validation runs shared by every viewer
- Manual `[Verify]` available to everyone — author, reviewer, watcher — across modes
- Direct inspection of each finding (cross-highlight to Map / SLD, Info Panel in read-only)

Approving with outstanding errors is allowed — the reviewer carries that decision, not the system.

---

## Architectural framing

Review Mode reuses the existing Validation panel without structural changes: same right drawer, same three views (Validation history / Results collapsed / Results expanded), same grouping by validation stage, same per-row severity, same click-to-cross-highlight.

Task 05 defines wrapper behavior around that panel inside Review Mode:

1. **Run triggers** — when validation runs happen across the lifecycle
2. **History scope** — where runs are stored and who can see them
3. **Access** — what each role can do across modes
4. **Approve relationship** — how results connect (or don't connect) to the Approve action

Entry point is unchanged across modes: the Validation icon (with severity badge) in the project header opens the panel.

---

## Run triggers

Validation runs are **manual only** — there are no system-triggered runs anywhere in the lifecycle.

| Event                              | Validation behavior                                                  |
|------------------------------------|----------------------------------------------------------------------|
| Author in Draft clicks `[Verify]`  | New run added to project history                                     |
| Author opens Send for review dialog | Dialog reflects the latest run already in history (when one exists); no new run is triggered |
| Reviewer or watcher in Review Mode clicks `[Verify]` | New run added to project history                                     |

The latest run in project history at the moment of Send for review serves as the **implicit snapshot** for the review. There is no separate snapshot record — reviewers simply look at the most recent run in the shared history.

If no run exists when the author opens Send for review, the dialog's validation banner is empty and the Reviews list `Validation` column shows the project as unvalidated. The author can run `[Verify]` before sending if they want validation context to travel with the project; this is optional.

### Stale state

Results can become stale when **master moves** between a run and the current session — another project merged in the meantime.

When this happens, the Validation panel header shows `Results may be outdated — master updated`. The indicator is informational. Any user with access — author, reviewer, watcher — can press `[Verify]` to trigger a fresh run that lands in the shared project history.

---

## History scope

Validation history is **scoped to the project**, not to individual users. Every user with access to the project — author, reviewers, watchers — sees the same Validation history ordered by recency.

Each entry in the list shows trigger context:

```
Run #6 · 12:40 · A. Reviewer
Run #5 · 12:34 · P. Petrygin
Run #4 · 11:20 · P. Petrygin
```

Older entries provide context on how validation evolved during the project's lifecycle, including runs triggered by different users.

---

## Access

The `[Verify]` button is available to every viewer of the project, in any mode:

- **Author** in Draft uses `[Verify]` to validate during edits. In Review Mode the author cannot edit the project but can still trigger validation runs.
- **Reviewer** in Review Mode uses `[Verify]` to refresh validation against the current master state if they want.
- **Watcher** in Review Mode uses `[Verify]` the same way.

All triggered runs land in the shared project history.

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
- Automatic validation runs at any point in the lifecycle — runs are manual only
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
