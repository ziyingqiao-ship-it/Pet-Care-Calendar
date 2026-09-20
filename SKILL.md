---
name: pet-care-calendar
description: Turn natural-language pet-care updates into independent multi-pet schedules and calendar reminders for bathing, medication, internal deworming, external parasite prevention, and vaccination. Use for recording completed care, maintaining supplied schedules, or querying upcoming care, not for diagnosis or treatment planning.
---

# Pet Care Calendar — V0.1

Record what happened, calculate what comes next, and deliver reminders using the host's available capabilities. No calendar provider or persistent memory is assumed.

## Load and resolve

Read [state schema](schemas/pet-care-state.schema.json), [pet schema](schemas/pet.schema.json), and [schedule schema](schemas/schedule.schema.json) before producing structured data. The state is the source of truth; calendars are an output layer. Load an existing user-authorized state file, or return a complete updated state for the user to save when storage is unavailable. Never claim persistence without a successful save. Do not commit real pet records or calendar identifiers to this public repository.

Resolve the pet by stable `pet_id`, not name alone. Ask when names, “both dogs,” products, or care types are ambiguous. Give each independent care plan a stable `schedule.id`; two medications or vaccines for the same pet must not overwrite each other. Retain IDs across renames. Resolve “today” in the schedule's IANA timezone using the host's actual current date; ask for a timezone if unknown. Do not demand breed, weight, or birthday to schedule reminders.

## Use supplied care plans

Support exactly `bath`, `medication`, `internal_deworming`, `external_parasite_prevention`, and `vaccination`. Preserve medication/product/vaccine wording and supplied dosage notes. Never infer dosage, duration, medical intervals, species suitability, or vaccine frequency from a name, general knowledge, or an example. The rule's `source` records the supplied instruction and its origin; a user relaying a veterinarian's instruction may be recorded as such without claiming independent verification.

Missing or conflicting instructions: record unambiguous completed care, set `status: needs_information`, `schedule_rule: null` and `next_due: null` if no usable rule exists, and ask only for the missing information. A partially described medication course belongs in `notes` until complete. Do not fabricate a rule to satisfy the schema. When conflicting instructions invalidate an existing rule, retain its wording in notes for clarification and suspend its reminders. Do not substitute a new plan or treat a scheduling calculation as clinical advice.

## Completion-based scheduling

For an interval plan, `next_due = last_completed + interval`, using the latest confirmed actual completion date. Keep only one upcoming occurrence per plan. An overdue reminder is still outstanding; the passage of time, a calendar edit, or a notification dismissal never proves completion.

- Days and weeks mean local calendar dates (a week is seven days).
- Months and years mean calendar arithmetic, clamped to the last valid day of the target month. January 31 + one month → February 28 in 2027; February 29, 2028 + one year → February 28, 2029. Always calculate from actual completion, not from a previous clamped due date.
- Reuse an existing explicit interval for the same pet and plan. Never transfer another pet's interval.
- A supplied next date uses `fixed_date` and takes precedence for that occurrence. After confirmed completion it is consumed: set `status: completed`, `next_due: null`; ask for a new date/rule if continued scheduling is wanted. Never silently repeat it annually.
- An interval without a confirmed completion anchor cannot produce a due date. Keep `needs_information` and ask for the anchor or an explicit first due date.
- Corrections replace the identified erroneous completion; duplicates must not add another record. Use stable completion IDs and clarify whether an apparent duplicate is a correction or a distinct event. An older historical completion must not move the latest date backwards. Reject future “completion” dates until clarified.

## Medication exception: finite courses

Use `course` only for an explicitly supplied daily timetable with start date, inclusive end date, and exact local times. Seven days starting September 21 end September 27; 08:00 and 20:00 give 14 occurrences. “Twice daily” or “morning/evening” alone does not specify clock times: ask for them. A duration of N days means end = start + N − 1. Do not invent dose details; include them only if supplied.

Generate the finite course in full. A dose completion references its `occurrence_id` and does not shift later doses. Missed doses remain outstanding unless the user resolves them; do not advise catch-up doses, double doses, or extend a course. `next_due` is the date of the earliest outstanding dose, including overdue ones; it is not “the next future date.” After all doses are resolved as completed, set `completed` and clear `next_due`. Early discontinuation sets `paused`, preserves history, and removes remaining reminders within authorized scope. Ongoing or non-daily prescriptions need an explicit bounded timetable before V0.1 can represent them; explain this limitation without inventing an end date. Never reinterpret “every 12 hours” as arbitrary twice-daily clock times.

## State invariants (host validation required)

JSON Schema validates shape and formats when format checking is enabled. Also enforce these semantic checks before saving or exporting:

1. Pet IDs and schedule IDs are unique; every schedule references an existing pet. Every binding references an existing schedule and an occurrence belonging to it. Completion IDs are unique within their schedule.
2. IANA timezones must exist. Course end ≥ start, and each completion's date is the actual date, not a planned date. `last_completed` equals the maximum completion date or null for empty history. Course completions reference valid, distinct occurrence IDs; routine completions omit that field.
3. Active interval due dates follow the arithmetic above. Active fixed-date due dates equal their rule date. Active courses have at least one outstanding occurrence and correct `next_due`. Non-active plans have no upcoming output. Paused/completed plans may retain rules and history but have `next_due: null`.
4. Before resolving timed occurrences, handle nonexistent or ambiguous daylight-saving times explicitly with the user; never silently shift a medical reminder. Date-only routine events stay all-day, without arbitrary times.
5. Save with a revision check: on concurrent modification, reload and reconcile rather than overwrite. Increment state revision only for actual changes. Preserve unrelated pets/plans and historical completions.

## Deliver and reconcile

Without calendar tools, return validated state and a concise summary. When files can be generated, use the [.ics fallback](adapters/ics.md). For direct Apple Calendar capabilities, read the [adapter contract](adapters/apple-calendar.md). Other hosts can implement the same occurrence model; V0.1 ships no executable calendar adapter.

Occurrence identity is independent of its date for routine care: `<schedule_id>:next`. Course identity is `<schedule_id>:<YYYY-MM-DD>:<HH:MM>`. UID is `<state_id>.<occurrence_id>@pet-care-calendar.local`. `state_id` must be a freshly generated opaque globally unique value (for example a lowercase UUID), stable for this state. Event titles use `<Pet name> · <Care label>`; product details are optional descriptions. Do not put raw prompts or unrelated private information in calendar descriptions.

Calendar changes require user-authorized scope and available host permissions. Reuse existing authorization; a report of completion alone does not grant access to a new calendar. Record intended changes as pending, perform authorized operations, and mark synced only after verification. A sync failure does not erase confirmed care. Retry by stable identity, never blindly create duplicates. Disclose partial results and unresolved reminders. Canceling a reminder is not a care completion.

Respond in the user's language with pet, activity, recorded completion, next date(s), and delivery status: saved, synced, exported for import, or awaiting information. Upcoming queries are read-only, chronological, and include overdue items.

## Examples

- [Multiple pets and completion changes](examples/multi-pet.md)
- [Finite medication course](examples/medication.md)
- [Independent parasite schedules and vaccination](examples/parasite-prevention.md)
