# Readme

This folder contains several scripts used in the GitHub actions:

- `check_james_planes`: A script that can be used to check if the planes mentioned in https://github.com/sdr-enthusiasts/plane-alert-db/issues/24 are already in the main databases.
- `check_main_databases`: A script that is used to check whether the main databases are correctly formatted.
- `create_db_derivatives`: A script that can be used to create the derivative databases based on the `plane-alert-db.csv` and `plane_images.csv` files.
- `get_unique_bangers_best_items`: A script that can be used to check if the `bangers-best.csv` database contains items not found in the main databases.
- `update_readme`: A script that can be used to automate the README.md updates using the mustache template language and the chevron parser.

## Aviation-taxonomy normalizer (`normalize_aircraft_v5.py`)

This script migrates the database from a personality/group-category system to a structured aviation-taxonomy scheme. It processes one or more CSV files and produces:

- `*_normalized.csv` — rows whose `Category` was successfully resolved to a valid taxonomy value.
- `*_review.csv` — rows that still need a `Category` assigned manually.

### Support files

| File | Purpose |
|------|---------|
| `aircraft_type_lookup.csv` | Maps ICAO type designators (and common free-text type names) to normalized taxonomy values. Required columns: `match_key`, `normalized_type`, `category`, `tag1`, `tag2`, `tag3`. |
| `aircraft_type_aliases.csv` | Maps alternate / free-text spellings to a canonical `match_key` so the lookup can still find a match. Required columns: `raw_value`, `match_key`. |

### Allowed values

**Category** (24 values):
`AEW&C`, `Attack / Strike`, `Business Jet`, `Cargo Freighter`, `Electronic Warfare`,
`Fighter / Interceptor`, `Helicopter - Attack`, `Helicopter - Maritime`,
`Helicopter - Transport`, `Helicopter - Utility`, `ISR / Surveillance`,
`Maritime Patrol`, `Passenger - Narrowbody`, `Passenger - Widebody`,
`Regional Passenger`, `Special Mission`, `Strategic Airlift`, `Tactical Airlift`,
`Tanker`, `Trainer`, `UAV - Combat`, `UAV - Recon`, `UAV - Utility`, `Utility`

**Tag 1** (primary mission):
`Tactical Transport`, `Strategic Transport`, `Maritime Patrol`, `ISR`, `Early Warning`,
`Air Superiority`, `Strike`, `Close Air Support`, `Refueling`, `Training`,
`Utility`, `Electronic Warfare`

**Tag 2** (capability / configuration):
`STOL`, `Long Range`, `Short Runway`, `Heavy Lift`, `Medium Lift`, `Multi-Role`,
`All-Weather`, `High Endurance`, `Aerial Refueling`, `Carrier Capable`,
`Amphibious`, `Basic Trainer`, `Light Lift`, `Low Altitude`

**Tag 3** (propulsion / airframe):
`Twin Turboprop`, `Turboprop`, `Twin Engine`, `Quad Engine`, `Jet`, `High Wing`,
`Low Wing`, `Rear Ramp`, `Side Door`, `Pressurized`, `Sensor Suite`,
`Modular Cabin`, `Single Engine`, `Rotorcraft`

### Usage

```bash
# Run normalizer (produces plane-alert-db_normalized.csv + plane-alert-db_review.csv)
python scripts/normalize_aircraft_v5.py plane-alert-db.csv \
    --lookup scripts/aircraft_type_lookup.csv \
    --aliases scripts/aircraft_type_aliases.csv \
    --keep-link

# For a production-ready output without diagnostic columns:
python scripts/normalize_aircraft_v5.py plane-alert-db.csv \
    --lookup scripts/aircraft_type_lookup.csv \
    --aliases scripts/aircraft_type_aliases.csv \
    --keep-link \
    --no-audit-cols
```

After running, inspect `plane-alert-db_review.csv`. For each unresolved row, either:
1. Add its ICAO type to `aircraft_type_lookup.csv`, or
2. Add an alias for the free-text `$Type` value to `aircraft_type_aliases.csv`.

Then re-run until the review file is empty (or acceptably small).

### Important: `#CMPG` passthrough

The `#CMPG` column (`Mil` / `Civ` / `Gov` / `Pol`) is **intentionally not modified** by the normalizer. It reflects the operator type and is used by `create_db_derivatives.py` to split the database into `plane-alert-mil.csv`, `plane-alert-civ.csv`, etc.

