# Polymarket Data API - Activity & Trade History

## Overview
Access user trading activity, transaction history, and order events.

## Get User Activity

### Recent Activity
**Python**:
```python
import requests

BASE_URL = "https://data-api.polymarket.com"

def get_user_activity(address, limit=50):
    """Get recent activity for a user"""
    url = f"{BASE_URL}/activity"
    params = {
        "user": address,
        "limit": limit
    }
    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()

address = "0x1234567890abcdef1234567890abcdef12345678"
activity = get_user_activity(address)

for event in activity:
    print(f"Type: {event['type']}")
    print(f"Market: {event['market']}")
    print(f"Time: {event['timestamp']}")
    print(f"Details: {event['details']}")
    print("---")
```

**TypeScript**:
```typescript
const BASE_URL = 'https://data-api.polymarket.com';

async function getUserActivity(address: string, limit: number = 50) {
  const url = `${BASE_URL}/activity`;
  const params = new URLSearchParams({
    user: address,
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  if (!response.ok) throw new Error('Failed to fetch activity');

  return response.json();
}

const address = '0x1234567890abcdef1234567890abcdef12345678';
const activity = await getUserActivity(address);

activity.forEach((event: any) => {
  console.log('Type:', event.type);
  console.log('Market:', event.market);
  console.log('Time:', event.timestamp);
  console.log('Details:', event.details);
  console.log('---');
});
```

### Filter by Activity Type
**Python**:
```python
def get_trades_only(address):
    """Get only trade events"""
    activity = get_user_activity(address, limit=100)
    return [event for event in activity if event['type'] == 'trade']

def get_orders_only(address):
    """Get only order events"""
    activity = get_user_activity(address, limit=100)
    return [event for event in activity if event['type'] in ['order_created', 'order_cancelled']]

trades = get_trades_only(address)
orders = get_orders_only(address)

print(f"Trades: {len(trades)}")
print(f"Orders: {len(orders)}")
```

**TypeScript**:
```typescript
async function getTradesOnly(address: string) {
  const activity = await getUserActivity(address, 100);
  return activity.filter((event: any) => event.type === 'trade');
}

async function getOrdersOnly(address: string) {
  const activity = await getUserActivity(address, 100);
  return activity.filter((event: any) =>
    ['order_created', 'order_cancelled'].includes(event.type)
  );
}

const trades = await getTradesOnly(address);
const orders = await getOrdersOnly(address);

console.log('Trades:', trades.length);
console.log('Orders:', orders.length);
```

## Trade History

### Get Trade Details
**Python**:
```python
def get_trade_history(address, market_id=None):
    """Get detailed trade history"""
    activity = get_user_activity(address, limit=100)
    trades = [e for e in activity if e['type'] == 'trade']

    if market_id:
        trades = [t for t in trades if t['market'] == market_id]

    trade_details = []
    for trade in trades:
        trade_details.append({
            'market': trade['market'],
            'side': trade['side'],
            'price': float(trade['price']),
            'size': float(trade['size']),
            'value': float(trade['price']) * float(trade['size']),
            'fee': float(trade.get('fee', 0)),
            'timestamp': trade['timestamp']
        })

    return trade_details

trades = get_trade_history(address)

for trade in trades[:10]:  # Last 10 trades
    print(f"{trade['side']} {trade['size']} @ ${trade['price']}")
    print(f"Value: ${trade['value']:.2f}, Fee: ${trade['fee']:.2f}")
    print(f"Time: {trade['timestamp']}")
    print("---")
```

**TypeScript**:
```typescript
async function getTradeHistory(address: string, marketId?: string) {
  const activity = await getUserActivity(address, 100);
  let trades = activity.filter((e: any) => e.type === 'trade');

  if (marketId) {
    trades = trades.filter((t: any) => t.market === marketId);
  }

  return trades.map((trade: any) => ({
    market: trade.market,
    side: trade.side,
    price: parseFloat(trade.price),
    size: parseFloat(trade.size),
    value: parseFloat(trade.price) * parseFloat(trade.size),
    fee: parseFloat(trade.fee || '0'),
    timestamp: trade.timestamp,
  }));
}

const trades = await getTradeHistory(address);

trades.slice(0, 10).forEach((trade: any) => {
  console.log(`${trade.side} ${trade.size} @ $${trade.price}`);
  console.log(`Value: $${trade.value.toFixed(2)}, Fee: $${trade.fee.toFixed(2)}`);
  console.log(`Time: ${trade.timestamp}`);
  console.log('---');
});
```

### Calculate Trading Metrics
**Python**:
```python
def calculate_trading_metrics(trades):
    """Calculate trading performance metrics"""
    if not trades:
        return None

    total_volume = sum(t['value'] for t in trades)
    total_fees = sum(t['fee'] for t in trades)
    avg_trade_size = sum(t['value'] for t in trades) / len(trades)

    buy_trades = [t for t in trades if t['side'] == 'BUY']
    sell_trades = [t for t in trades if t['side'] == 'SELL']

    avg_buy_price = sum(t['price'] for t in buy_trades) / len(buy_trades) if buy_trades else 0
    avg_sell_price = sum(t['price'] for t in sell_trades) / len(sell_trades) if sell_trades else 0

    return {
        'total_trades': len(trades),
        'total_volume': total_volume,
        'total_fees': total_fees,
        'avg_trade_size': avg_trade_size,
        'buy_count': len(buy_trades),
        'sell_count': len(sell_trades),
        'avg_buy_price': avg_buy_price,
        'avg_sell_price': avg_sell_price,
        'gross_spread': avg_sell_price - avg_buy_price if avg_sell_price and avg_buy_price else 0
    }

metrics = calculate_trading_metrics(trades)

print(f"Total Trades: {metrics['total_trades']}")
print(f"Total Volume: ${metrics['total_volume']:.2f}")
print(f"Total Fees: ${metrics['total_fees']:.2f}")
print(f"Avg Trade Size: ${metrics['avg_trade_size']:.2f}")
print(f"Buy/Sell Ratio: {metrics['buy_count']}/{metrics['sell_count']}")
print(f"Avg Buy Price: ${metrics['avg_buy_price']:.4f}")
print(f"Avg Sell Price: ${metrics['avg_sell_price']:.4f}")
print(f"Gross Spread: ${metrics['gross_spread']:.4f}")
```

**TypeScript**:
```typescript
function calculateTradingMetrics(trades: any[]) {
  if (trades.length === 0) return null;

  const totalVolume = trades.reduce((sum, t) => sum + t.value, 0);
  const totalFees = trades.reduce((sum, t) => sum + t.fee, 0);
  const avgTradeSize = totalVolume / trades.length;

  const buyTrades = trades.filter(t => t.side === 'BUY');
  const sellTrades = trades.filter(t => t.side === 'SELL');

  const avgBuyPrice = buyTrades.length > 0
    ? buyTrades.reduce((sum, t) => sum + t.price, 0) / buyTrades.length
    : 0;

  const avgSellPrice = sellTrades.length > 0
    ? sellTrades.reduce((sum, t) => sum + t.price, 0) / sellTrades.length
    : 0;

  return {
    totalTrades: trades.length,
    totalVolume,
    totalFees,
    avgTradeSize,
    buyCount: buyTrades.length,
    sellCount: sellTrades.length,
    avgBuyPrice,
    avgSellPrice,
    grossSpread: avgSellPrice && avgBuyPrice ? avgSellPrice - avgBuyPrice : 0,
  };
}

const metrics = calculateTradingMetrics(trades);

console.log('Total Trades:', metrics.totalTrades);
console.log('Total Volume:', metrics.totalVolume.toFixed(2));
console.log('Total Fees:', metrics.totalFees.toFixed(2));
console.log('Avg Trade Size:', metrics.avgTradeSize.toFixed(2));
console.log(`Buy/Sell Ratio: ${metrics.buyCount}/${metrics.sellCount}`);
console.log('Avg Buy Price:', metrics.avgBuyPrice.toFixed(4));
console.log('Avg Sell Price:', metrics.avgSellPrice.toFixed(4));
console.log('Gross Spread:', metrics.grossSpread.toFixed(4));
```

## Activity Feed

### Real-time Activity Monitor
**Python**:
```python
import time

def monitor_user_activity(address, interval=30):
    """Monitor user activity in real-time"""
    last_timestamp = None

    while True:
        activity = get_user_activity(address, limit=10)

        # Filter new events
        if last_timestamp:
            new_events = [e for e in activity if e['timestamp'] > last_timestamp]

            for event in new_events:
                print(f"NEW: {event['type']} - {event['market']}")
                print(f"Details: {event['details']}")
                print("---")

        if activity:
            last_timestamp = max(e['timestamp'] for e in activity)

        time.sleep(interval)

# Monitor every 30 seconds
monitor_user_activity(address)
```

**TypeScript**:
```typescript
async function monitorUserActivity(address: string, interval: number = 30000) {
  let lastTimestamp: number | null = null;

  while (true) {
    const activity = await getUserActivity(address, 10);

    if (lastTimestamp) {
      const newEvents = activity.filter((e: any) => e.timestamp > lastTimestamp!);

      newEvents.forEach((event: any) => {
        console.log(`NEW: ${event.type} - ${event.market}`);
        console.log('Details:', event.details);
        console.log('---');
      });
    }

    if (activity.length > 0) {
      lastTimestamp = Math.max(...activity.map((e: any) => e.timestamp));
    }

    await new Promise(resolve => setTimeout(resolve, interval));
  }
}

// Monitor every 30 seconds
await monitorUserActivity(address);
```

## Time-Based Analysis

### Activity by Time Period
**Python**:
```python
from datetime import datetime, timedelta

def get_activity_by_period(address, days=7):
    """Get activity statistics for a time period"""
    activity = get_user_activity(address, limit=1000)

    cutoff_time = int((datetime.now() - timedelta(days=days)).timestamp())
    recent_activity = [e for e in activity if e['timestamp'] >= cutoff_time]

    trades = [e for e in recent_activity if e['type'] == 'trade']
    orders = [e for e in recent_activity if e['type'] in ['order_created', 'order_cancelled']]

    return {
        'period_days': days,
        'total_events': len(recent_activity),
        'trades': len(trades),
        'orders': len(orders),
        'avg_per_day': len(recent_activity) / days
    }

weekly_stats = get_activity_by_period(address, days=7)
print(f"Last 7 Days:")
print(f"Total Events: {weekly_stats['total_events']}")
print(f"Trades: {weekly_stats['trades']}")
print(f"Orders: {weekly_stats['orders']}")
print(f"Avg per Day: {weekly_stats['avg_per_day']:.1f}")
```

**TypeScript**:
```typescript
async function getActivityByPeriod(address: string, days: number = 7) {
  const activity = await getUserActivity(address, 1000);

  const cutoffTime = Math.floor(Date.now() / 1000) - (days * 86400);
  const recentActivity = activity.filter((e: any) => e.timestamp >= cutoffTime);

  const trades = recentActivity.filter((e: any) => e.type === 'trade');
  const orders = recentActivity.filter((e: any) =>
    ['order_created', 'order_cancelled'].includes(e.type)
  );

  return {
    periodDays: days,
    totalEvents: recentActivity.length,
    trades: trades.length,
    orders: orders.length,
    avgPerDay: recentActivity.length / days,
  };
}

const weeklyStats = await getActivityByPeriod(address, 7);
console.log('Last 7 Days:');
console.log('Total Events:', weeklyStats.totalEvents);
console.log('Trades:', weeklyStats.trades);
console.log('Orders:', weeklyStats.orders);
console.log('Avg per Day:', weeklyStats.avgPerDay.toFixed(1));
```

## Response Format

### Activity Event
```json
{
  "type": "trade",
  "market": "0x1234...",
  "outcome": "Yes",
  "side": "BUY",
  "price": "0.52",
  "size": "100.0",
  "fee": "0.50",
  "timestamp": 1234567890,
  "tx_hash": "0xtx123...",
  "details": {
    "order_id": "0xorder...",
    "maker": "0xmaker...",
    "taker": "0xtaker..."
  }
}
```

## Best Practices
1. **Limit query size** - Use pagination for large histories
2. **Filter by type** - Focus on relevant events
3. **Cache historical data** - Activity doesn't change
4. **Aggregate metrics** - Calculate locally when possible
5. **Monitor in batches** - Don't poll too frequently

## Resources
- [Activity API Docs](https://docs.polymarket.com/developers/data-api/activity)
- [Trade History](https://docs.polymarket.com/developers/data-api/trades)
