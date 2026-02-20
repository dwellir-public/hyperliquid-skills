# Info API Proxy (Base Plan)

Dwellir proxies Hyperliquid's `/info` endpoint through a [filtering REST server](https://github.com/dwellir-public/hyperliquid-rest-server). This validates requests and blocks certain high-risk query types (e.g., `fileSnapshot` is restricted on lower-tier plans).

## Endpoint

```
POST https://api-hyperliquid-mainnet.n.dwellir.com/{API_KEY}/info
Content-Type: application/json
```

All Info API requests use `POST` with a JSON body containing a `type` field.

## Market Data Queries

### Get All Mid Prices

```javascript
const DWELLIR_INFO = `https://api-hyperliquid-mainnet.n.dwellir.com/${process.env.DWELLIR_API_KEY}/info`;

const mids = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'allMids' }),
}).then(r => r.json());
// { "BTC": "113377.0", "ETH": "3245.5", "HYPE": "28.4", ... }
```

### Get Order Book Snapshot

```javascript
const book = await fetch(DWELLIR_INFO, {
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

```javascript
const candles = await fetch(DWELLIR_INFO, {
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

```javascript
const [meta, assetCtxs] = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'metaAndAssetCtxs' }),
}).then(r => r.json());
// meta.universe = [{ name, szDecimals, maxLeverage, ... }]
// assetCtxs = [{ funding, openInterest, prevDayPx, dayNtlVlm, premium, ... }]
```

### Funding Rate History

```javascript
const history = await fetch(DWELLIR_INFO, {
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

```javascript
const predictions = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'predictedFundings' }),
}).then(r => r.json());
// Predicted rates across Hyperliquid, Binance, Bybit, etc.
```

## Spot Metadata

```javascript
const [meta, assetCtxs] = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'spotMetaAndAssetCtxs' }),
}).then(r => r.json());
// meta.tokens = [{ name, tokenId, szDecimals, ... }]
// meta.universe = [{ name, tokens: [baseIdx, quoteIdx], ... }]
```

## User Account Queries

These require knowing the user's address (blockchain data is public).

### Perpetual Positions & Margin

```javascript
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

```javascript
const fills = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'userFills', user: '0x...' }),
}).then(r => r.json());
// [{ coin, px, sz, side, dir, closedPnl, fee, time, hash, ... }]
```

### Order Status

```javascript
const result = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'orderStatus', user: '0x...', oid: 91490942 }),
}).then(r => r.json());
// { status: "order", order: { order: {...}, status: "filled"|"open"|"canceled"|... } }
// or { status: "unknownOid" }
```

## Available Query Types

All of these are available through Dwellir's Info API proxy unless noted.

| Type | Description | Risk Level |
|------|-------------|------------|
| `meta` | Perpetuals metadata (universe, margin tables) | LOW |
| `spotMeta` | Spot token metadata and trading pairs | LOW |
| `metaAndAssetCtxs` | Combined perp metadata + live market data | LOW |
| `spotMetaAndAssetCtxs` | Combined spot metadata + live market data | LOW |
| `exchangeStatus` | Exchange status with L1 timestamp | LOW |
| `allMids` | Mid prices for all coins | LOW |
| `l2Book` | Order book snapshot (20 levels/side) | LOW |
| `candleSnapshot` | OHLCV candle data (up to 5000 candles) | LOW |
| `clearinghouseState` | User perp positions and margin | MEDIUM |
| `spotClearinghouseState` | User spot balances | MEDIUM |
| `openOrders` | User's active orders | MEDIUM |
| `frontendOpenOrders` | Active orders with extra metadata | MEDIUM |
| `orderStatus` | Single order status by ID | MEDIUM |
| `userFills` | User fill history (max 2000) | MEDIUM |
| `userFillsByTime` | Paginated fills by time range | MEDIUM |
| `historicalOrders` | Recent order history (max 2000) | MEDIUM |
| `userFunding` | User funding payment history | MEDIUM |
| `userFees` | Fee schedule, volume, discounts | MEDIUM |
| `userRateLimit` | API rate limit status | LOW |
| `fundingHistory` | Historical funding rates for a coin | LOW |
| `predictedFundings` | Predicted funding across venues | LOW |
| `activeAssetData` | User leverage, max trade sizes per coin | MEDIUM |
| `delegations` | User staking delegations | MEDIUM |
| `delegatorSummary` | Staking summary | LOW |
| `subAccounts` | User sub-account list | MEDIUM-HIGH |
| `vaultDetails` | Vault info, followers, P&L | LOW |
| `userVaultEquities` | User's vault deposits | MEDIUM |
| `portfolio` | User P&L history (day/week/month/all) | MEDIUM |
| `referral` | Referral rewards and status | MEDIUM |
| `maxBuilderFee` | Builder fee approval check | LOW |
| `perpDexs` | All HIP-3 perpetual DEXes | LOW |
| `validatorL1Votes` | Validator governance votes | LOW |
| `borrowLendUserState` | User borrow/lend positions | MEDIUM |
| `allBorrowLendReserveStates` | All reserve interest rates | LOW |
| `tokenDetails` | Token supply and deployment info | LOW |
| `fileSnapshot` | **Restricted on Free/Starter plans** — full L4 order book dump | CRITICAL |

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
const DWELLIR_INFO = `https://api-hyperliquid-mainnet.n.dwellir.com/${process.env.DWELLIR_API_KEY}/info`;

// Fetch all key market data in parallel
const [mids, meta, book] = await Promise.all([
  fetch(DWELLIR_INFO, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ type: 'allMids' }),
  }).then(r => r.json()),

  fetch(DWELLIR_INFO, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ type: 'metaAndAssetCtxs' }),
  }).then(r => r.json()),

  fetch(DWELLIR_INFO, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ type: 'l2Book', coin: 'BTC' }),
  }).then(r => r.json()),
]);

console.log(`BTC mid: $${mids.BTC}`);
console.log(`BTC OI: ${meta[1][0].openInterest}`);
console.log(`BTC funding: ${meta[1][0].funding}`);
console.log(`BTC book: ${book.levels[0].length} bid levels`);
```

### Funding Rate Monitor

```javascript
const DWELLIR_INFO = `https://api-hyperliquid-mainnet.n.dwellir.com/${process.env.DWELLIR_API_KEY}/info`;

const predictions = await fetch(DWELLIR_INFO, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ type: 'predictedFundings' }),
}).then(r => r.json());

// Compare Hyperliquid vs other venues
for (const [coin, venues] of Object.entries(predictions)) {
  const hlRate = venues.find(v => v.venue === 'Hyperliquid')?.rate;
  const binanceRate = venues.find(v => v.venue === 'Binance')?.rate;
  if (hlRate && binanceRate) {
    const diff = Math.abs(parseFloat(hlRate) - parseFloat(binanceRate));
    if (diff > 0.001) {
      console.log(`${coin}: HL=${hlRate} vs Binance=${binanceRate} (diff: ${diff.toFixed(6)})`);
    }
  }
}
```

### Account Health Monitor

```javascript
const DWELLIR_INFO = `https://api-hyperliquid-mainnet.n.dwellir.com/${process.env.DWELLIR_API_KEY}/info`;

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
