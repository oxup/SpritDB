# SpritDB

Automated snapshot of all Austrian gas station prices, updated every **n minutes**.

---

## Served files

| URL | Content | Size |
|-----|---------|------|
| `/data/meta.json` | Freshness check (timestamp, count) | ~120 B |
| `/data/stations.bin.gz` | Full binary snapshot — all stations, current prices | ~100 KB |
| `/data/history/{id}.csv.gz` | Per-station price time-series | ~5 KB each |

---

## Binary format (SPRT v1)

### Header — 16 bytes, big-endian

| Offset | Size | Field | Value |
|--------|------|-------|-------|
| 0 | 4 | magic | `b'SPRT'` |
| 4 | 1 | version | `1` |
| 5 | 1 | reserved | `0` |
| 6 | 2 | reserved | `0` |
| 8 | 4 | timestamp | unix epoch UTC (uint32) |
| 12 | 2 | count | number of station records (uint16) |
| 14 | 2 | reserved | `0` |

### Per-station record — variable length, big-endian

| Size | Field | Notes |
|------|-------|-------|
| 4 | id | uint32 |
| 4 | lat | float32 |
| 4 | lon | float32 |
| 2 | diesel | uint16 · price×1000 (2199 = €2.199), 0 = n/a |
| 2 | super95 | uint16 |
| 2 | superplus | uint16 |
| 2 | gas | uint16 |
| 1 | flags | uint8 · bit0=open, bit1=openAllDay |
| 2 | postal | uint16 (0 = unknown) |
| 1+? | name | uint8 length + UTF-8 bytes (max 255 B) |
| 1+? | city | uint8 length + UTF-8 bytes |
| 1+? | address | uint8 length + UTF-8 bytes |

---

## Per-station history format

`data/history/{id}.csv.gz` — one file per station, gzip-compressed CSV:

```csv
ts,diesel,super95,superplus,gas
1743523200,2.159,1.819,,
1743525000,2.159,1.819,,
1743526800,2.129,1.799,,
```

- `ts` — unix timestamp UTC
- Empty string = price not available at that snapshot

---
