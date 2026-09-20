# Multiple pets and completion changes

All identities and care instructions in these examples are fictional fixtures, not recommendations. Reference date: 2026-09-20; timezone: Asia/Shanghai. The user explicitly supplied each interval.

> Mochi and Nori are dogs. Mochi had a bath today; every 21 days. Nori had a bath yesterday; every 28 days.

Complete state (no calendar writes yet):

<!-- schema: pet-care-state -->
```json
{
  "schema_version": "0.1",
  "state_id": "550e8400-e29b-41d4-a716-446655440000",
  "revision": 1,
  "updated_at": "2026-09-20T04:00:00Z",
  "pets": [
    {
      "id": "mochi",
      "name": "Mochi",
      "species": "dog"
    },
    {
      "id": "nori",
      "name": "Nori",
      "species": "dog"
    }
  ],
  "schedules": [
    {
      "id": "mochi-bath",
      "pet_id": "mochi",
      "care_type": "bath",
      "label": "Bath",
      "timezone": "Asia/Shanghai",
      "status": "active",
      "schedule_rule": {
        "type": "interval",
        "value": 21,
        "unit": "day",
        "source": {
          "kind": "user",
          "detail": "Bathe Mochi every 21 days."
        }
      },
      "last_completed": "2026-09-20",
      "next_due": "2026-10-11",
      "completions": [
        {
          "id": "completion-1",
          "date": "2026-09-20"
        }
      ]
    },
    {
      "id": "nori-bath",
      "pet_id": "nori",
      "care_type": "bath",
      "label": "Bath",
      "timezone": "Asia/Shanghai",
      "status": "active",
      "schedule_rule": {
        "type": "interval",
        "value": 28,
        "unit": "day",
        "source": {
          "kind": "user",
          "detail": "Bathe Nori every 28 days."
        }
      },
      "last_completed": "2026-09-19",
      "next_due": "2026-10-17",
      "completions": [
        {
          "id": "completion-1",
          "date": "2026-09-19"
        }
      ]
    }
  ],
  "calendar_bindings": []
}
```

## Late completion

On 2026-10-15 the user says, “Mochi had his bath today.” Append a new completion with a new ID and date 2026-10-15; set `last_completed` to 2026-10-15 and `next_due` to 2026-11-05. Increment state revision and use the actual update timestamp. Nori remains due 2026-10-17. If authorized calendar access exists, update `mochi-bath:next`; otherwise export a replacement and explain manual removal of the old reminder.

On replay of that same completion report, do not append another completion or duplicate the event. A historical correction to September's bath must not overwrite October's latest completion.

## Clarifications and edge cases

- “Both dogs had a bath” updates both known dogs independently using their own intervals. If which dogs are meant is unclear, ask.
- “Mochi had a bath” with no date uses the actual host date, not this fixture date. With no rule, save the completion as `needs_information` with null rule/due date and ask for the interval.
- A confirmed 2027-01-31 completion plus one month is due 2027-02-28, not March 3.
- Deleting a calendar event does not mark either pet's bath as completed.
