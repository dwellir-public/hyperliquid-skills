# HyperEVM JSON-RPC (Base Plan)

Standard EVM JSON-RPC served by [Nanoreth](https://github.com/hl-archive-node/nanoreth) (a reth fork for Hyperliquid EVM data). Supports HTTP and WebSocket. For the full list of supported EVM methods, see [Dwellir HyperEVM docs](https://www.dwellir.com/docs/hyperliquid).

## Endpoint

```
HTTPS: https://api-hyperliquid-mainnet-evm.n.dwellir.com/{API_KEY}
WSS:   wss://api-hyperliquid-mainnet-evm.n.dwellir.com/{API_KEY}
```

## Connection

### ethers.js

```typescript
import { JsonRpcProvider } from 'ethers';

const provider = new JsonRpcProvider(
  `https://api-hyperliquid-mainnet-evm.n.dwellir.com/${process.env.DWELLIR_API_KEY}`
);

// Standard EVM methods work
const blockNumber = await provider.getBlockNumber();
const balance = await provider.getBalance('0x...');
const chainId = await provider.getNetwork(); // chainId: 999
```

### viem

```typescript
import { createPublicClient, http } from 'viem';

const client = createPublicClient({
  chain: {
    id: 999,
    name: 'Hyperliquid',
    nativeCurrency: { name: 'HYPE', symbol: 'HYPE', decimals: 18 },
    rpcUrls: {
      default: {
        http: [`https://api-hyperliquid-mainnet-evm.n.dwellir.com/${process.env.DWELLIR_API_KEY}`],
      },
    },
  },
  transport: http(),
});
```

## Supported Methods

All standard Ethereum JSON-RPC methods: `eth_blockNumber`, `eth_getBalance`, `eth_call`, `eth_getTransactionByHash`, `eth_getBlockByNumber`, `eth_getLogs`, `eth_sendRawTransaction`, etc.

## Use Cases

- Deploy and interact with Solidity contracts on HyperEVM
- Query EVM state (balances, contract storage, logs)
- Monitor EVM-side events and transactions
- Build HyperEVM dApps that interact with HyperCore liquidity
