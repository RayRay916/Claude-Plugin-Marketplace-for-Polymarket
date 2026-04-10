---
description: "Gamma API overview and structure"
---

# Polymarket Gamma API - Introduction

## Overview
The Gamma API is Polymarket's market discovery and metadata service. It provides comprehensive access to events, markets, series, tags, comments, and search functionality. Use Gamma to discover markets, get market details, and build market exploration features.

**Base URL**: `https://gamma-api.polymarket.com/`

## Key Features
- **Events**: High-level prediction events (e.g., "2024 Presidential Election")
- **Markets**: Individual prediction markets within events
- **Series**: Collections of related events (e.g., "Elections 2024")
- **Search**: Full-text search across markets and events
- **Tags**: Categorical organization of markets
- **Comments**: Community discussion data
- **Sports**: Specialized sports betting markets

## No Authentication Required
Most Gamma API endpoints are **public** and don't require authentication. Perfect for building market browsers and discovery features.

## Quick Start

### Python
```python
import requests

GAMMA_BASE_URL = "https://gamma-api.polymarket.com"

# Get all active markets
def get_active_markets():
    url = f"{GAMMA_BASE_URL}/markets"
    params = {"active": "true", "closed": "false"}
    response = requests.get(url, params=params)
    return response.json()

markets = get_active_markets()

for market in markets[:5]:  # First 5 markets
    print(f"Title: {market['question']}")
    print(f"Category: {market['category']}")
    print(f"Volume: ${market['volume']:.2f}")
    print(f"End Date: {market['end_date']}")
    print("---")
```

### TypeScript
```typescript
const GAMMA_BASE_URL = 'https://gamma-api.polymarket.com';

// Get all active markets
async function getActiveMarkets() {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    active: 'true',
    closed: 'false',
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const markets = await getActiveMarkets();

markets.slice(0, 5).forEach((market: any) => {
  console.log('Title:', market.question);
  console.log('Category:', market.category);
  console.log('Volume:', market.volume.toFixed(2));
  console.log('End Date:', market.end_date);
  console.log('---');
});
```

## Core Concepts

### Events vs Markets
- **Event**: A high-level prediction question (e.g., "Who will win the 2024 election?")
- **Market**: Individual outcomes within an event (e.g., "Donald Trump", "Joe Biden")
- One event can have multiple markets (binary or multi-outcome)

### Market States
- **Active**: Currently tradeable
- **Closed**: Trading ended, awaiting resolution
- **Resolved**: Market settled with final outcome
- **Archived**: Historical market

### Condition IDs
- Unique identifier for a market condition
- Used to link Gamma data with CLOB trading data
- Format: `0x` + 64 hex characters

### Token IDs
- Each outcome in a market has a unique token ID
- Used for trading via CLOB API
- Derived from condition ID and outcome index

## Common Query Patterns

### Filter by Status
```python
# Get only active, tradeable markets
active_markets = requests.get(
    f"{GAMMA_BASE_URL}/markets",
    params={"active": "true", "closed": "false"}
).json()
```

```typescript
const activeMarkets = await fetch(
  `${GAMMA_BASE_URL}/markets?active=true&closed=false`
).then(r => r.json());
```

### Filter by Category
```python
# Get markets in specific category
politics_markets = requests.get(
    f"{GAMMA_BASE_URL}/markets",
    params={"category": "Politics"}
).json()
```

```typescript
const politicsMarkets = await fetch(
  `${GAMMA_BASE_URL}/markets?category=Politics`
).then(r => r.json());
```

### Sort by Volume
```python
# Get highest volume markets
top_markets = requests.get(
    f"{GAMMA_BASE_URL}/markets",
    params={"order": "volume", "ascending": "false", "limit": 10}
).json()
```

```typescript
const topMarkets = await fetch(
  `${GAMMA_BASE_URL}/markets?order=volume&ascending=false&limit=10`
).then(r => r.json());
```

## Market Categories
Common categories include:
- **Politics**: Elections, policy, government
- **Crypto**: Cryptocurrency prices and events
- **Sports**: Sports betting markets
- **Science & Tech**: Technology and scientific outcomes
- **Business**: Corporate and economic events
- **Pop Culture**: Entertainment and media
- **News**: Current events

## Response Pagination
Most endpoints support pagination:
```python
# Get markets with pagination
def get_markets_page(limit=20, offset=0):
    params = {
        "limit": limit,
        "offset": offset
    }
    return requests.get(f"{GAMMA_BASE_URL}/markets", params=params).json()

# First page
page1 = get_markets_page(limit=20, offset=0)
# Second page
page2 = get_markets_page(limit=20, offset=20)
```

```typescript
async function getMarketsPage(limit: number = 20, offset: number = 0) {
  const params = new URLSearchParams({
    limit: limit.toString(),
    offset: offset.toString(),
  });

  return fetch(`${GAMMA_BASE_URL}/markets?${params}`).then(r => r.json());
}

const page1 = await getMarketsPage(20, 0);
const page2 = await getMarketsPage(20, 20);
```

## Available Commands
- `/gamma-intro` - This overview
- `/gamma-events` - Events and series endpoints
- `/gamma-markets` - Market data and filtering
- `/gamma-search` - Search and discovery

## Rate Limits
- **Public Endpoints**: 300 requests/minute
- **No authentication required** for most operations
- Implement caching for frequently accessed data

## Common Use Cases
1. **Market Browser**: Display and filter available markets
2. **Event Pages**: Show all markets for an event
3. **Search Interface**: Let users find markets by keyword
4. **Category Pages**: Group markets by topic
5. **Popular Markets**: Show trending/high-volume markets
6. **Sports Dashboard**: Display sports betting markets

## Best Practices
1. **Cache market data** - Update periodically, not on every request
2. **Filter server-side** - Use query parameters to reduce data transfer
3. **Paginate results** - Don't fetch thousands of markets at once
4. **Handle missing fields** - Not all markets have all metadata
5. **Use slugs for URLs** - Markets have URL-friendly slugs

## Resources
- [Gamma API Docs](https://docs.polymarket.com/developers/gamma)
- [Market Schema](https://docs.polymarket.com/developers/gamma/markets)
- [Events Schema](https://docs.polymarket.com/developers/gamma/events)
