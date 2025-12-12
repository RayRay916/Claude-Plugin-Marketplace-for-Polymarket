# Polymarket Developer Tools - Claude Code Plugin Marketplace

A comprehensive suite of modular Claude Code plugins for the Polymarket prediction market platform. Each plugin provides documentation, code generation, and expert assistance for a specific API domain.

## Overview

This marketplace contains **4 specialized plugins** covering all Polymarket APIs:

1. **polymarket-clob** - Central Limit Order Book API for trading
2. **polymarket-data** - User data, positions, and analytics
3. **polymarket-gamma** - Market discovery and exploration
4. **polymarket-websocket** - Real-time data streams

Each plugin includes:
- 📚 **Comprehensive documentation** via slash commands
- 🤖 **Expert AI assistant** for questions and code generation
- 🐍 **Python examples** using official Polymarket clients
- 📘 **TypeScript examples** for web/Node.js applications

## Installation

### Add the Marketplace

```bash
# Navigate to this directory
cd claude-polymarket

# Add the marketplace to Claude Code
/plugin marketplace add .
```

### Install Plugins

Install all plugins or pick specific ones:

```bash
# Install all plugins
/plugin install polymarket-clob
/plugin install polymarket-data
/plugin install polymarket-gamma
/plugin install polymarket-websocket

# Or install selectively based on your needs
```

## Plugin Details

### 1. Polymarket CLOB (Trading API)

**ID**: `polymarket-clob`

Central Limit Order Book for trading operations.

**Slash Commands**:
- `/clob-intro` - CLOB overview and quickstart
- `/clob-auth` - Authentication setup (L1/L2, API keys)
- `/clob-orderbook` - Orderbook data endpoints
- `/clob-pricing` - Pricing and spreads
- `/clob-orders` - Order management (create, cancel, get)
- `/clob-trades` - Trade history and analytics

**Agent**: `@clob-assistant`

Use when you need help with:
- Trading operations
- Order placement and management
- Orderbook analysis
- Price calculations
- Authentication setup

**Example Usage**:
```
You: How do I place a market buy order?
@clob-assistant: [Provides code examples in Python and TypeScript]

You: /clob-orders
[Shows comprehensive order management guide]
```

---

### 2. Polymarket Data API

**ID**: `polymarket-data`

User data, portfolio tracking, and analytics.

**Slash Commands**:
- `/data-intro` - Data API overview
- `/data-positions` - User positions and holdings
- `/data-activity` - Trading activity and history
- `/data-leaderboard` - Rankings and leaderboards

**Agent**: `@data-assistant`

Use when you need help with:
- Portfolio tracking
- User positions and P&L
- Activity feeds
- Leaderboard data
- Performance analytics

**Example Usage**:
```
You: How do I get a user's portfolio value?
@data-assistant: [Provides code to fetch and calculate portfolio value]

You: /data-positions
[Shows position tracking examples]
```

---

### 3. Polymarket Gamma (Market Discovery)

**ID**: `polymarket-gamma`

Market exploration, events, and search.

**Slash Commands**:
- `/gamma-intro` - Gamma API overview
- `/gamma-events` - Events and series endpoints
- `/gamma-markets` - Market data and filtering
- `/gamma-search` - Search and discovery

**Agent**: `@gamma-assistant`

Use when you need help with:
- Finding markets
- Building market browsers
- Search functionality
- Event organization
- Category filtering

**Example Usage**:
```
You: How do I get the top 10 markets by volume?
@gamma-assistant: [Shows how to query and sort markets]

You: /gamma-search
[Shows search implementation examples]
```

---

### 4. Polymarket WebSocket (Real-time Data)

**ID**: `polymarket-websocket`

Real-time streaming for live trading and market data.

**Slash Commands**:
- `/ws-intro` - WebSocket overview and setup
- `/ws-user-channel` - User-specific real-time updates
- `/ws-market-channel` - Market data streams

**Agent**: `@ws-assistant`

Use when you need help with:
- Real-time orderbook updates
- Live trade notifications
- User order fills
- WebSocket connections
- Event streaming

**Example Usage**:
```
You: How do I subscribe to real-time orderbook updates?
@ws-assistant: [Shows WebSocket connection and subscription code]

You: /ws-market-channel
[Displays market streaming documentation]
```

## Quick Start Guide

### 1. Choose Your Use Case

**Building a trading bot?**
→ Install `polymarket-clob` + `polymarket-websocket`

**Building a portfolio tracker?**
→ Install `polymarket-data`

**Building a market browser?**
→ Install `polymarket-gamma`

**Building a complete trading interface?**
→ Install all plugins

### 2. Get API Credentials

For trading operations, you'll need Polymarket API credentials:

```bash
# Create .env file
POLYGON_PRIVATE_KEY=0x...
POLYMARKET_API_KEY=your_api_key
POLYMARKET_SECRET=your_secret
POLYMARKET_PASSPHRASE=your_passphrase
```

See `/clob-auth` for detailed authentication setup.

### 3. Start Coding

Use slash commands for documentation:
```
/clob-intro    # Learn about the CLOB API
/data-intro    # Learn about the Data API
/gamma-intro   # Learn about the Gamma API
/ws-intro      # Learn about WebSocket streaming
```

Ask the AI assistants for help:
```
@clob-assistant How do I create a buy order?
@data-assistant How do I calculate P&L?
@gamma-assistant How do I search for markets?
@ws-assistant How do I connect to real-time data?
```

## Code Examples

All commands provide examples in **both Python and TypeScript**.

### Python
```python
from py_clob_client.client import ClobClient

client = ClobClient(
    host="https://clob.polymarket.com",
    chain_id=137
)

# Get orderbook
orderbook = client.get_order_book(token_id)
```

### TypeScript
```typescript
import { ClobClient } from '@polymarket/clob-client';

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  chainId: 137,
});

// Get orderbook
const orderbook = await client.getOrderBook(tokenId);
```

## Plugin Architecture

```
claude-polymarket/
├── .claude-plugin/
│   └── marketplace.json         # Marketplace definition
├── plugins/
│   ├── polymarket-clob/         # CLOB API Plugin
│   │   ├── plugin.json
│   │   ├── commands/            # 6 slash commands
│   │   └── agents/              # clob-assistant
│   ├── polymarket-data/         # Data API Plugin
│   │   ├── plugin.json
│   │   ├── commands/            # 4 slash commands
│   │   └── agents/              # data-assistant
│   ├── polymarket-gamma/        # Gamma API Plugin
│   │   ├── plugin.json
│   │   ├── commands/            # 4 slash commands
│   │   └── agents/              # gamma-assistant
│   └── polymarket-websocket/    # WebSocket Plugin
│       ├── plugin.json
│       ├── commands/            # 3 slash commands
│       └── agents/              # ws-assistant
└── README.md                    # This file
```

## Feature Matrix

| Feature | CLOB | Data | Gamma | WebSocket |
|---------|------|------|-------|-----------|
| **Trading** | ✅ | ❌ | ❌ | ✅ |
| **Orderbook** | ✅ | ❌ | ❌ | ✅ |
| **Positions** | ❌ | ✅ | ❌ | ✅ |
| **Market Discovery** | ❌ | ❌ | ✅ | ❌ |
| **Search** | ❌ | ❌ | ✅ | ❌ |
| **Real-time** | ❌ | ❌ | ❌ | ✅ |
| **Analytics** | ✅ | ✅ | ✅ | ❌ |
| **Auth Required** | Trading only | Optional | No | User data only |

## Common Workflows

### Workflow 1: Build a Trading Bot

```bash
# Install required plugins
/plugin install polymarket-clob
/plugin install polymarket-websocket

# Learn the basics
/clob-intro
/ws-intro

# Get help implementing
@clob-assistant How do I authenticate?
@ws-assistant How do I subscribe to orderbook updates?
```

### Workflow 2: Build a Portfolio Dashboard

```bash
# Install required plugins
/plugin install polymarket-data
/plugin install polymarket-gamma

# Learn the APIs
/data-intro
/gamma-intro

# Get implementation help
@data-assistant How do I fetch user positions?
@gamma-assistant How do I get market metadata?
```

### Workflow 3: Build a Market Browser

```bash
# Install required plugin
/plugin install polymarket-gamma

# Learn market discovery
/gamma-markets
/gamma-search

# Get help
@gamma-assistant How do I filter markets by category?
```

## Resources

### Official Documentation
- [Polymarket Docs](https://docs.polymarket.com/)
- [CLOB API](https://docs.polymarket.com/developers/CLOB/introduction)
- [Gamma API](https://docs.polymarket.com/developers/gamma)
- [WebSocket API](https://docs.polymarket.com/developers/websocket)

### Python Client
- [py-clob-client](https://github.com/Polymarket/py-clob-client)
```bash
pip install py-clob-client
```

### TypeScript Client
- [@polymarket/clob-client](https://www.npmjs.com/package/@polymarket/clob-client)
```bash
npm install @polymarket/clob-client
```

## Support

### Using the Plugins

Each plugin includes:
- Comprehensive slash command documentation
- Expert AI assistant for questions
- Code examples in Python and TypeScript
- Best practices and common patterns

### Getting Help

1. **Use slash commands** for reference documentation
2. **Ask the AI assistants** for specific implementation help
3. **Check official docs** for API specifications
4. **Report issues** to plugin maintainers

## Contributing

Want to improve these plugins?

1. Fork the repository
2. Make your changes
3. Test with `/plugin validate .`
4. Submit a pull request

## License

MIT License - see LICENSE file for details

## About

Created by the Polymarket developer community to help builders integrate with Polymarket's prediction market platform.

**Version**: 1.0.0
**Last Updated**: 2025-12-12

---

**Happy Building! 🚀**

Start with `/clob-intro`, `/data-intro`, `/gamma-intro`, or `/ws-intro` to begin your Polymarket integration journey.
