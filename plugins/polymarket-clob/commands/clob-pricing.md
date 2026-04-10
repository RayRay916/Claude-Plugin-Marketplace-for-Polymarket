---
description: "Pricing and spreads endpoints"
---

# Polymarket CLOB API - Pricing & Spreads

## Overview
Get current and historical pricing data, spreads, and market statistics.

## Get Market Price

### Current Price
**Python**:
```python
from py_clob_client.client import ClobClient

client = ClobClient(host="https://clob.polymarket.com", chain_id=137)

token_id = "71321045679252212594626385532706912750332728571942532289631379312455583992563"

# Get current market price
price = client.get_price(token_id)
print(f"Current Price: {price}")
```

**TypeScript**:
```typescript
import { ClobClient } from '@polymarket/clob-client';

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  chainId: 137,
});

const tokenId = '71321045679252212594626385532706912750332728571942532289631379312455583992563';

const price = await client.getPrice(tokenId);
console.log('Current Price:', price);
```

### Midpoint Price
Get the midpoint between best bid and best ask.

**Python**:
```python
# Get orderbook summary for midpoint
summary = client.get_order_book_summary(token_id)

bid = float(summary['bid'])
ask = float(summary['ask'])
midpoint = (bid + ask) / 2

print(f"Bid: {bid}")
print(f"Ask: {ask}")
print(f"Midpoint: {midpoint}")
print(f"Spread: {ask - bid}")
```

**TypeScript**:
```typescript
const summary = await client.getOrderBookSummary(tokenId);

const bid = parseFloat(summary.bid);
const ask = parseFloat(summary.ask);
const midpoint = (bid + ask) / 2;

console.log('Bid:', bid);
console.log('Ask:', ask);
console.log('Midpoint:', midpoint);
console.log('Spread:', ask - bid);
```

## Get Spread Data

### Current Spread
**Python**:
```python
def get_spread_info(client, token_id):
    """Get detailed spread information"""
    summary = client.get_order_book_summary(token_id)

    bid = float(summary['bid'])
    ask = float(summary['ask'])
    spread = ask - bid
    spread_percentage = (spread / ask) * 100 if ask > 0 else 0

    return {
        'bid': bid,
        'ask': ask,
        'spread': spread,
        'spread_percentage': spread_percentage,
        'midpoint': (bid + ask) / 2
    }

spread_info = get_spread_info(client, token_id)
print(f"Spread: {spread_info['spread']:.4f} ({spread_info['spread_percentage']:.2f}%)")
```

**TypeScript**:
```typescript
async function getSpreadInfo(client: ClobClient, tokenId: string) {
  const summary = await client.getOrderBookSummary(tokenId);

  const bid = parseFloat(summary.bid);
  const ask = parseFloat(summary.ask);
  const spread = ask - bid;
  const spreadPercentage = ask > 0 ? (spread / ask) * 100 : 0;

  return {
    bid,
    ask,
    spread,
    spreadPercentage,
    midpoint: (bid + ask) / 2,
  };
}

const spreadInfo = await getSpreadInfo(client, tokenId);
console.log(`Spread: ${spreadInfo.spread.toFixed(4)} (${spreadInfo.spreadPercentage.toFixed(2)}%)`);
```

## Historical Prices

### Get Last Trade Price
**Python**:
```python
# Get recent trades to find last execution price
trades = client.get_trades(token_id, limit=1)

if trades:
    last_trade = trades[0]
    print(f"Last Trade Price: {last_trade['price']}")
    print(f"Last Trade Size: {last_trade['size']}")
    print(f"Last Trade Time: {last_trade['timestamp']}")
```

**TypeScript**:
```typescript
const trades = await client.getTrades(tokenId, { limit: 1 });

if (trades.length > 0) {
  const lastTrade = trades[0];
  console.log('Last Trade Price:', lastTrade.price);
  console.log('Last Trade Size:', lastTrade.size);
  console.log('Last Trade Time:', lastTrade.timestamp);
}
```

### Price History
**Python**:
```python
from datetime import datetime, timedelta

def get_price_history(client, token_id, hours=24):
    """Get price history from recent trades"""
    # Calculate timestamp for N hours ago
    start_time = int((datetime.now() - timedelta(hours=hours)).timestamp())

    trades = client.get_trades(
        token_id,
        start_time=start_time,
        limit=1000
    )

    prices = [float(trade['price']) for trade in trades]

    return {
        'count': len(prices),
        'min': min(prices) if prices else None,
        'max': max(prices) if prices else None,
        'avg': sum(prices) / len(prices) if prices else None,
        'latest': prices[0] if prices else None
    }

history = get_price_history(client, token_id, hours=24)
print(f"24h Stats: Min={history['min']}, Max={history['max']}, Avg={history['avg']}")
```

**TypeScript**:
```typescript
async function getPriceHistory(
  client: ClobClient,
  tokenId: string,
  hours: number = 24
) {
  const startTime = Math.floor(Date.now() / 1000) - (hours * 3600);

  const trades = await client.getTrades(tokenId, {
    startTime,
    limit: 1000,
  });

  const prices = trades.map(trade => parseFloat(trade.price));

  return {
    count: prices.length,
    min: prices.length > 0 ? Math.min(...prices) : null,
    max: prices.length > 0 ? Math.max(...prices) : null,
    avg: prices.length > 0 ? prices.reduce((a, b) => a + b, 0) / prices.length : null,
    latest: prices.length > 0 ? prices[0] : null,
  };
}

const history = await getPriceHistory(client, tokenId, 24);
console.log(`24h Stats: Min=${history.min}, Max=${history.max}, Avg=${history.avg}`);
```

## Price Analysis

### Calculate Price Impact
**Python**:
```python
def calculate_price_impact(orderbook, size):
    """Calculate price impact for a given order size"""
    cumulative_size = 0
    weighted_price = 0

    for ask in orderbook['asks']:
        ask_price = float(ask['price'])
        ask_size = float(ask['size'])

        if cumulative_size + ask_size >= size:
            remaining = size - cumulative_size
            weighted_price += ask_price * remaining
            cumulative_size = size
            break
        else:
            weighted_price += ask_price * ask_size
            cumulative_size += ask_size

    if cumulative_size < size:
        return None  # Not enough liquidity

    avg_execution_price = weighted_price / size
    best_ask = float(orderbook['asks'][0]['price'])
    impact = ((avg_execution_price - best_ask) / best_ask) * 100

    return {
        'avg_price': avg_execution_price,
        'best_price': best_ask,
        'impact_percentage': impact
    }

orderbook = client.get_order_book(token_id)
impact = calculate_price_impact(orderbook, 1000)
print(f"Price Impact for 1000 shares: {impact['impact_percentage']:.2f}%")
```

**TypeScript**:
```typescript
function calculatePriceImpact(orderbook: any, size: number) {
  let cumulativeSize = 0;
  let weightedPrice = 0;

  for (const ask of orderbook.asks) {
    const askPrice = parseFloat(ask.price);
    const askSize = parseFloat(ask.size);

    if (cumulativeSize + askSize >= size) {
      const remaining = size - cumulativeSize;
      weightedPrice += askPrice * remaining;
      cumulativeSize = size;
      break;
    } else {
      weightedPrice += askPrice * askSize;
      cumulativeSize += askSize;
    }
  }

  if (cumulativeSize < size) {
    return null; // Not enough liquidity
  }

  const avgExecutionPrice = weightedPrice / size;
  const bestAsk = parseFloat(orderbook.asks[0].price);
  const impact = ((avgExecutionPrice - bestAsk) / bestAsk) * 100;

  return {
    avgPrice: avgExecutionPrice,
    bestPrice: bestAsk,
    impactPercentage: impact,
  };
}

const orderbook = await client.getOrderBook(tokenId);
const impact = calculatePriceImpact(orderbook, 1000);
console.log(`Price Impact for 1000 shares: ${impact.impactPercentage.toFixed(2)}%`);
```

### Monitor Price Changes
**Python**:
```python
import time

def monitor_price_changes(client, token_id, threshold=0.01):
    """Alert on price changes exceeding threshold"""
    last_price = None

    while True:
        current_price = float(client.get_price(token_id))

        if last_price:
            change = abs(current_price - last_price)
            change_pct = (change / last_price) * 100

            if change >= threshold:
                print(f"Price Alert! Changed {change_pct:.2f}%: {last_price} -> {current_price}")

        last_price = current_price
        time.sleep(10)

monitor_price_changes(client, token_id, threshold=0.01)
```

**TypeScript**:
```typescript
async function monitorPriceChanges(
  client: ClobClient,
  tokenId: string,
  threshold: number = 0.01
) {
  let lastPrice: number | null = null;

  while (true) {
    const currentPrice = parseFloat(await client.getPrice(tokenId));

    if (lastPrice !== null) {
      const change = Math.abs(currentPrice - lastPrice);
      const changePct = (change / lastPrice) * 100;

      if (change >= threshold) {
        console.log(`Price Alert! Changed ${changePct.toFixed(2)}%: ${lastPrice} -> ${currentPrice}`);
      }
    }

    lastPrice = currentPrice;
    await new Promise(resolve => setTimeout(resolve, 10000));
  }
}

await monitorPriceChanges(client, tokenId, 0.01);
```

## Best Practices
1. **Use midpoint for fair pricing** - Average of bid/ask
2. **Check spread before trading** - Wide spreads = poor liquidity
3. **Calculate price impact** - Especially for large orders
4. **Monitor historical volatility** - Assess risk
5. **Use WebSocket for real-time** - More efficient than polling

## Resources
- [CLOB Pricing Docs](https://docs.polymarket.com/developers/CLOB/pricing)
- [Market Data API](https://docs.polymarket.com/developers/CLOB/market-data)
