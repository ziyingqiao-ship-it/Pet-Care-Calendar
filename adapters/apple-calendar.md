# Apple Calendar adapter contract — V0.1

This is a provider contract, not executable code. It assumes no AppleScript, EventKit, MCP server, OS, or installed connector. A host with authorized Apple Calendar tools may implement it; otherwise use [ICS](ics.md) or structured output.

## Inputs and projection

Accept a validated [state](../schemas/pet-care-state.schema.json), target calendar chosen by the user, and the authorized create/update/remove scope. Enforce the [skill invariants](../SKILL.md#state-invariants-host-validation-required) first. Discover actual tool capabilities: calendar listing, lookup, create, update, remove, stable IDs, and alarms. Report missing capabilities rather than claim successful synchronization.

For each active non-medication schedule, derive one all-day event on `next_due`, occurrence ID `<schedule_id>:next`. For a course, expand inclusive dates × sorted local `times`, excluding completed occurrence IDs. Include outstanding overdue doses; never move them to today. Resolve the schedule timezone and DST before any timed write. Medication events are point reminders: omit end when supported, or use an explicitly disclosed host-required display duration that does not imply dose duration.

Use the UID defined in [SKILL.md](../SKILL.md). Title: `<Pet name> · <Care label>`. Description may contain the supplied product/dosage note and an ownership marker with state ID, schedule ID, occurrence ID, and UID. Use the minimum necessary care information. No attendees or invitation emails. An optional `reminder_minutes_before` becomes an alarm relative to event start; for all-day events this is local midnight. If the host cannot honor an alarm, report that limitation.

## Reconciliation and results

1. Load state at its current revision. Identify existing owned bindings by `(provider, calendar_id, occurrence_id)` and verify that mapped event IDs still belong to this state/plan. Never identify events by title alone.
2. Build the desired event set and compare with existing owned events. Routine completion changes the date of the same upcoming event; course changes add/update/remove only affected occurrences. Plans needing information, paused plans, and completed plans have no desired events. Remove their pending owned reminders only within authorized scope; retain completion history in state.
3. Persist pending binding changes before external writes, keeping stable UID and occurrence ID. New bindings may omit `event_id` until creation succeeds. Use the selected `calendar_id`, `provider: apple_calendar`, and UTC `updated_at`.
4. Upsert by mapped ID or exact UID/ownership lookup. Re-read and verify event dates, timezone, title, and supported alarm after writing. Only then store the returned ID and `sync_status: synced`. Increment `sequence` when the intended event payload changes; keep it unchanged on identical retries.
5. Remove obsolete owned events by verified ID and mark their bindings `removed`. Do not delete unowned, historical, or manually created events. Never bulk-delete a calendar. If a user edit conflicts with state, explain the discrepancy and clarify intent rather than overwrite it silently.
6. On failure, retain confirmed care and mark affected bindings `failed`. Return per-occurrence outcomes: created, updated, unchanged, removed, failed, or ambiguous. Persist results with revision checking. Report partial success.

After a timeout or a write followed by state-save failure, look up the same UID before retrying. If the provider cannot find an event by a reliable ID/ownership marker, stop that retry as ambiguous and ask the user to reconcile; do not create a possible duplicate. For a missing mapped event, search by UID before recreating. If deletion is already verified absent, it is an idempotent success. Do not retry indefinitely or switch calendars silently.

## Acceptance cases

- Replaying an unchanged state produces no duplicate event.
- Bath completed October 15 with 21-day rule updates October 11's reminder to November 5 using the same routine occurrence identity.
- Updating one pet leaves every other pet's events unchanged.
- Two medication products for one pet retain separate schedule and occurrence IDs.
- A failed write preserves completion history and exposes the failed item for safe retry.
- A reported medication dose removes only that dose's reminder and never shifts remaining doses.
- Pausing a plan removes its remaining owned reminders without marking care completed.

No direct calendar integration is tested or shipped by this specification.
