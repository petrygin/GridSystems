# Task 05 — Validation in Review Mode

**Scope:** How the existing Validation Results Panel surfaces inside Review Mode. Run-trigger policy, isolated per-user histories, and the relationship between validation results and the Approve action.

**Related:** Review Mode surface and status popover → Task 04. Send-for-review entry → Task 01. Reviews list inbox → Task 03. Project-vs-master sync state → Task 06.

**Design:** [Figma — Validation in Review Mode](https://www.figma.com/design/0FW49cYCouwjKw3YzqQwcI/Grid-Model-Work-in-Progress?node-id=3283-113999)

For shared terminology, state model, and the validation policy at Send for Review, see `ReviewFlow_00_Overview.md`. For the panel design itself (three views, section grouping, severity treatment, cross-highlight), see `UI-VAL-ValidationResultsPanel.md`.

---

## Goal

Give reviewers a path to verify project health on their own terms:

- Manual control via `[Verify]` to trigger a validation run when needed
- Direct inspection of each finding (cross-highlight to Map / SLD, Info Panel in read-only)
- Clean separation between their own validation work and prior author activity

Approving with outstanding errors is allowed — the reviewer carries that decision, not the system.

---

## Architectural framing

Review Mode reuses the existing Validation Results Panel without structural changes: same right drawer, same three views (Verifications list / Results collapsed / Results expanded), same grouping by verification stage, same per-row severity, same click-to-cross-highlight.

Task 05 defines wrapper behavior around that panel inside Review Mode:

1. **Run triggers** — when validation runs happen across the lifecycle
2. **Access and history** — who can run validation and what they see
3. **Approve relationship** — how results connect (or don't connect) to the Approve action

Entry point is unchanged: the warnings/errors icon in the project header, present across Edit and Review modes.

---

## Run triggers

Validation runs are triggered **exclusively by the user** via the `[Verify]` button. No moment in the lifecycle triggers an automatic run.

| Event                              | Validation behavior                                     |
|------------------------------------|---------------------------------------------------------|
| Author works in Draft              | Manual `[Verify]` runs, recorded in author's history    |
| Author clicks Send for review      | Author's last manual run carries forward as the project's validation snapshot on the Reviews list |
| Reviewer opens project             | Reviewer sees their own (initially empty) Verifications list |
| Anyone clicks `[Verify]`           | Manual run, recorded in that user's history             |

The reviewer who opens the project for the first time sees an empty Verifications list — histories are isolated per user (see below), so prior runs by the author are not visible to them. When the reviewer wants to inspect the project's current validation state, they press `[Verify]` to trigger a run.

### Stale state

The project model is locked while in Review Mode, so validation results cannot go stale from within the project. The single trigger for staleness is **master moving** between the reviewer's last run and their current session — i.e., another project merged in the meantime.

When this happens, the Verification Results header shows a `Results may be outdated — master updated` indicator. The reviewer presses `[Verify]` when they want fresh results.

---

## Access and history

The `[Verify]` button is visible to every viewer of the project in Review Mode: author, reviewers, watchers. Anyone present can run validation at any time.

### Isolated histories per user

Each user's validation run history is scoped to their identity:

- Author sees runs they triggered (in Draft and any later runs in Review Mode)
- Reviewer sees runs they triggered (which starts empty and grows from their first manual `[Verify]`)
- Watcher sees runs they triggered

A user's history does not include runs triggered by anyone else. Authors don't see what reviewers found; reviewers don't see prior author work.

Cross-user signal travels through other channels:

- **Reviews list** (Task 03) — project-level summary in the `Validation` column (`21 errors`, `7 warnings`, or empty), populated from the author's last manual run at the moment of Send for Review
- **Review comments** — `Request changes` with explicit comment when the reviewer wants the author to address specific findings

The Verifications list (panel header → ☰) shows the current user's runs only, ordered by recency.

### Cross-highlight

Cross-highlight (clicking a check row → component highlights on Map and SLD + opens Info Panel in read-only) works for the user's latest run only — identical to existing panel behavior. Older runs in the user's own history are inspectable but not re-highlightable.

---

## Approve relationship

Validation results inform the Approve action; they do not gate it. The Approve button in the status popover remains enabled regardless of validation severity counts.

### Why no gate

The reviewer is the responsible party at the approval moment. A hard gate would remove the option to approve with known caveats — e.g., load-flow study attached separately, downstream remediation tracked in another project, or junior-engineer work that the senior is choosing to merge with follow-ups. When the reviewer wants the author to fix issues before merge, the appropriate action is `Request changes` with comment (Task 04), not a blocked button.

### No validation summary in the Approve popover

The validation summary already surfaces in two places ahead of the Approve action:

1. **Reviews list** (Task 03) — the `Validation` column gives the reviewer the project-level signal before they ever open the project
2. **Validation Results Panel** — full detail once inside Review Mode (after the reviewer presses `[Verify]`)

Adding the same summary into the Approve popover would duplicate signal already visible upstream.

---

## Out of scope

- Changes to the Validation Results Panel itself (locked design — `UI-VAL-ValidationResultsPanel.md`)
- Automatic validation runs at any point in the lifecycle — runs are user-triggered only
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
