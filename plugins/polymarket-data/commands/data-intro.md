---
description: "Data API overview and authentication"
---

# Polymarket Data API - Introduction

## Overview
The Data API provides access to user-specific data, portfolio information, trading activity, and leaderboard rankings. Unlike the CLOB API which focuses on trading operations, the Data API is designed for analytics, portfolio tracking, and user insights.

**Base URL**: `https://data-api.polymarket.com/`

## Key Features
- **User Positions**: Current holdings and open positions
- **Portfolio Analytics**: P&L, portfolio value, performance metrics
- **Activity Feed**: Trade history, order activity, transaction logs
- **Leaderboards**: Rankings, top traders, competition stats
- **Builder Data**: Advanced analytics for institutional users

## Authentication
Most Data API endpoints are **read-only** and require either:
- **API Key** (for public data)
- **Signed Requests** (for user-specific data)

See `/data-intro` authentication section below.

## Quick Start

### Python
```python
import requests

BASE_URL = "https://data-api.polymarket.com"

# Get user positions (public endpoint)
def get_positions(address):
    url = f"{BASE_URL}/positions"
    params = {"user": address}
    response = requests.get(url, params=params)
    return response.json()

# Example usage
address = "0x1234567890abcdef1234567890abcdef12345678"
positions = get_positions(address)

for position in positions:
    print(f"Market: {position['market']}")
    print(f"Shares: {position['size']}")
    print(f"Value: ${position['value']}")
```

### TypeScript
```typescript
const BASE_URL = 'https://data-api.polymarket.com';

// Get user positions
async function getPositions(address: string) {
  const url = `${BASE_URL}/positions`;
  const params = new URLSearchParams({ user: address });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

// Example usage
const address = '0x1234567890abcdef1234567890abcdef12345678';
const positions = await getPositions(address);

positions.forEach((position: any) => {
  console.log('Market:', position.market);
  console.log('Shares:', position.size);
  console.log('Value:', position.value);
});
```

## Authentication Methods

### 1. Public Access (No Auth)
Some endpoints are public and don't require authentication:
```python
# Python - No auth needed
response = requests.get(f"{BASE_URL}/leaderboard")
```

```typescript
// TypeScript - No auth needed
const response = await fetch(`${BASE_URL}/leaderboard`);
```

### 2. API Key Authentication
For user-specific data:
```python
# Python
headers = {
    "Authorization": f"Bearer {API_KEY}"
}
response = requests.get(f"{BASE_URL}/user/positions", headers=headers)
```

```typescript
// TypeScript
const headers = {
  'Authorization': `Bearer ${API_KEY}`,
};
const response = await fetch(`${BASE_URL}/user/positions`, { headers });
```

### 3. Signed Requests (Advanced)
For sensitive operations, use wallet signatures:
```python
from eth_account import Account
from eth_account.messages import encode_defunct

def create_signed_request(private_key, message):
    account = Account.from_key(private_key)
    message_hash = encode_defunct(text=message)
    signed = account.sign_message(message_hash)

    return {
        'address': account.address,
        'signature': signed.signature.hex()
    }
```

```typescript
import { Wallet } from 'ethers';

async function createSignedRequest(privateKey: string, message: string) {
  const wallet = new Wallet(privateKey);
  const signature = await wallet.signMessage(message);

  return {
    address: wallet.address,
    signature,
  };
}
```

## Key Concepts

### Market IDs vs Token IDs
- **Market ID**: Identifies a prediction market (condition)
- **Token ID**: Identifies a specific outcome within a market
- The Data API primarily uses market IDs for aggregated data

### Position States
- **Open**: Active position with shares held
- **Closed**: Position fully exited (sold or redeemed)
- **Settled**: Market resolved, position can be redeemed

### Value Calculations
Position values are typically denominated in USDC:
- **Market Value**: Current worth at market prices
- **Cost Basis**: Original purchase cost
- **Unrealized P&L**: Market Value - Cost Basis

## Rate Limits
- **Public Endpoints**: 100 requests/minute
- **Authenticated**: 300 requests/minute
- **Builder Tier**: 1000 requests/minute

Implement exponential backoff for 429 (rate limit) responses.

## Available Commands
- `/data-intro` - This overview
- `/data-positions` - Position and holdings data
- `/data-activity` - Activity and trade history
- `/data-leaderboard` - Leaderboard and rankings

## Common Use Cases
1. **Portfolio Tracker**: Monitor user positions and P&L
2. **Activity Feed**: Display recent trading activity
3. **Leaderboard Widget**: Show top traders
4. **Performance Analytics**: Calculate returns and metrics
5. **Risk Monitor**: Track exposure and concentration

## Best Practices
1. **Cache responses** - Reduce unnecessary API calls
2. **Batch requests** - Fetch multiple users/markets together
3. **Handle pagination** - Use limit/offset parameters
4. **Error handling** - Gracefully handle missing data
5. **Respect rate limits** - Implement backoff strategies

## Resources
- [Data API Docs](https://docs.polymarket.com/developers/data-api)
- [API Reference](https://docs.polymarket.com/api/data)
