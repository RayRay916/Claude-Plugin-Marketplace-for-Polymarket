# Polymarket CLOB API - Trade History

## Overview
Access trade execution data, historical trades, and user trade activity.

## Get Trades

### Get Recent Trades for Market
**Python**:
```python
from py_clob_client.client import ClobClient

client = ClobClient(host="https://clob.polymarket.com", chain_id=137)

token_id = "71321045679252212594626385532706912750332728571942532289631379312455583992563"

# Get recent trades
trades = client.get_trades(token_id, limit=10)

for trade in trades:
    print(f"Trade: {trade['side']} {trade['size']} @ {trade['price']}")
    print(f"  Time: {trade['timestamp']}")
    print(f"  ID: {trade['id']}")
```

**TypeScript**:
```typescript
import { ClobClient } from '@polymarket/clob-client';

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  chainId: 137,
});

const tokenId = '71321045679252212594626385532706912750332728571942532289631379312455583992563';

const trades = await client.getTrades(tokenId, { limit: 10 });

trades.forEach(trade => {
  console.log(`Trade: ${trade.side} ${trade.size} @ ${trade.price}`);
  console.log(`  Time: ${trade.timestamp}`);
  console.log(`  ID: ${trade.id}`);
});
```

### Get Trades with Time Range
**Python**:
```python
from datetime import datetime, timedelta

# Get trades from last 24 hours
start_time = int((datetime.now() - timedelta(hours=24)).timestamp())
end_time = int(datetime.now().timestamp())

trades = client.get_trades(
    token_id,
    start_time=start_time,
    end_time=end_time,
    limit=100
)

print(f"Found {len(trades)} trades in last 24 hours")
```

**TypeScript**:
```typescript
const now = Math.floor(Date.now() / 1000);
const oneDayAgo = now - (24 * 3600);

const trades = await client.getTrades(tokenId, {
  startTime: oneDayAgo,
  endTime: now,
  limit: 100,
});

console.log(`Found ${trades.length} trades in last 24 hours`);
```

### Get User Trades
**Python**:
```python
# Get all trades for authenticated user
user_trades = client.get_user_trades()

for trade in user_trades:
    print(f"{trade['side']} {trade['size']} @ {trade['price']}")
    print(f"  Market: {trade['market']}")
    print(f"  Fee: {trade['fee']}")
```

**TypeScript**:
```typescript
const userTrades = await client.getUserTrades();

userTrades.forEach(trade => {
  console.log(`${trade.side} ${trade.size} @ ${trade.price}`);
  console.log(`  Market: ${trade.market}`);
  console.log(`  Fee: ${trade.fee}`);
});
```

### Get User Trades for Market
**Python**:
```python
# Get user's trades for specific market
market_trades = client.get_user_trades(market=token_id)

print(f"Your trades in this market: {len(market_trades)}")
```

**TypeScript**:
```typescript
const marketTrades = await client.getUserTrades({ market: tokenId });
console.log('Your trades in this market:', marketTrades.length);
```

## Trade Analytics

### Calculate Volume
**Python**:
```python
def calculate_volume(trades):
    """Calculate total trading volume"""
    total_volume = sum(float(trade['size']) for trade in trades)
    total_value = sum(float(trade['size']) * float(trade['price']) for trade in trades)

    return {
        'total_shares': total_volume,
        'total_value': total_value,
        'avg_price': total_value / total_volume if total_volume > 0 else 0
    }

trades = client.get_trades(token_id, limit=1000)
volume = calculate_volume(trades)

print(f"Volume: {volume['total_shares']} shares")
print(f"Value: ${volume['total_value']:.2f}")
print(f"Avg Price: {volume['avg_price']:.4f}")
```

**TypeScript**:
```typescript
function calculateVolume(trades: any[]) {
  const totalVolume = trades.reduce(
    (sum, trade) => sum + parseFloat(trade.size),
    0
  );

  const totalValue = trades.reduce(
    (sum, trade) => sum + parseFloat(trade.size) * parseFloat(trade.price),
    0
  );

  return {
    totalShares: totalVolume,
    totalValue,
    avgPrice: totalVolume > 0 ? totalValue / totalVolume : 0,
  };
}

const trades = await client.getTrades(tokenId, { limit: 1000 });
const volume = calculateVolume(trades);

console.log(`Volume: ${volume.totalShares} shares`);
console.log(`Value: $${volume.totalValue.toFixed(2)}`);
console.log(`Avg Price: ${volume.avgPrice.toFixed(4)}`);
```

### Buy/Sell Pressure
**Python**:
```python
def analyze_trade_pressure(trades):
    """Analyze buy vs sell pressure"""
    buy_volume = sum(
        float(trade['size'])
        for trade in trades
        if trade['side'] == 'BUY'
    )

    sell_volume = sum(
        float(trade['size'])
        for trade in trades
        if trade['side'] == 'SELL'
    )

    total = buy_volume + sell_volume

    return {
        'buy_volume': buy_volume,
        'sell_volume': sell_volume,
        'buy_percentage': (buy_volume / total * 100) if total > 0 else 0,
        'sell_percentage': (sell_volume / total * 100) if total > 0 else 0,
        'net_pressure': buy_volume - sell_volume
    }

pressure = analyze_trade_pressure(trades)
print(f"Buy: {pressure['buy_percentage']:.1f}% | Sell: {pressure['sell_percentage']:.1f}%")
print(f"Net Pressure: {pressure['net_pressure']:+.2f}")
```

**TypeScript**:
```typescript
function analyzeTradePress(trades: any[]) {
  const buyVolume = trades
    .filter(t => t.side === 'BUY')
    .reduce((sum, t) => sum + parseFloat(t.size), 0);

  const sellVolume = trades
    .filter(t => t.side === 'SELL')
    .reduce((sum, t) => sum + parseFloat(t.size), 0);

  const total = buyVolume + sellVolume;

  return {
    buyVolume,
    sellVolume,
    buyPercentage: total > 0 ? (buyVolume / total * 100) : 0,
    sellPercentage: total > 0 ? (sellVolume / total * 100) : 0,
    netPressure: buyVolume - sellVolume,
  };
}

const pressure = analyzeTradePress(trades);
console.log(`Buy: ${pressure.buyPercentage.toFixed(1)}% | Sell: ${pressure.sellPercentage.toFixed(1)}%`);
console.log(`Net Pressure: ${pressure.netPressure >= 0 ? '+' : ''}${pressure.netPressure.toFixed(2)}`);
```

### Calculate VWAP
**Python**:
```python
def calculate_vwap(trades):
    """Calculate Volume Weighted Average Price"""
    total_value = sum(
        float(trade['size']) * float(trade['price'])
        for trade in trades
    )

    total_volume = sum(float(trade['size']) for trade in trades)

    vwap = total_value / total_volume if total_volume > 0 else 0

    return vwap

vwap = calculate_vwap(trades)
print(f"VWAP: {vwap:.4f}")
```

**TypeScript**:
```typescript
function calculateVWAP(trades: any[]): number {
  const totalValue = trades.reduce(
    (sum, t) => sum + parseFloat(t.size) * parseFloat(t.price),
    0
  );

  const totalVolume = trades.reduce(
    (sum, t) => sum + parseFloat(t.size),
    0
  );

  return totalVolume > 0 ? totalValue / totalVolume : 0;
}

const vwap = calculateVWAP(trades);
console.log(`VWAP: ${vwap.toFixed(4)}`);
```

### User P&L Calculation
**Python**:
```python
def calculate_user_pnl(user_trades, current_price):
    """Calculate realized and unrealized P&L"""
    position = 0
    avg_entry = 0
    realized_pnl = 0
    total_fees = 0

    for trade in user_trades:
        size = float(trade['size'])
        price = float(trade['price'])
        fee = float(trade.get('fee', 0))

        total_fees += fee

        if trade['side'] == 'BUY':
            # Update average entry price
            avg_entry = ((avg_entry * position) + (price * size)) / (position + size)
            position += size
        else:  # SELL
            # Realize P&L on sale
            realized_pnl += (price - avg_entry) * size
            position -= size

    # Calculate unrealized P&L on remaining position
    unrealized_pnl = (current_price - avg_entry) * position if position > 0 else 0

    return {
        'position': position,
        'avg_entry': avg_entry,
        'realized_pnl': realized_pnl,
        'unrealized_pnl': unrealized_pnl,
        'total_pnl': realized_pnl + unrealized_pnl,
        'total_fees': total_fees,
        'net_pnl': realized_pnl + unrealized_pnl - total_fees
    }

current_price = float(client.get_price(token_id))
pnl = calculate_user_pnl(user_trades, current_price)

print(f"Position: {pnl['position']} @ {pnl['avg_entry']:.4f}")
print(f"Realized P&L: ${pnl['realized_pnl']:.2f}")
print(f"Unrealized P&L: ${pnl['unrealized_pnl']:.2f}")
print(f"Net P&L: ${pnl['net_pnl']:.2f}")
```

**TypeScript**:
```typescript
function calculateUserPnL(userTrades: any[], currentPrice: number) {
  let position = 0;
  let avgEntry = 0;
  let realizedPnl = 0;
  let totalFees = 0;

  for (const trade of userTrades) {
    const size = parseFloat(trade.size);
    const price = parseFloat(trade.price);
    const fee = parseFloat(trade.fee || '0');

    totalFees += fee;

    if (trade.side === 'BUY') {
      avgEntry = ((avgEntry * position) + (price * size)) / (position + size);
      position += size;
    } else {
      realizedPnl += (price - avgEntry) * size;
      position -= size;
    }
  }

  const unrealizedPnl = position > 0 ? (currentPrice - avgEntry) * position : 0;

  return {
    position,
    avgEntry,
    realizedPnl,
    unrealizedPnl,
    totalPnl: realizedPnl + unrealizedPnl,
    totalFees,
    netPnl: realizedPnl + unrealizedPnl - totalFees,
  };
}

const currentPrice = parseFloat(await client.getPrice(tokenId));
const pnl = calculateUserPnL(userTrades, currentPrice);

console.log(`Position: ${pnl.position} @ ${pnl.avgEntry.toFixed(4)}`);
console.log(`Realized P&L: $${pnl.realizedPnl.toFixed(2)}`);
console.log(`Unrealized P&L: $${pnl.unrealizedPnl.toFixed(2)}`);
console.log(`Net P&L: $${pnl.netPnl.toFixed(2)}`);
```

## Trade Response Format
```json
{
  "id": "0x1234567890abcdef...",
  "market": "0xabcd...",
  "asset_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "side": "BUY",
  "price": "0.52",
  "size": "100.0",
  "fee": "0.50",
  "timestamp": 1234567890,
  "order_id": "0xorder123...",
  "maker_address": "0xmaker...",
  "taker_address": "0xtaker..."
}
```

## Best Practices
1. **Use pagination** - Don't fetch all trades at once
2. **Filter by time range** - Reduce data transfer
3. **Cache historical data** - Trades don't change
4. **Calculate metrics locally** - Reduce API calls
5. **Track fees** - Include in P&L calculations

## Resources
- [Trade History Docs](https://docs.polymarket.com/developers/CLOB/trades)
- [Market Data API](https://docs.polymarket.com/developers/CLOB/market-data)
