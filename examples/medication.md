# Finite medication course

Fictional scheduling fixture, not a prescription. Pet `nori` exists in the [multi-pet state](multi-pet.md); timezone is Asia/Shanghai. The user supplies:

> Nori starts Medicine A on September 21, 2026, at 08:00 and 20:00 daily for seven days, according to her prescription.

Append this schedule to that state's schedules (update revision and timestamp when saving):

<!-- schema: schedule -->
```json
{
  "id": "nori-medication-a",
  "pet_id": "nori",
  "care_type": "medication",
  "label": "Medication · Medicine A",
  "timezone": "Asia/Shanghai",
  "status": "active",
  "schedule_rule": {
    "type": "course",
    "start_date": "2026-09-21",
    "end_date": "2026-09-27",
    "times": [
      "08:00",
      "20:00"
    ],
    "source": {
      "kind": "prescription",
      "detail": "User-supplied prescription: Medicine A, September 21–27, at 08:00 and 20:00 daily."
    }
  },
  "last_completed": null,
  "next_due": "2026-09-21",
  "completions": [],
  "product_name": "Medicine A"
}
```

The inclusive date range is September 21–27: 7 × 2 = 14 reminders. No dosage note or advance alarm is invented. First occurrence ID: `nori-medication-a:2026-09-21:08:00`; last: `nori-medication-a:2026-09-27:20:00`. In Asia/Shanghai these convert to 2026-09-21T00:00:00Z and 2026-09-27T12:00:00Z.

When the user confirms the first dose, append `{"id":"dose-1","date":"2026-09-21","occurrence_id":"nori-medication-a:2026-09-21:08:00"}` to completions and set `last_completed: 2026-09-21`. Thirteen outstanding reminders remain; `next_due` is still 2026-09-21 for the 20:00 dose. Do not shift later doses. On completion of all 14, status becomes `completed` and next due becomes null.

If an earlier dose is overdue, it remains outstanding; ask how the user wants the reminder handled without recommending a medical action. If the user discontinues the course, set `paused`, clear next due, and reconcile remaining reminders. Never extend the course automatically.

## Missing information

> Nori starts medication tomorrow, morning and evening for seven days.

Record the supplied details in notes, keep a null rule and next due with `needs_information`, and ask for the medication identifier and exact reminder times. If timezone is unknown, ask for that too. Do not guess 08:00/20:00 from this example. Two independent medicines need two schedule IDs.

“Continue indefinitely” and “every other day” are outside V0.1's finite daily course model; request an explicit supported bounded timetable instead of silently changing the prescription.
