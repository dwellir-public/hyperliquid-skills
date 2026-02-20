# Orderbook WebSocket (Premium — $199/mo)

Real-time L2 order book data served by Dwellir's [order book server](https://github.com/dwellir-public/hyperliquid-orderbook-server), which reads Hypercore data directly from disk. **WSS only** — HTTP requests are not supported.

3-day free trial available.

## Why Dwellir's Orderbook vs Public Hyperliquid

| Feature | Dwellir Orderbook | Public Hyperliquid WS |
|---------|-------------------|----------------------|
| **Book depth** | Up to **100 levels** per side (configurable via `n_levels`) | Max 20 levels per side |
| **Spot markets** | Yes — perpetuals, spot (`@{index}`), and HIP-3 DEX tokens | Perpetuals only on public l2Book |
| **HIP-3 DEX markets** | Yes — all builder-deployed perp markets | Limited |
| **Rate limits** | API key-based, no IP limits | Introducing IP-based rate limits |
| **L4 full book** | Yes — complete order-level diffs | Not available on public WS |
| **Infrastructure** | Dedicated edge servers (Singapore, Tokyo) | Shared public endpoint |

## Endpoint

```
WSS: wss://api-hyperliquid-mainnet-orderbook.n.dwellir.com/{API_KEY}/ws
```

**Critical:** The `/ws` suffix after the API key is required. Without it, the connection drops immediately.

## Subscription Types

| Type | Description |
|------|-------------|
| `l2Book` | Aggregated order book with configurable depth (up to 100 levels) |
| `l4Book` | Full order book with individual order diffs |
| `trades` | Trade stream (perpetuals and spot when enabled) |

## Benchmarked Message Rates

| Data Type | Messages/sec | Monthly Messages (per pair) |
|-----------|-------------|----------------------------|
| L2 Orderbook | ~10.4 msg/s | ~27M |
| L4 Orderbook | ~9.0 msg/s | ~23M |

## Connection

```javascript
const ws = new WebSocket(
  `wss://api-hyperliquid-mainnet-orderbook.n.dwellir.com/${process.env.DWELLIR_API_KEY}/ws`
);

ws.on('open', () => {
  // Subscribe with 100 levels of depth (default: 20, max: 100)
  ws.send(JSON.stringify({
    method: 'subscribe',
    subscription: { type: 'l2Book', coin: 'ETH', n_levels: 100 }
  }));
});

ws.on('message', (data) => {
  const update = JSON.parse(data);
  // update.levels[0] = bids, update.levels[1] = asks
  // Each level: { px: "price", sz: "size", n: numOrders }
  console.log('Book update:', update);
});

// Subscribe to a spot market
ws.send(JSON.stringify({
  method: 'subscribe',
  subscription: { type: 'l2Book', coin: '@107' } // spot token by index
}));

// Subscribe to a HIP-3 DEX market
ws.send(JSON.stringify({
  method: 'subscribe',
  subscription: { type: 'l2Book', coin: 'xyz:XYZ100' }
}));

// Unsubscribe
ws.send(JSON.stringify({
  method: 'unsubscribe',
  subscription: { type: 'l2Book', coin: 'ETH' }
}));
```

## Market-Making Example

```javascript
const ws = new WebSocket(
  `wss://api-hyperliquid-mainnet-orderbook.n.dwellir.com/${process.env.DWELLIR_API_KEY}/ws`
);

ws.on('open', () => {
  ws.send(JSON.stringify({
    method: 'subscribe',
    subscription: { type: 'l2Book', coin: 'ETH' }
  }));
});

ws.on('message', (data) => {
  const msg = JSON.parse(data);
  const bestBid = msg.levels[0][0]; // { px, sz, n }
  const bestAsk = msg.levels[1][0];
  const spread = parseFloat(bestAsk.px) - parseFloat(bestBid.px);
  const mid = (parseFloat(bestBid.px) + parseFloat(bestAsk.px)) / 2;

  console.log(`ETH mid: ${mid.toFixed(2)}, spread: ${spread.toFixed(2)}`);

  // Place/update orders via Hyperliquid Exchange API (not Dwellir)
  // exchange.order("ETH", true, size, mid - offset, {"limit": {"tif": "Alo"}});
  // exchange.order("ETH", false, size, mid + offset, {"limit": {"tif": "Alo"}});
});
```

## Use Cases

- Market making — 100-level depth gives full picture of liquidity
- Arbitrage — compare order books across venues with deeper data
- Liquidity analysis — track depth changes across spot, perp, and HIP-3 markets
- Trading signals — detect large order placement/removal deeper in the book
