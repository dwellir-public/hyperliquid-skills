# Hyperliquid Native API (Not Dwellir)

These endpoints are operated by Hyperliquid directly. They are documented here for completeness. A typical application uses Dwellir for reading data and these native endpoints for writing.

## Exchange API (Order Placement)

Order placement, cancellation, transfers, and other write operations go directly to Hyperliquid's Exchange API. These require EIP-712 signatures from your wallet and are **not** proxied by Dwellir.

**Endpoint:** `POST https://api.hyperliquid.xyz/exchange`

### Python SDK

```python
from hyperliquid.utils import constants
from hyperliquid.exchange import Exchange
from eth_account import Account
import os

wallet = Account.from_key(os.environ["PRIVATE_KEY"])
exchange = Exchange(wallet, constants.MAINNET_API_URL)

# Limit buy 0.1 BTC at $100,000
result = exchange.order("BTC", True, 0.1, 100000, {"limit": {"tif": "Gtc"}})

# Cancel an order
exchange.cancel("BTC", order_id)
```

For full Exchange API documentation: [Hyperliquid Exchange Endpoint docs](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/exchange-endpoint)

## Native WebSocket

For real-time user events, trades, candle updates, and position changes, use Hyperliquid's native WebSocket directly. This is separate from Dwellir's Orderbook WebSocket.

**WebSocket URL:** `wss://api.hyperliquid.xyz/ws`

### Subscription Types

| Type | Description |
|------|-------------|
| `allMids` | All mid prices |
| `trades` | Trade executions per coin |
| `candle` | OHLCV candle updates |
| `l2Book` | Order book updates |
| `bbo` | Best bid/offer (lighter than l2Book) |
| `orderUpdates` | User order status changes |
| `userEvents` | Fills, funding, liquidations |
| `userFills` | Trade executions with snapshots |
| `clearinghouseState` | Position/margin updates |

### Connection

```javascript
const ws = new WebSocket('wss://api.hyperliquid.xyz/ws');

ws.on('open', () => {
  ws.send(JSON.stringify({
    method: 'subscribe',
    subscription: { type: 'trades', coin: 'BTC' }
  }));
});

ws.on('message', (data) => {
  const msg = JSON.parse(data);
  if (msg.channel === 'trades') {
    for (const trade of msg.data) {
      console.log(`${trade.side} ${trade.sz} BTC @ ${trade.px}`);
    }
  }
});
```

For full WebSocket documentation: [Hyperliquid WebSocket docs](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/websocket)
