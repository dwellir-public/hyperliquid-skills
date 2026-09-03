# L1 gRPC Gateway

Low-latency gRPC streaming from the Hyperliquid L1. This is a Dwellir-built gateway that reads Hypercore data directly from disk and serves it via gRPC. Data not available through the native Info API (like raw block data and fill streams) is accessible here.

Docs are ground truth: the gateway moves fast, so this file names the services and the connection contract, then points at the versioned reference. **For current methods, params, pricing, and full documentation, see [Dwellir gRPC API docs](https://www.dwellir.com/docs/hyperliquid/grpc).**

## Versions and services

Use V3 for new integrations. V2 keeps working for existing clients.

| Version | Service | Surface |
|---------|---------|---------|
| V3 | `hyperliquid_l1_gateway.v3.HyperliquidL1Gateway` | Feed discovery (`ListFeeds`), 7 raw streams, 7 raw unary lookups |
| V3 | `hyperliquid_l1_gateway.v3.MarketStreaming` | 14 typed live market streams (books, BBO, trades, fills, candles, mids) |
| V2 | `hyperliquid_l1_gateway.v2.HyperliquidL1Gateway` | Compatibility surface, 14 established raw methods |

Download the contracts and generate stubs from them. Do not hand-write messages.

```bash
curl -fsSLo v3.proto https://www.dwellir.com/docs/hyperliquid/grpc/v3.proto
curl -fsSLo v2.proto https://www.dwellir.com/docs/hyperliquid/grpc/v2.proto
```

Raw feeds: blocks, fills, raw book diffs, order statuses, misc events, TWAP statuses, generated order-book snapshots. Market streams: `StreamL2Book`, `StreamBbo`, `StreamL2BookDiff`, `StreamL4Book`, `StreamL4OrderUpdates`, `StreamTrades`, fills family, `StreamCandle`, `StreamAllMids`, `StreamActiveAssetCtx`. Full method list lives in the [docs](https://www.dwellir.com/docs/hyperliquid/grpc).

All raw streams take `StreamRequest`, all raw unary methods take `GetRequest` (position by timestamp or block number, plus field filters).

## Endpoint and auth

TLS plus API key in the `x-api-key` metadata header. Without a valid key the gateway answers 403 "API key does not exist" (verified live 2026-09-04).

```bash
grpcurl -proto v3.proto \
  -H "x-api-key: ${DWELLIR_API_KEY}" \
  "${HYPERLIQUID_GRPC_ENDPOINT}" \
  hyperliquid_l1_gateway.v3.HyperliquidL1Gateway/ListFeeds
```

### Python

```python
import grpc

channel = grpc.secure_channel(
    'api-hyperliquid-mainnet-grpc.n.dwellir.com:443',
    grpc.ssl_channel_credentials(),
    options=[('grpc.primary_user_agent', 'dwellir-skill')],
)
metadata = [('x-api-key', DWELLIR_API_KEY)]
# Use stubs generated from v3.proto, e.g.
# stub = HyperliquidL1GatewayStub(channel)
# for event in stub.StreamBlocks(StreamRequest(...), metadata=metadata):
#     ...
```

### Go

```go
import (
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials"
    "google.golang.org/grpc/metadata"
)

conn, err := grpc.NewClient(
    "api-hyperliquid-mainnet-grpc.n.dwellir.com:443",
    grpc.WithTransportCredentials(credentials.NewTLS(nil)),
)
// Attach per-RPC credentials: metadata.Pairs("x-api-key", apiKey)
// Use stubs generated from v3.proto
```

## Stream behavior worth knowing

- Market streams are live-only, no backfill. One RPC is one subscription; cancel to unsubscribe.
- No heartbeats; an idle subscription can be healthy.
- Incremental streams end with `ABORTED` on frame loss; reconnect and rebuild from the opening snapshot.
- Oversized snapshot responses return `RESOURCE_EXHAUSTED`.
- Prices and sizes are decimal strings; times are Unix milliseconds.

## Use Cases

- Real-time block data ingestion for analytics
- Trade/fill streaming for backtesting engines
- Order book snapshots at specific timestamps
- Building indexers and data pipelines
