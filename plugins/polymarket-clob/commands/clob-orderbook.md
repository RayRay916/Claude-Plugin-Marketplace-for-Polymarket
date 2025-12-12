# Polymarket CLOB API - Orderbook Data

## Overview
Access real-time orderbook data including depth, spreads, and liquidity information for any market.

## Endpoints

### Get Orderbook
Retrieve full orderbook with bids and asks.

**Python**:
```python
from py_clob_client.client import ClobClient

client = ClobClient(host="https://clob.polymarket.com", chain_id=137)

# Get orderbook for a token
token_id = "71321045679252212594626385532706912750332728571942532289631379312455583992563"
orderbook = client.get_order_book(token_id)

print(f"Market: {orderbook['market']}")
print(f"Asset ID: {orderbook['asset_id']}")

# Bids (buy orders)
for bid in orderbook['bids']:
    print(f"Bid: Price={bid['price']}, Size={bid['size']}")

# Asks (sell orders)
for ask in orderbook['asks']:
    print(f"Ask: Price={ask['price']}, Size={ask['size']}")
```

**TypeScript**:
```typescript
import { ClobClient } from '@polymarket/clob-client';

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  chainId: 137,
});

const tokenId = '71321045679252212594626385532706912750332728571942532289631379312455583992563';
const orderbook = await client.getOrderBook(tokenId);

console.log('Market:', orderbook.market);
console.log('Asset ID:', orderbook.asset_id);

// Bids (buy orders)
orderbook.bids.forEach(bid => {
  console.log(`Bid: Price=${bid.price}, Size=${bid.size}`);
});

// Asks (sell orders)
orderbook.asks.forEach(ask => {
  console.log(`Ask: Price=${ask.price}, Size=${ask.size}`);
});
```

### Get Orderbook Summary
Get aggregated orderbook statistics.

**Python**:
```python
# Get summary for a market
summary = client.get_order_book_summary(token_id)

print(f"Market: {summary['market']}")
print(f"Bid: {summary['bid']}")
print(f"Ask: {summary['ask']}")
print(f"Spread: {summary['spread']}")
print(f"Last Price: {summary['last']}")
```

**TypeScript**:
```typescript
const summary = await client.getOrderBookSummary(tokenId);

console.log('Market:', summary.market);
console.log('Bid:', summary.bid);
console.log('Ask:', summary.ask);
console.log('Spread:', summary.spread);
console.log('Last Price:', summary.last);
```

### Get Multiple Orderbooks
Fetch orderbooks for multiple tokens at once.

**Python**:
```python
token_ids = [
    "71321045679252212594626385532706912750332728571942532289631379312455583992563",
    "52114319501245915516055106046884209969926127482827954674443846427813813222426"
]

orderbooks = client.get_order_books(token_ids)

for book in orderbooks:
    print(f"Token: {book['asset_id']}")
    print(f"Best Bid: {book['bids'][0] if book['bids'] else 'N/A'}")
    print(f"Best Ask: {book['asks'][0] if book['asks'] else 'N/A'}")
```

**TypeScript**:
```typescript
const tokenIds = [
  '71321045679252212594626385532706912750332728571942532289631379312455583992563',
  '52114319501245915516055106046884209969926127482827954674443846427813813222426'
];

const orderbooks = await client.getOrderBooks(tokenIds);

orderbooks.forEach(book => {
  console.log('Token:', book.asset_id);
  console.log('Best Bid:', book.bids[0] || 'N/A');
  console.log('Best Ask:', book.asks[0] || 'N/A');
});
```

## Response Format

### Orderbook Structure
```json
{
  "market": "0x1234...",
  "asset_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "bids": [
    {
      "price": "0.52",
      "size": "100.50"
    }
  ],
  "asks": [
    {
      "price": "0.54",
      "size": "200.25"
    }
  ],
  "timestamp": 1234567890
}
```

### Summary Structure
```json
{
  "market": "0x1234...",
  "asset_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "bid": "0.52",
  "ask": "0.54",
  "spread": "0.02",
  "last": "0.53",
  "timestamp": 1234567890
}
```

## Advanced Usage

### Calculate Market Depth
```python
def calculate_depth(orderbook, price_levels=5):
    """Calculate cumulative size at top N price levels"""
    bid_depth = sum(float(bid['size']) for bid in orderbook['bids'][:price_levels])
    ask_depth = sum(float(ask['size']) for ask in orderbook['asks'][:price_levels])

    return {
        'bid_depth': bid_depth,
        'ask_depth': ask_depth,
        'total_depth': bid_depth + ask_depth
    }

depth = calculate_depth(orderbook)
print(f"Top 5 levels - Bid: {depth['bid_depth']}, Ask: {depth['ask_depth']}")
```

```typescript
function calculateDepth(orderbook: any, priceLevels: number = 5) {
  const bidDepth = orderbook.bids
    .slice(0, priceLevels)
    .reduce((sum: number, bid: any) => sum + parseFloat(bid.size), 0);

  const askDepth = orderbook.asks
    .slice(0, priceLevels)
    .reduce((sum: number, ask: any) => sum + parseFloat(ask.size), 0);

  return {
    bidDepth,
    askDepth,
    totalDepth: bidDepth + askDepth,
  };
}

const depth = calculateDepth(orderbook);
console.log(`Top 5 levels - Bid: ${depth.bidDepth}, Ask: ${depth.askDepth}`);
```

### Monitor Spread Changes
```python
import time

def monitor_spread(client, token_id, interval=5):
    """Monitor bid-ask spread over time"""
    while True:
        summary = client.get_order_book_summary(token_id)
        spread = float(summary['spread'])
        print(f"Spread: {spread:.4f} ({spread*100:.2f}%)")
        time.sleep(interval)

# Monitor spread every 5 seconds
monitor_spread(client, token_id)
```

```typescript
async function monitorSpread(client: ClobClient, tokenId: string, interval: number = 5000) {
  while (true) {
    const summary = await client.getOrderBookSummary(tokenId);
    const spread = parseFloat(summary.spread);
    console.log(`Spread: ${spread.toFixed(4)} (${(spread * 100).toFixed(2)}%)`);
    await new Promise(resolve => setTimeout(resolve, interval));
  }
}

// Monitor spread every 5 seconds
await monitorSpread(client, tokenId);
```

## Best Practices
1. **Cache orderbook data** - Don't poll excessively
2. **Use WebSocket** for real-time updates (see `/ws-market-channel`)
3. **Handle empty orderbooks** - Check for empty bids/asks arrays
4. **Validate token IDs** - Ensure valid format before querying

## Resources
- [CLOB API Docs](https://docs.polymarket.com/developers/CLOB/orderbook)
- [Orderbook Spec](https://docs.polymarket.com/developers/CLOB/orderbook-spec)
