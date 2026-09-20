# ICS fallback — V0.1

Generate a UTF-8 `.ics` file from validated active schedules when direct calendar access is unavailable. Derive occurrences and stable identities exactly as described in [SKILL.md](../SKILL.md) and the [adapter projection](apple-calendar.md#inputs-and-projection). Export one routine occurrence per active plan or the outstanding finite course occurrences. Exporting does not record care completion or prove that the file was imported.

## Serialization contract

Follow [RFC 5545](https://www.rfc-editor.org/rfc/rfc5545). Use a `VCALENDAR` with `VERSION:2.0` and `PRODID`. Each `VEVENT` has a stable `UID`, UTC `DTSTAMP`, `SEQUENCE`, `DTSTART`, and escaped `SUMMARY`. No `RRULE`, attendees, or invitations. Omit `METHOD` for this import snapshot.

All-day care uses `DTSTART;VALUE=DATE:YYYYMMDD` and an exclusive next-day `DTEND;VALUE=DATE`. Timed course reminders are converted from their IANA timezone to UTC `DTSTART:...Z`; omit `DTEND` for point reminders. Never convert using today's offset for future dates. Resolve DST ambiguity before export. Optional reminders use `VALARM`, `ACTION:DISPLAY`, escaped `DESCRIPTION`, and `TRIGGER:-PT<n>M` (zero is `PT0M`). All-day relative alarms start at midnight in the importing calendar's timezone; explain that timezone changes may affect their notification time.

Escape backslash, comma, semicolon, and newline in text properties. Normalize text newlines to literal `\n`; never insert raw input as a new ICS property. Serialize CRLF line endings, fold long content lines at no more than 75 UTF-8 octets using CRLF followed by one space, without splitting a UTF-8 character, and include a final CRLF.

## Example

Illustrative all-day bath export; the code block displays readable lines. A generated file must use CRLF. The timestamp below is a fixture, not the actual export time. No alarm is included because none was requested.

```ics
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//Pet Care Calendar//V0.1//EN
BEGIN:VEVENT
UID:550e8400-e29b-41d4-a716-446655440000.mochi-bath:next@pet-care-calendar.local
DTSTAMP:20260920T040000Z
SEQUENCE:0
DTSTART;VALUE=DATE:20261011
DTEND;VALUE=DATE:20261012
SUMMARY:Mochi · Bath
END:VEVENT
END:VCALENDAR
```

The long UID line also needs folding during serialization. Validate the serialized output by parsing it back: compare UIDs, event count, dates, timezone conversions, alarms, and text. Check for duplicate UIDs and absent recurrence rules. A course with two times per day over seven inclusive days produces 14 events before completions.

## Export versus synchronization

Retain the UID across date changes; increment `sequence` when an event's intended content changes. Store bindings with `provider: ics`, `sync_status: exported`, UID, occurrence ID, schedule ID, sequence, and actual export timestamp. Do not invent a provider event ID or mark `synced` without a verified import through an integration. Re-exporting identical content retains sequence.

Manual import behavior varies: the same UID does not guarantee replacement, and a new snapshot cannot reliably remove a previously imported event. Explain which old events need removal or replacement, including course cancellation and shifted routine dates. Do not claim that a cancellation file or absent event deletes anything automatically. Prefer direct synchronization when reliable update/removal is required.

Tell the user: “File generated; import it into your chosen calendar. Existing imported reminders may need manual replacement.” If file creation is unavailable, return structured state and describe that limitation instead of providing a nonexistent download link.
