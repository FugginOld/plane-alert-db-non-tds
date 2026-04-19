# Alias expansion, compatible with the latest normalizer

This version is aligned to the current normalizer, which loads aliases using only:

- `raw_value`
- `match_key`

## Main output to use with the normalizer

Use this file with your normalizer's `--aliases` argument:

- `*_verified_expanded_for_normalizer.csv`

That file contains exactly:

```csv
raw_value,match_key
```

## Other outputs

- `*_verified_expanded.csv`  
  Same verified aliases, with a `source` column for audit.
- `*_ambiguous_review.csv`  
  Terms that should not be auto-mapped without more context.
- `*_rejected.csv`  
  Bad or unusable rows.
- `*_expansion_report.json`  
  Summary and file paths.

## Example

```bash
python3 expand_aircraft_aliases_v2.py aliases.csv
```

With local public metadata exports:

```bash
python3 expand_aircraft_aliases_v2.py aliases.csv \
  --public-metadata opensky.csv faa_export.csv
```

Then run the normalizer with:

```bash
python3 normalize_aircraft_v5.py input.csv \
  --lookup verify_lookup_template_v2.csv \
  --aliases aliases_verified_expanded_for_normalizer.csv
```

Replace `aliases_verified_expanded_for_normalizer.csv` with the actual generated filename.
