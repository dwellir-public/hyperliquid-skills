# L1 gRPC Gateway

Low-latency gRPC streaming from the Hyperliquid L1. This is a Dwellir-built gateway that reads Hypercore data directly from disk and serves it via gRPC. Data not available through the native Info API (like raw block data and fill streams) is accessible here.

**For current pricing, available methods, and full documentation, see [Dwellir gRPC API docs](https://www.dwellir.com/docs/hyperliquid/grpc).**

## Endpoint

```
Host: api-hyperliquid-mainnet-grpc.n.dwellir.com:443
Service: hyperliquid_l1_gateway.v1.HyperLiquidL1Gateway
```

## Available Methods

The gRPC gateway exposes both streaming and unary (point-in-time) methods. Check the [full docs](https://www.dwellir.com/docs/hyperliquid/grpc) for the current method list. The service expands over time.

| Method | Type | Description |
|--------|------|-------------|
| `StreamBlocks` | Server streaming | Real-time block data from a timestamp |
| `StreamFills` | Server streaming | Order fill executions in real-time |
| `StreamOrderbookSnapshots` | Server streaming | Real-time order book snapshots |
| `GetBlock` | Unary | Block at a specific height or timestamp |
| `GetFills` | Unary | Fill data at a specific point in time |
| `GetOrderBookSnapshot` | Unary | Order book snapshot at a given timestamp |

Data retention: 24 hours of historical data.

## Connection

Proto files are available upon request from support@dwellir.com.

### Python

```python
import grpc

channel = grpc.secure_channel(
    'api-hyperliquid-mainnet-grpc.n.dwellir.com:443',
    grpc.ssl_channel_credentials()
)
# Use generated stubs from Hyperliquid L1 gateway proto files
# stub = HyperLiquidL1GatewayStub(channel)
# response = stub.GetOrderBookSnapshot(request)
```

### Go

```go
import (
    "crypto/tls"
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials"
)

creds := credentials.NewTLS(&tls.Config{})
conn, err := grpc.Dial(
    "api-hyperliquid-mainnet-grpc.n.dwellir.com:443",
    grpc.WithTransportCredentials(creds),
)
// Use generated stubs for method calls
```

## Use Cases

- Real-time block data ingestion for analytics
- Trade/fill streaming for backtesting engines
- Order book snapshots at specific timestamps
- Building indexers and data pipelines
