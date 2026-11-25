# FieldPulse - Step 1: data model and confidence engine

Standard library only (Python 3.8+). No installs needed.

## Files
- `schema.sql` - SQLite schema (Region, Category, SKU, Contributor by Pi UID only, Observation, Verification)
- `fieldpulse_engine.py` - confidence score, market-state classification, Pi presence, opportunity category, price spread
- `seed.py` - loads YOUR real observations from CSV and prints a report
- `observations_template.csv` - header for the field-collection sheet
- `test_engine.py` - 22 unit tests

## Run
```
python3 -m unittest -v test_engine
python3 seed.py observations.csv fieldpulse.db
```

## CSV values (Arabic or English both work)
- availability: `in_stock` / متوفر, `low` / قليل, `out` / غير متوفر
- source: `photo_evidence` / صورة, `self_observed` / شفت بنفسي, `merchant_reported` / قالي التاجر
- accepts_pi: yes / نعم, no / لا
- date: `YYYY-MM-DD` or `DD/MM/YYYY`; Arabic digits are fine in prices

## How the score works (all numbers are in `Config`, change them there)
Confidence 0-100 = volume (50) + price agreement (30) + verification (20).
- Source weights: photo 1.0, self-observed 0.85, merchant-reported 0.5
- Field-verified x1.25, disputed excluded entirely
- Freshness: weight halves every 14 days, ignored after 60 days, only the last 30 days count for classification

## Classification
- `data_gap`: fewer than 3 fresh observations. Missing data is never a shortage.
- `true_supply_gap`: shortage share >= 60% (or median price >= 20% above baseline) AND confidence >= 60
- `unproven_demand`: same signal, confidence below 60
- `normal`: enough data, no signal

Opportunity categories: golden_gap, b2b_supply_match, active_price_benchmark, needs_verification.
One category was added: `no_opportunity` (stable supply, no Pi merchants).

## Honest limits
- "Pi presence" comes from the `accepts_pi` column you record by hand. It is not an integration with Map of Pi.
- The weights are a reasonable starting point, not calibrated on real data. Re-check them after your first 30-50 observations.
