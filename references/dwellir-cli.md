# Dwellir CLI for Hyperliquid Development

The Dwellir CLI (`dwellir`) gives terminal access to endpoint discovery, API key management, usage monitoring, and documentation -- useful when building on Hyperliquid without leaving the editor or terminal.

Install: `brew install dwellir-public/homebrew-tap/dwellir` (macOS), `yay -S dwellir-cli-bin` (Arch), or `curl -fsSL https://raw.githubusercontent.com/dwellir-public/cli/main/scripts/install.sh | sh`.

Full install options and CLI docs: [dwellir.com/docs/cli](https://www.dwellir.com/docs/cli)

## Authentication

```bash
# Browser-based login
dwellir auth login

# Headless / CI (use a CLI token from dashboard.dwellir.com)
dwellir auth login --token <CLI_TOKEN>

# Or set token via environment variable
export DWELLIR_TOKEN=<CLI_TOKEN>

# Check current auth status
dwellir auth status
```

## Discover Hyperliquid Endpoints

```bash
# List all Hyperliquid endpoints with your API key substituted
dwellir endpoints search hyperliquid

# Output:
#  CHAIN                            PROTOCOL  ENDPOINT
#  Hyperliquid EVM                  https     https://api-hyperliquid-mainnet-evm.n.dwellir.com/<key>
#  Hyperliquid EVM                  wss       wss://api-hyperliquid-mainnet-evm.n.dwellir.com/<key>
#  Hyperliquid HyperCore Info       https     https://api-hyperliquid-mainnet-info.n.dwellir.com/<key>/info
#  Hyperliquid HyperCore Orderbook  wss       wss://api-hyperliquid-mainnet-orderbook.n.dwellir.com/<key>/ws
#  Hyperliquid HyperCore gRPC       https     https://api-hyperliquid-mainnet-grpc.n.dwellir.com/<key>

# Get details for a specific chain
dwellir endpoints get "hyperliquid evm"

# JSON output for scripting or agent consumption
dwellir endpoints search hyperliquid --json
```

## Search and Read Documentation

```bash
# Find all Hyperliquid doc pages
dwellir docs search hyperliquid

# Fetch a specific doc page as markdown
dwellir docs get hyperliquid
dwellir docs get hyperliquid/info-endpoint
dwellir docs get hyperliquid/grpc
dwellir docs get hyperliquid/order-book-server
dwellir docs get hyperliquid/historical-data

# Also works with full URLs
dwellir docs get https://www.dwellir.com/docs/hyperliquid/historical-data

# Search across all Dwellir docs
dwellir docs search "order book"
dwellir docs search grpc
```

This is particularly useful for AI agents that need to pull up-to-date documentation inline.

## Manage API Keys

```bash
# List all keys
dwellir keys list

# Create a key for a Hyperliquid project
dwellir keys create --name my-hl-bot --daily-quota 50000

# Disable a key without deleting it
dwellir keys disable <key-id>

# Re-enable
dwellir keys enable <key-id>

# Delete permanently
dwellir keys delete <key-id>
```

## Monitor Usage

```bash
# Current billing cycle summary
dwellir usage summary

# Usage broken down by day
dwellir usage history --interval day

# View recent errors (e.g., rate-limited requests)
dwellir logs errors --status-code 429 --limit 20

# Error stats aggregated
dwellir logs stats
```

## Output Modes

Every command supports `--human` (default), `--json`, or `--toon`:

```bash
# Machine-readable JSON for scripts and agents
dwellir endpoints search hyperliquid --json

# JSON uses a standard envelope:
# { "ok": true, "data": [...], "meta": { "command": "...", "timestamp": "..." } }
```

When stdout is not a terminal (piped or in CI), the CLI automatically outputs JSON.

## Per-Project Profiles

Bind a Dwellir profile to a project directory with a `.dwellir.json` file:

```json
{ "profile": "hl-prod" }
```

This lets you use different API keys or accounts for different Hyperliquid projects without passing `--profile` every time.

## Links

- CLI docs: [dwellir.com/docs/cli](https://www.dwellir.com/docs/cli)
- CLI source: [github.com/dwellir-public/cli](https://github.com/dwellir-public/cli)
- Dashboard (create CLI tokens): [dashboard.dwellir.com](https://dashboard.dwellir.com)
