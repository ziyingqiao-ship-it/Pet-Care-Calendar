# Parasite prevention and vaccination

Fictional products, dates, and instructions for demonstrating scheduling only. Never reuse these intervals as care recommendations. Pets `mochi` and `nori` exist in the [multi-pet state](multi-pet.md). Each JSON block below is one independent schedule that can be appended to that state.

> Mochi had internal treatment today, every three months, and external treatment today, every month. Use Asia/Shanghai; today is September 20, 2026.

<!-- schema: schedule -->
```json
{
  "id": "mochi-internal",
  "pet_id": "mochi",
  "care_type": "internal_deworming",
  "label": "Internal Deworming",
  "timezone": "Asia/Shanghai",
  "status": "active",
  "schedule_rule": {
    "type": "interval",
    "value": 3,
    "unit": "month",
    "source": {
      "kind": "user",
      "detail": "Internal product A every three months, as supplied by the user."
    }
  },
  "last_completed": "2026-09-20",
  "next_due": "2026-12-20",
  "completions": [
    {
      "id": "completion-1",
      "date": "2026-09-20"
    }
  ],
  "product_name": "Internal product A"
}
```

<!-- schema: schedule -->
```json
{
  "id": "mochi-external",
  "pet_id": "mochi",
  "care_type": "external_parasite_prevention",
  "label": "External Parasite Prevention",
  "timezone": "Asia/Shanghai",
  "status": "active",
  "schedule_rule": {
    "type": "interval",
    "value": 1,
    "unit": "month",
    "source": {
      "kind": "user",
      "detail": "External product B every month, as supplied by the user."
    }
  },
  "last_completed": "2026-09-20",
  "next_due": "2026-10-20",
  "completions": [
    {
      "id": "completion-1",
      "date": "2026-09-20"
    }
  ],
  "product_name": "External product B"
}
```

“What is Mochi's next parasite care?” returns both plans: external October 20, internal December 20. If external care actually completes October 23, its next date is November 23. The internal plan is unchanged. “Mochi was dewormed today” is ambiguous with these two plans; clarify which treatment before updating.

## Product without a cycle

> Nori received Internal product D today. I haven't provided the next interval.

<!-- schema: schedule -->
```json
{
  "id": "nori-internal",
  "pet_id": "nori",
  "care_type": "internal_deworming",
  "label": "Internal Deworming",
  "timezone": "Asia/Shanghai",
  "status": "needs_information",
  "schedule_rule": null,
  "last_completed": "2026-09-20",
  "next_due": null,
  "completions": [
    {
      "id": "completion-1",
      "date": "2026-09-20"
    }
  ],
  "product_name": "Internal product D"
}
```

Record completion and ask for the existing veterinarian/product/user-supplied schedule. Create no next reminder. Do not copy Mochi's interval. If the user describes one product covering multiple care types, clarify the intended single administration plan instead of creating duplicate administration reminders from assumed coverage.

## Vaccination date supplied explicitly

<!-- schema: schedule -->
```json
{
  "id": "nori-vaccine",
  "pet_id": "nori",
  "care_type": "vaccination",
  "label": "Vaccination · Vaccine C",
  "timezone": "Asia/Shanghai",
  "status": "active",
  "schedule_rule": {
    "type": "fixed_date",
    "date": "2027-09-20",
    "source": {
      "kind": "vaccination_record",
      "detail": "User supplied vaccination record explicitly lists next date 2027-09-20."
    }
  },
  "last_completed": "2026-09-20",
  "next_due": "2027-09-20",
  "completions": [
    {
      "id": "completion-1",
      "date": "2026-09-20"
    }
  ],
  "product_name": "Vaccine C"
}
```

This is one supplied appointment date, not an inferred annual series. After completion on September 20, 2027, append the completion, set last completed to that date, set status to `completed` and next due to null. Another date or interval must be supplied before another reminder is created. “Nori had a vaccination today” alone only records completion and requests the missing next-date instruction.
