---
name: hyperliquid
description: >
  Hyperliquid L1 reference for Dwellir endpoints — HyperEVM JSON-RPC, Info API proxy,
  gRPC L1 streaming, order book WebSocket, dedicated nodes, and trading patterns.
  Covers HyperCore trading layer, HyperEVM smart contracts (chain ID 998),
  market data queries, perpetuals metadata, spot markets, and best practices.
  Use when working with Hyperliquid, HYPE, HyperEVM, HyperCore,
  perpetual futures, order books, funding rates, or Hyperliquid trading through Dwellir.
  Triggers on mentions of hyperliquid, HYPE, HyperEVM, HyperCore,
  order book, perpetuals, funding rate, l2Book, l4Book, gRPC streaming,
  nanoreth, or hyperliquid trading.
---

# Hyperliquid via Dwellir

Hyperliquid is a purpose-built L1 blockchain optimized for trading. Dwellir runs its own Hyperliquid nodes and offers infrastructure beyond standard RPC: a custom gRPC gateway for Hypercore data, a real-time order book server, and a filtering Info API proxy — with edge servers in Singapore and Tokyo.

## How Hyperliquid Works

Hyperliquid has two layers:

**HyperCore** — The native trading layer. Fully on-chain perpetual futures and spot order books. Every order, cancellation, trade, and liquidation settles within one block. Handles ~200,000 orders/second with sub-second finality via HyperBFT consensus.

**HyperEVM** — A general-purpose EVM smart contract layer that runs alongside HyperCore. Developers can deploy Solidity contracts that interact with HyperCore's liquidity. Chain ID: **998**. Native gas token: **HYPE**.

Key properties:
- All order books are fully on-chain — no off-chain matching
- Sub-second block times with one-block finality
- Perpetuals support up to 50x leverage
- Native spot trading with HIP-3 DEX deployment

## What Dwellir Provides

Dwellir runs full Hyperliquid infrastructure: the official HL node, plus custom software built by Dwellir and the community for serving specific data channels.

| Endpoint | What It Serves | Protocol | Pricing | Reference |
|----------|---------------|----------|---------|-----------|
| **HyperEVM JSON-RPC** | EVM state, smart contracts, blocks | HTTPS + WSS | Base plan | [hyperevm-json-rpc.md](references/hyperevm-json-rpc.md) |
| **Info API proxy** | Market data, user state, metadata | HTTPS (POST) | Base plan | [info-api.md](references/info-api.md) |
| **L1 gRPC Gateway** | Hypercore block/fill streaming | gRPC | $299/mo add-on | [grpc-gateway.md](references/grpc-gateway.md) |
| **Orderbook WebSocket** | Real-time L2/L4 order book data | WSS only | $199/mo add-on | [orderbook-websocket.md](references/orderbook-websocket.md) |
| **Dedicated Node** (Tokyo) | Full stack, uncapped throughput | All | $1,150/mo | See below |
| **Dedicated Node** (Testnet) | Full stack for testing | All | $800/mo | See below |

### What Dwellir Does NOT Proxy

**Exchange API** — Order placement, cancellation, transfers, and other write operations require EIP-712 signatures and go directly to `api.hyperliquid.xyz/exchange`. See [native-api.md](references/native-api.md).

**Native WebSocket** — Hyperliquid's subscription WebSocket (`wss://api.hyperliquid.xyz/ws`) for user events, trades, and candles is separate from Dwellir's Orderbook WebSocket. See [native-api.md](references/native-api.md).

### Read vs Write Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Your Application                                        │
├──────────────────┬──────────────────────────────────────┤
│  READ (Dwellir)  │  WRITE (Hyperliquid native)          │
│                  │                                       │
│  EVM state ──────┤  Place orders ─── api.hyperliquid.xyz │
│  Info queries ───┤  Cancel orders    /exchange           │
│  gRPC streams ───┤  Transfers        (requires sig)      │
│  Order book ─────┤  Set leverage                         │
└──────────────────┴──────────────────────────────────────┘
```

## When to Use Which Reference

| You want to... | Use this reference |
|----------------|-------------------|
| Deploy or interact with Solidity contracts on HyperEVM | [hyperevm-json-rpc.md](references/hyperevm-json-rpc.md) |
| Query EVM state (balances, logs, blocks) | [hyperevm-json-rpc.md](references/hyperevm-json-rpc.md) |
| Get market data (prices, order books, candles, funding rates) | [info-api.md](references/info-api.md) |
| Query user positions, orders, fills, or balances | [info-api.md](references/info-api.md) |
| Get perpetuals/spot metadata (universe, leverage, assets) | [info-api.md](references/info-api.md) |
| Stream real-time order book updates with deep levels | [orderbook-websocket.md](references/orderbook-websocket.md) |
| Build market-making or arbitrage systems | [orderbook-websocket.md](references/orderbook-websocket.md) |
| Stream L1 block data or fill executions | [grpc-gateway.md](references/grpc-gateway.md) |
| Build indexers or data pipelines from Hypercore | [grpc-gateway.md](references/grpc-gateway.md) |
| Place, cancel, or modify orders | [native-api.md](references/native-api.md) |
| Subscribe to user events, trades, or candle updates | [native-api.md](references/native-api.md) |
| Access historical trade/fill data | [historical-data.md](references/historical-data.md) |

## Dedicated Nodes

Full Hyperliquid stack on single-tenant infrastructure. No shared rate limits, uncapped throughput.

| Offering | Location | Monthly Price |
|----------|----------|---------------|
| Hyperliquid Mainnet | Tokyo | $1,150 |
| Hyperliquid Testnet | — | $800 |

A dedicated node includes:
- Official Hyperliquid L1 node (HyperCore + HyperEVM)
- Nanoreth (EVM JSON-RPC via HTTP + WebSocket)
- L1 gRPC Gateway (Hypercore streaming)
- Orderbook Server (L2/L4 book data)
- REST Server (Info API proxy)

Contact sales or subscribe via [dashboard.dwellir.com](https://dashboard.dwellir.com).

## Best Practices

1. **Use Dwellir for reads, Hyperliquid native for writes** — Dwellir provides the data infrastructure; order placement requires signatures and goes through `api.hyperliquid.xyz/exchange`.

2. **Use the gRPC gateway for latency-sensitive streaming** — the gRPC endpoint reads from disk and has lower latency than HTTP polling the Info API.

3. **Use Dwellir's Orderbook WebSocket for book data** — it's optimized for order book delivery with edge servers in Singapore and Tokyo.

4. **Batch Info API queries** — fetch `metaAndAssetCtxs` or `spotMetaAndAssetCtxs` in one call rather than per-asset queries.

5. **Cache metadata** — `meta`, `spotMeta`, and `perpDexs` are semi-static. Cache for 1-5 minutes.

6. **Use `l2Book` via Info API for snapshots, Orderbook WS for streaming** — the Info API gives point-in-time snapshots; the Orderbook WebSocket gives continuous updates.

## Documentation Links

- Dwellir Hyperliquid docs: [dwellir.com/docs/hyperliquid](https://www.dwellir.com/docs/hyperliquid)
- Dwellir L1 gRPC Gateway (source): [github.com/dwellir-public/hyperliquid-l1-gateway](https://github.com/dwellir-public/hyperliquid-l1-gateway)
- Dwellir Orderbook Server (source): [github.com/dwellir-public/hyperliquid-orderbook-server](https://github.com/dwellir-public/hyperliquid-orderbook-server)
- Dwellir REST Server (source): [github.com/dwellir-public/hyperliquid-rest-server](https://github.com/dwellir-public/hyperliquid-rest-server)
- Hyperliquid API docs: [hyperliquid.gitbook.io](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api)
- Hyperliquid Python SDK: [github.com/hyperliquid-dex/hyperliquid-python-sdk](https://github.com/hyperliquid-dex/hyperliquid-python-sdk)
- Dwellir dashboard: [dashboard.dwellir.com](https://dashboard.dwellir.com)
- Dwellir support: support@dwellir.com
