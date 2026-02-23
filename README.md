# Hyperliquid Skill for AI Agents

An agent skill that gives Claude Code (and other AI coding agents) deep knowledge of Hyperliquid's infrastructure through [Dwellir](https://www.dwellir.com/), including HyperEVM JSON-RPC, Info API, L1 gRPC streaming, and real-time order book WebSocket.

## Install

```bash
npx skills add dwellir-public/hyperliquid-skills
```

This installs the skill into your project's `.claude/skills/` directory. Works with [Claude Code](https://code.claude.com/), [Cursor](https://cursor.sh/), [Windsurf](https://codeium.com/windsurf), and other agents that support the [Agent Skills](https://agentskills.io) standard.

After installation, the skill activates automatically when your agent encounters Hyperliquid-related tasks. No manual invocation needed.

## What the Agent Learns

Once installed, your agent knows how to:

- **Connect to Dwellir's Hyperliquid endpoints** with correct hostnames, auth methods, and connection patterns
- **Query the Info API** knowing which types are proxied vs public-only, with working code examples
- **Stream order book data** via L2/L4 subscriptions on Dwellir's WebSocket with up to 100 levels of depth
- **Use L1 gRPC streaming** for block, fill, and order book snapshot streams from HyperCore
- **Deploy on HyperEVM** using ethers.js and viem setup with chain ID 999
- **Place orders via native API** through Hyperliquid's Exchange API with the Python SDK
- **Choose the right endpoint** for read (Dwellir) vs write (Hyperliquid native) architecture

The skill references [Dwellir's public docs](https://www.dwellir.com/docs/hyperliquid) for information that changes over time (supported Info API types, pricing, gRPC methods), so it stays current without needing skill updates.

## Repo Structure

```
SKILL.md                          # Main skill entry point (auto-loaded by agent)
references/
  hyperevm-json-rpc.md            # HyperEVM JSON-RPC endpoints and examples
  info-api.md                     # Info API proxy, supported types, code patterns
  orderbook-websocket.md          # L2/L4 order book WebSocket API
  grpc-gateway.md                 # L1 gRPC streaming (blocks, fills, snapshots)
  native-api.md                   # Hyperliquid Exchange API and native WebSocket
  historical-data.md              # Archival, OHLCV, and tick data products
```

## Getting a Dwellir API Key

Sign up at [dashboard.dwellir.com](https://dashboard.dwellir.com) to get an API key for HyperEVM JSON-RPC. The Info API proxy is currently in beta; contact support@dwellir.com for access. The gRPC gateway and order book server are premium add-ons.

## License

MIT
