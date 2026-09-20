# 🐾 Pet Care Calendar

Turn everyday pet-care updates into calendar schedules for multiple pets.

> “Mochi had a bath today. Every three weeks.”

With today set to September 20, 2026, the next bath is October 11. If the next bath actually happens October 15, the following one moves to November 5.

**V0.1 is a portable AI skill, JSON data contracts, and calendar adapter specifications.** It does not include an executable scheduling engine, an Apple Calendar connector, or a background reminder service. The host AI reads the skill, maintains state when storage is available, and uses its own tools to produce files or operate calendars.

## Supported care

| Care | JSON value | Scheduling |
| --- | --- | --- |
| Bath | `bath` | Actual completion + supplied interval, or supplied date |
| Medication | `medication` | Finite daily course with explicit dates and times |
| Internal Deworming | `internal_deworming` | Actual completion + supplied interval, or supplied date |
| External Parasite Prevention | `external_parasite_prevention` | Actual completion + supplied interval, or supplied date |
| Vaccination | `vaccination` | Supplied next date, or explicitly supplied interval |

Each pet has independent plans, including multiple medications, parasite products, or vaccines. No default medical cycles, dosages, or treatment durations are built in. “Vaccination” never implies annual recurrence.

## Use it

1. Download or clone this repository. For hosts that load skill folders, use a folder named `pet-care-calendar` and follow that host's skill-loading mechanism. Other AI hosts can read `SKILL.md` with its linked files as instructions; compatibility and tool access depend on the host.
2. Supply the pet's name, species, timezone, and existing care instructions. For example: “Mochi is a dog. Use Asia/Shanghai. He had a bath on 2026-09-20; remind me every 21 days.”
3. Keep the returned `pet-care-state.json` in private storage and supply it in later sessions when the host cannot persist it. Do not store real care records in this public repository.
4. Choose structured output, an importable `.ics`, or an available calendar integration. Calendar writes follow your authorization and the host's permissions.

No particular AI account or calendar platform is required by the data format. Installing these files alone does not grant tools, storage, or calendar access.

## How it works

```text
Natural language → Skill → Pet care state → Upcoming occurrences → Calendar adapter
```

State is the source of truth; the calendar is a reminder layer. Routine plans create one upcoming event and reschedule from actual completion. Fixed medication courses create their finite occurrences without shifting the prescription after a late dose. Nothing creates an infinite recurring series.

Three capability levels:

- **AI only:** return structured state and upcoming care.
- **AI + files:** export `.ics` for manual import; file export is not calendar synchronization.
- **AI + calendar tools:** an adapter can create, update, and remove owned events with verification. The Apple Calendar document specifies this contract, not an implementation.

## Repository

```text
Pet-Care-Calendar/
├── README.md
├── SKILL.md
├── LICENSE
├── schemas/
│   ├── pet.schema.json
│   ├── pet-care-state.schema.json
│   └── schedule.schema.json
├── adapters/
│   ├── apple-calendar.md
│   └── ics.md
└── examples/
    ├── multi-pet.md
    ├── medication.md
    └── parasite-prevention.md
```

Start with [SKILL.md](SKILL.md). The [multi-pet example](examples/multi-pet.md) contains a complete initial state. [Medication](examples/medication.md) and [parasite prevention](examples/parasite-prevention.md) demonstrate individual schedule objects, missing information, and vaccination.

## Data contract and validation

Schemas use JSON Schema Draft 2020-12. `pet.schema.json` describes one pet; `schedule.schema.json` describes one independent care plan; `pet-care-state.schema.json` contains pets, plans, and delivery bindings. Rule types are `interval`, `fixed_date`, `course`, or null when unknown. Optional reminder offsets are only populated from the user's preference.

Load all three schemas into a local validator registry by their `$id`, enable date/date-time format checks, and validate state against `pet-care-state.schema.json`. The `$id` URLs are identifiers, not a promise of a hosted schema service. Relative references resolve against those identifiers using the local registry. Example JSON blocks are labeled by their preceding schema comment.

Schema validation is necessary but not sufficient. The [skill's state invariants](SKILL.md#state-invariants-host-validation-required) require uniqueness, references, timezone validity, date arithmetic, course expansion, and completion consistency. Adapters consume validated state and derive occurrences; they do not make clinical decisions.

## Boundaries

The skill executes care instructions supplied by the user, veterinarian, prescription, vaccination record, or supplied product instructions. It does not diagnose, prescribe, or infer cycles from product names. If information is missing, record known care and ask for the missing schedule details.

V0.1 does not implement ongoing/unbounded medication courses, non-daily medication patterns, background execution, or automatic import reconciliation. An explicit next vaccination date is consumed once. Calendar failures leave state intact and must be reported. See the [Apple Calendar contract](adapters/apple-calendar.md) and [.ics fallback](adapters/ics.md) for delivery limitations.

## License

[MIT](LICENSE).
