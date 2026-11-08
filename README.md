# orbital-works-data

Automated data mirror for [Orbital Works](https://github.com/maxmoneycash/orbital-works).

GitHub Actions fetch from CelesTrak and SatNOGS on tiered cron schedules and
commit the results as static JSON. The app reads those files straight from
`raw.githubusercontent.com`, so it gets fast, CORS-friendly, rate-limit-free
data and stays up even when the upstream APIs are throttling.

## Layout

```
celestrak/json/{group}.json          orbital elements, JSON
celestrak/tle/{group}.tle            orbital elements, three-line format
celestrak/special/json/{group}.json  GEO protected-zone datasets
catalog/satnogs.json                 SatNOGS transmitter database snapshot
catalog/stdmag.json                  standard magnitudes (brightness)
manifest.json                        per-group counts and update timestamps
```

Base URL:

```
https://raw.githubusercontent.com/maxmoneycash/orbital-works-data/main
```

## Update tiers

Groups are split by how fast their elements go stale, so hot objects stay
fresh without rewriting 20 MB every two hours.

| Workflow | Schedule | Contents |
|---|---|---|
| `update-hot` | every 2 h | stations, visual, active LEO constellations |
| `update-warm` | every 6 h | most CelesTrak groups |
| `update-cold` | every 12 h | debris and slow-moving catalogs |
| `update-catalog` | weekly | SatNOGS transmitters, standard magnitudes |
| `cleanup` | weekly | prunes files for retired groups |

Each workflow commits only when the fetched data actually differs, so the
history stays meaningful rather than one commit per cron tick.

## Running locally

No dependencies — stock Node, no `npm install`.

```bash
node scripts/fetch-celestrak.mjs --tier hot
node scripts/generate-satnogs.mjs
node scripts/generate-stdmag.mjs
```

## Data sources

- Orbital elements — [CelesTrak](https://celestrak.org/) (Dr. T.S. Kelso)
- Transmitters — [SatNOGS DB](https://db.satnogs.org/)
- Standard magnitudes — Mike McCants' visual magnitude tables

This repository redistributes public data from those sources; it does not
claim ownership of it.
