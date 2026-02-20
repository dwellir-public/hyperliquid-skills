# L1 gRPC Gateway (Premium — $299/mo)

Low-latency gRPC streaming from the Hyperliquid L1. This is a [Dwellir-built gateway](https://github.com/dwellir-public/hyperliquid-l1-gateway) that reads Hypercore data directly from disk and serves it via gRPC. Data not available through the native Info API (like raw block data and fill streams) is accessible here.

3-day free trial available.

## Endpoint

```
Host: api-hyperliquid-mainnet-grpc.n.dwellir.com:443
Service: hyperliquid_l1_gateway.v1.HyperLiquidL1Gateway
```

## Available Methods

| Method | Type | Description |
|--------|------|-------------|
| `GetOrderBookSnapshot` | Unary | Order book snapshot at a given timestamp |
| `StreamBlocks` | Server streaming | Real-time block data from a timestamp |
| `StreamBlockFills` | Server streaming | Order fill executions in real-time |

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
