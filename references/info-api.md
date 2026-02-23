# Info API Proxy (Base Plan)

Dwellir proxies a subset of Hyperliquid's `/info` endpoint through a filtering REST server. This validates requests and blocks certain high-risk query types (e.g., `fileSnapshot` is restricted on lower-tier plans). Not all Info API query types are supported — unsupported types return HTTP 422. For unsupported types, fall back to the public endpoint at `https://api.hyperliquid.xyz/info`.

**For the current list of supported query types, see [Dwellir Info API docs](https://www.dwellir.com/docs/hyperliquid/info-endpoint).** The supported types change over time as Dwellir expands proxy coverage.

## Endpoint

Two authentication methods are supported (both equivalent):

**Path-based auth (API key in URL):**
```
POST https://api-hyperliquid-mainnet-info.n.dwellir.com/{API_KEY}/info
Content-Type: application/json
```

**Header-based auth (recommended by Dwellir docs):**
```
POST https://api-hyperliquid-mainnet-info.n.dwellir.com/info
Content-Type: application/json
X-Api-Key: {API_KEY}
```

All Info API requests use `POST` with a JSON body containing a `type` field.

## Market Data Queries

### Get All Mid Prices

**Note:** `allMids` is not supported on the Dwellir proxy. Use the public endpoint.

```javascript
const mids = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'allMids' }),
}).then(r => r.json());
// { "BTC": "66867.5", "ETH": "1935.9", "HYPE": "28.4", ... }
```

### Get Order Book Snapshot

**Note:** `l2Book` is not supported on the Dwellir proxy. Use the public endpoint.

```javascript
const book = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    type: 'l2Book',
    coin: 'BTC',
    nSigFigs: null, // null = full precision, or 2/3/4/5
  }),
}).then(r => r.json());
// book.levels[0] = bids [{ px, sz, n }], book.levels[1] = asks
```

### Get Candle Data

**Note:** `candleSnapshot` is not supported on the Dwellir proxy. Use the public endpoint.

```javascript
const candles = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    type: 'candleSnapshot',
    req: {
      coin: 'BTC',
      interval: '1h', // 1m,3m,5m,15m,30m,1h,2h,4h,8h,12h,1d,3d,1w,1M
      startTime: Date.now() - 86400000,
      endTime: Date.now(),
    },
  }),
}).then(r => r.json());
// [{ t: openTime, T: closeTime, o, h, l, c, v, n, s, i }]
```

## Perpetuals Metadata

### Universe & Asset Contexts (Funding, OI, Volume)

**Note:** `metaAndAssetCtxs` is not supported on the Dwellir proxy. Use the public endpoint.

```javascript
const [meta, assetCtxs] = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'metaAndAssetCtxs' }),
}).then(r => r.json());
// meta.universe = [{ name, szDecimals, maxLeverage, ... }]
// assetCtxs = [{ funding, openInterest, prevDayPx, dayNtlVlm, premium, ... }]
```

### Funding Rate History

**Note:** `fundingHistory` is not supported on the Dwellir proxy. Use the public endpoint.

```javascript
const history = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    type: 'fundingHistory',
    coin: 'BTC',
    startTime: Date.now() - 86400000,
  }),
}).then(r => r.json());
// [{ coin, fundingRate, premium, time }]
```

### Predicted Funding Rates (Cross-Venue)

**Note:** `predictedFundings` is not supported on the Dwellir proxy (returns 422). Use the public endpoint.

```javascript
const predictions = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'predictedFundings' }),
}).then(r => r.json());
// Returns array of [coin, venues] tuples:
// [["BTC", [["HlPerp", { fundingRate, nextFundingTime, fundingIntervalHours }], ["BinPerp", {...}]]], ...]
```

## Spot Metadata

**Note:** `spotMetaAndAssetCtxs` is not supported on the Dwellir proxy. Use the public endpoint. For just the metadata (without live contexts), `spotMeta` works on the proxy.

```javascript
const [meta, assetCtxs] = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'spotMetaAndAssetCtxs' }),
}).then(r => r.json());
// meta.tokens = [{ name, tokenId, szDecimals, ... }]
// meta.universe = [{ name, tokens: [baseIdx, quoteIdx], ... }]
```

## User Account Queries

These require knowing the user's address (blockchain data is public). The types below work on the Dwellir proxy.

### Perpetual Positions & Margin

```javascript
const DWELLIR_INFO = `https://api-hyperliquid-mainnet-info.n.dwellir.com/${process.env.DWELLIR_API_KEY}/info`;

const state = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'clearinghouseState', user: '0x...' }),
}).then(r => r.json());
// state.marginSummary = { accountValue, totalNtlPos, totalRawUsd, totalMarginUsed }
// state.assetPositions = [{ position: { coin, szi, leverage, liquidationPx, ... } }]
```

### Spot Balances

```javascript
const state = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'spotClearinghouseState', user: '0x...' }),
}).then(r => r.json());
// state.balances = [{ coin, hold, total, entryNtl }]
```

### Open Orders

```javascript
const orders = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'frontendOpenOrders', user: '0x...' }),
}).then(r => r.json());
// [{ coin, side, limitPx, sz, oid, orderType, reduceOnly, ... }]
```

### User Fills / Trade History

**Note:** `userFills` is not supported on the Dwellir proxy. Use the public endpoint.

```javascript
const fills = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'userFills', user: '0x...' }),
}).then(r => r.json());
// [{ coin, px, sz, side, dir, closedPnl, fee, time, hash, ... }]
```

### Order Status

**Note:** `orderStatus` is not supported on the Dwellir proxy. Use the public endpoint.

```javascript
const result = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'orderStatus', user: '0x...', oid: 91490942 }),
}).then(r => r.json());
// { status: "order", order: { order: {...}, status: "filled"|"open"|"canceled"|... } }
// or { status: "unknownOid" }
```

## Available Query Types

**For the definitive list of supported types, see [Dwellir Info API docs](https://www.dwellir.com/docs/hyperliquid/info-endpoint).** The proxy coverage expands over time.

**General rule:** If a type returns HTTP 422 on the Dwellir proxy, fall back to the public endpoint at `https://api.hyperliquid.xyz/info`.

Types that typically work on the proxy include: user account queries (`clearinghouseState`, `spotClearinghouseState`, `openOrders`, `frontendOpenOrders`, `userFees`, `userRateLimit`), metadata (`meta`, `spotMeta`, `exchangeStatus`), staking (`delegations`), and several others. Types that typically require the public endpoint include: market data aggregates (`allMids`, `metaAndAssetCtxs`, `l2Book`, `candleSnapshot`), historical queries (`userFills`, `fundingHistory`, `historicalOrders`), and portfolio/referral data.

`fileSnapshot` (full L4 order book dump) is restricted on lower-tier plans and returns HTTP 403.

## Coin Naming Conventions

| Context | Format | Example |
|---------|--------|---------|
| Perpetual | Coin name | `"BTC"`, `"ETH"`, `"HYPE"` |
| Spot | `@{tokenIndex}` | `"@1"` (PURR), `"@150"` |
| Spot (display) | `SYMBOL/USDC` | `"PURR/USDC"` |
| HIP-3 DEX token | `dexname:SYMBOL` | `"xyz:XYZ100"` |

## Common Patterns

### Market Data Dashboard

```javascript
// allMids, metaAndAssetCtxs, and l2Book are not on the Dwellir proxy — use public endpoint
const HL_INFO = 'https://api.hyperliquid.xyz/info';
const post = (body) => fetch(HL_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(body),
}).then(r => r.json());

const [mids, meta, book] = await Promise.all([
  post({ type: 'allMids' }),
  post({ type: 'metaAndAssetCtxs' }),
  post({ type: 'l2Book', coin: 'BTC' }),
]);

console.log(`BTC mid: $${mids.BTC}`);
console.log(`BTC OI: ${meta[1][0].openInterest}`);
console.log(`BTC funding: ${meta[1][0].funding}`);
console.log(`BTC book: ${book.levels[0].length} bid levels`);
```

### Funding Rate Monitor

```javascript
// predictedFundings is not proxied — use public endpoint
const predictions = await fetch('https://api.hyperliquid.xyz/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'predictedFundings' }),
}).then(r => r.json());

// Response is array of [coin, venues] tuples
// Each venue is [venueName, { fundingRate, nextFundingTime, fundingIntervalHours }]
for (const [coin, venues] of predictions) {
  const hl = venues.find(([name]) => name === 'HlPerp');
  const bin = venues.find(([name]) => name === 'BinPerp');
  if (hl && bin) {
    const hlRate = hl[1].fundingRate;
    const binRate = bin[1].fundingRate;
    const diff = Math.abs(parseFloat(hlRate) - parseFloat(binRate));
    if (diff > 0.001) {
      console.log(`${coin}: HL=${hlRate} vs Binance=${binRate} (diff: ${diff.toFixed(6)})`);
    }
  }
}
```

### Account Health Monitor

```javascript
const DWELLIR_INFO = `https://api-hyperliquid-mainnet-info.n.dwellir.com/${process.env.DWELLIR_API_KEY}/info`;

async function checkAccountHealth(userAddress) {
  const state = await fetch(DWELLIR_INFO, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ type: 'clearinghouseState', user: userAddress }),
  }).then(r => r.json());

  const { accountValue, totalMarginUsed } = state.marginSummary;
  const marginRatio = parseFloat(totalMarginUsed) / parseFloat(accountValue);

  console.log(`Account value: $${parseFloat(accountValue).toFixed(2)}`);
  console.log(`Margin used: ${(marginRatio * 100).toFixed(1)}%`);

  for (const { position: pos } of state.assetPositions) {
    console.log(`  ${pos.coin}: ${pos.szi} @ ${pos.entryPx} (liq: ${pos.liquidationPx})`);
  }

  if (marginRatio > 0.8) {
    console.warn('WARNING: Margin utilization above 80%');
  }
}
```

## Tips

- Paginate fills: `userFills` returns max 2000 entries. Use `userFillsByTime` with the last timestamp for pagination.
- Check exchange status for staleness: query `exchangeStatus` to verify the L1 timestamp. Reject stale data.
- Use `l2Book` via Info API for snapshots, Orderbook WS for streaming.
