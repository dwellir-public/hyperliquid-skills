# Hyperliquid Historical Data

Dwellir hosts Hyperliquid archival data in S3.

## Endpoint

```
Endpoint: https://hyperliquid-archival-data.n.dwellir.com
Bucket: hyperliquid-historical-data
```

## Available Data

| Path | Description |
|------|-------------|
| `node_fills/hourly/` | Fill data by hour |
| `node_fills_by_block/hourly/` | Fills organized by block |
| `node_trades/hourly/` | Trade data by hour |

Files are compressed with LZ4. Bandwidth limit: 500 Mbit shared.
