# FRED US Macro Core Data

Core U.S. macro history and replay-ready market event data sourced from FRED.

This repository now publishes the high-impact P1 macro dataset used for market replay. The older broad dataset whose event coverage effectively started in 2019 has been removed and replaced with full available history for the selected core releases.

This refresh also adds consensus actual / forecast / previous values matched from Investing.com Economic Calendar history where coverage is available.

## What's inside

- `data/fred-us-macro-history.json`
  - raw historical FRED observations for the included series
- `data/fred-us-macro-events.json`
  - replay-ready macro event records derived from the same series, now enriched with `metadata.consensus` when matched consensus history exists
- `data/investing-us-macro-consensus.json`
  - raw Investing.com Economic Calendar occurrence snapshot used for the consensus enrichment
- `metadata/series-catalog.json`
  - series metadata, transformed event value type, event counts, coverage, and source URLs
- `metadata/dataset-metadata.json`
  - generation metadata, dataset stats, and scope notes
- `metadata/excluded-series.json`
  - known third-party FRED series that are not shipped because their notes include copyright or redistribution restrictions

## Current coverage

Latest refresh in this repository:

- 15 core high-impact series
- 18,359 historical observations
- 11,986 replay-ready events
- oldest observation: 1913-01-01
- oldest event: 1914-01-01T13:30:00.000Z
- newest event: 2026-04-23T12:30:00.000Z
- 5,752 events with FRED vintage release dates
- 6,234 older events with approximate release dates
- 4,682 events enriched with Investing.com consensus actual / previous values
- 2,728 events with explicit consensus forecast values
- 14 of 15 shipped series have some forecast coverage; `GDP` currently has no matched forecast series

## Included core series

| FRED ID | Event value | Market use |
| --- | --- | --- |
| `DFEDTARU` | Federal funds target upper limit | Fed policy rate changes |
| `CPIAUCNS` | CPI year-over-year percent | Headline inflation |
| `CPILFENS` | Core CPI year-over-year percent | Core inflation |
| `PPIACO` | PPI year-over-year percent | Producer inflation |
| `PCEPI` | PCE price index year-over-year percent | Fed-preferred inflation trend |
| `PCEPILFE` | Core PCE year-over-year percent | Fed-preferred core inflation |
| `UNRATE` | Unemployment rate level | Labor market slack |
| `PAYEMS` | Nonfarm payroll monthly change | Employment growth |
| `ADPMNUSNERSA` | ADP employment monthly change | Private payroll preview |
| `CES0500000003` | Average hourly earnings month-over-month percent | Wage inflation |
| `ICSA` | Initial jobless claims level | High-frequency labor stress |
| `JTSJOL` | JOLTS job openings level | Labor demand |
| `GDP` | Nominal GDP quarter-over-quarter annualized percent | Growth |
| `GDPC1` | Real GDP quarter-over-quarter annualized percent | Real growth |
| `RSAFS` | Retail sales month-over-month percent | Consumption demand |

`DFEDTARU` is a daily source series, but this dataset only emits replay events when the policy rate changes. The noisy one-observation-per-day event stream is intentionally not published.

## Actual vs expected values

FRED provides observations and vintage/release metadata for these series, but it does not provide consensus forecast or expected-value fields. This dataset keeps FRED as the source of record for observations and release timing, then enriches matching event records with consensus values from Investing.com Economic Calendar history.

When available, event records include:

- `metadata.consensus.actual`
- `metadata.consensus.forecast`
- `metadata.consensus.previous`
- `metadata.consensus.previousRevisedFrom`
- `metadata.consensus.surprise`
- `metadata.consensus.sourceId = investing-economic-calendar`
- `metadata.consensus.sourceUrl`

Coverage is partial and varies by series and year. Missing forecasts are stored as `null`, not `0`. Older rows may have actual / previous values but no forecast.

### Consensus forecast coverage

| FRED ID | Release | Consensus-matched events | Events with forecast | First forecast observation | Latest forecast observation |
| --- | --- | ---: | ---: | --- | --- |
| `DFEDTARU` | Fed funds target upper limit | 32 | 30 | 2008-12-16 | 2025-12-11 |
| `CPIAUCNS` | Headline CPI YoY | 258 | 167 | 2012-04-01 | 2026-03-01 |
| `CPILFENS` | Core CPI YoY | 578 | 167 | 2012-04-01 | 2026-03-01 |
| `PPIACO` | PPI YoY | 168 | 168 | 2012-04-01 | 2026-03-01 |
| `PCEPI` | PCE Price Index YoY | 80 | 39 | 2019-10-01 | 2026-02-01 |
| `PCEPILFE` | Core PCE Price Index YoY | 580 | 129 | 2014-10-01 | 2026-02-01 |
| `UNRATE` | Unemployment Rate | 307 | 212 | 2008-07-01 | 2026-03-01 |
| `PAYEMS` | Nonfarm Payrolls | 627 | 218 | 2008-02-01 | 2026-03-01 |
| `ADPMNUSNERSA` | ADP Employment Change | 98 | 98 | 2010-04-01 | 2026-03-01 |
| `CES0500000003` | Avg Hourly Earnings MoM | 209 | 186 | 2008-05-01 | 2026-03-01 |
| `ICSA` | Initial Jobless Claims | 1,077 | 874 | 2009-05-30 | 2026-04-18 |
| `JTSJOL` | JOLTS Job Openings | 267 | 150 | 2013-06-01 | 2026-02-01 |
| `GDP` | Nominal GDP QoQ annualized | 0 | 0 | - | - |
| `GDPC1` | Real GDP QoQ annualized | 72 | 72 | 2008-01-01 | 2025-10-01 |
| `RSAFS` | Retail Sales MoM | 329 | 218 | 2008-01-01 | 2026-03-01 |

## Data format

### History

`data/fred-us-macro-history.json` contains raw FRED observation history:

- `generatedAt`
- `historyStart`
- `sourceId`
- `sourceLabel`
- `stats`
- `series[]`
  - `id`
  - `label`
  - `labelZh`
  - `title`
  - `category`
  - `frequency`
  - `frequencyShort`
  - `observationStart`
  - `observationEnd`
  - `units`
  - `unitsShort`
  - `sourceUrl`
  - `observations[]`
    - `date`
    - `value`

### Events

`data/fred-us-macro-events.json` contains replay-ready records sorted newest first:

- `generatedAt`
- `sourceId`
- `sourceLabel`
- `stats`
- `events[]`
  - `id`
  - `createdAt`
  - `primaryCategory`
  - `text`
  - `textEn`
  - `textZh`
  - `url`
  - `metadata`
    - `seriesId`
    - `observationDate`
    - `releaseDate`
    - `releaseDateApproximate`
    - `rawValue`
    - `value`
    - `valueKind`
    - `valueUnit`
    - `previousValue`
    - `change`
    - `pctChange`
    - `consensus`
      - `actual`
      - `forecast`
      - `previous`
      - `previousRevisedFrom`
      - `surprise`
      - `surprisePct`
      - `sourceId`
      - `sourceLabel`
      - `sourceUrl`

For transformed event series, `metadata.value` is the market-facing actual value used in replay. For example, CPI/PCE/PPI events use year-over-year percent, payroll events use period change, GDP events use annualized quarter-over-quarter percent, and `metadata.rawValue` preserves the original FRED observation.

## Release-date caveat

Where FRED vintage dates are available, event timestamps use those release dates with a market-standard release time. Older observations often do not have vintage release dates in FRED. Those records are still useful for long-run replay context, but they are marked with `metadata.releaseDateApproximate: true`.

## Attribution

This dataset is sourced from FRED and the original data owners behind each series. When displaying or redistributing derived views, please keep source attribution. A safe default is:

`Source: Original series owner via FRED, Federal Reserve Bank of St. Louis`

For series-specific source URLs, see `metadata/series-catalog.json`.

## Important note

This repository is a focused core macro dataset, not a complete mirror of FRED. You are still responsible for checking the original source terms for your own use case.

## Upstream references

- FRED API overview: <https://fred.stlouisfed.org/docs/api/fred/overview.html>
- FRED API terms of use: <https://fred.stlouisfed.org/docs/api/terms_of_use.html>
- FRED legal terms: <https://fred.stlouisfed.org/legal/terms/>
