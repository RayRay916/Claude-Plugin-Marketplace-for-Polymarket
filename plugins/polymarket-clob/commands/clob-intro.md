# Polymarket CLOB API - Introduction

## Overview
The Central Limit Order Book (CLOB) API is Polymarket's core trading infrastructure. It provides endpoints for:
- **Orderbook data** (depth, summaries)
- **Pricing information** (market prices, midpoints, historical data)
- **Order management** (create, cancel, retrieve orders)
- **Trade execution** (trade history, execution data)

**Base URL**: `https://clob.polymarket.com/`

## Architecture
The CLOB operates on Polygon (Layer 2) for fast, low-cost trades:
- **L1**: Ethereum mainnet (CTF Exchange contract)
- **L2**: Polygon (trading operations)
- **Settlement**: Trades settled on-chain via CTF (Conditional Token Framework)

## Authentication
All trading operations require authentication. See `/clob-auth` for details on:
- API Key authentication
- L2 signature-based auth
- Builder credentials

## Quick Start

### Python
```python
from py_clob_client.client import ClobClient

# Initialize client
client = ClobClient(
    host="https://clob.polymarket.com",
    key="YOUR_API_KEY",
    chain_id=137  # Polygon
)

# Get orderbook for a market
orderbook = client.get_order_book(token_id="YOUR_TOKEN_ID")
print(orderbook)
```

### TypeScript
```typescript
import { ClobClient } from '@polymarket/clob-client';

// Initialize client
const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  key: 'YOUR_API_KEY',
  chainId: 137, // Polygon
});

// Get orderbook for a market
const orderbook = await client.getOrderBook('YOUR_TOKEN_ID');
console.log(orderbook);
```

## Key Concepts

### Token IDs
Each market outcome has a unique token ID. You'll need token IDs to:
- Query orderbook data
- Place orders
- Get pricing information

### Condition IDs
Markets are identified by condition IDs. One condition can have multiple outcomes (token IDs).

### Tick Size
Prices are represented in tick size (0.01 = 1 cent). Price range: 0.01 to 0.99.

## Available Commands
- `/clob-intro` - This overview
- `/clob-auth` - Authentication setup
- `/clob-orderbook` - Orderbook data endpoints
- `/clob-pricing` - Pricing and spreads
- `/clob-orders` - Order management
- `/clob-trades` - Trade history

## Resources
- [Official Docs](https://docs.polymarket.com/developers/CLOB/introduction)
- [py-clob-client](https://github.com/Polymarket/py-clob-client)
- [@polymarket/clob-client](https://www.npmjs.com/package/@polymarket/clob-client)
