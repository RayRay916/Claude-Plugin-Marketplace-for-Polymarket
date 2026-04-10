---
description: "Leaderboard and rankings data"
---

# Polymarket Data API - Leaderboards & Rankings

## Overview
Access leaderboard data, trader rankings, competition stats, and top performer metrics.

## Get Leaderboard

### Global Leaderboard
**Python**:
```python
import requests

BASE_URL = "https://data-api.polymarket.com"

def get_leaderboard(limit=100):
    """Get global leaderboard rankings"""
    url = f"{BASE_URL}/leaderboard"
    params = {"limit": limit}
    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()

leaderboard = get_leaderboard(limit=50)

for idx, trader in enumerate(leaderboard, 1):
    print(f"#{idx} {trader['address'][:10]}...")
    print(f"  P&L: ${trader['pnl']:.2f}")
    print(f"  Volume: ${trader['volume']:.2f}")
    print(f"  Trades: {trader['trade_count']}")
    print(f"  ROI: {trader['roi']:.2f}%")
    print("---")
```

**TypeScript**:
```typescript
const BASE_URL = 'https://data-api.polymarket.com';

async function getLeaderboard(limit: number = 100) {
  const url = `${BASE_URL}/leaderboard`;
  const params = new URLSearchParams({ limit: limit.toString() });

  const response = await fetch(`${url}?${params}`);
  if (!response.ok) throw new Error('Failed to fetch leaderboard');

  return response.json();
}

const leaderboard = await getLeaderboard(50);

leaderboard.forEach((trader: any, idx: number) => {
  console.log(`#${idx + 1} ${trader.address.substring(0, 10)}...`);
  console.log(`  P&L: $${trader.pnl.toFixed(2)}`);
  console.log(`  Volume: $${trader.volume.toFixed(2)}`);
  console.log(`  Trades: ${trader.trade_count}`);
  console.log(`  ROI: ${trader.roi.toFixed(2)}%`);
  console.log('---');
});
```

### Time-Based Leaderboard
**Python**:
```python
def get_weekly_leaderboard():
    """Get weekly leaderboard"""
    url = f"{BASE_URL}/leaderboard"
    params = {
        "period": "week",
        "limit": 50
    }
    response = requests.get(url, params=params)
    return response.json()

def get_monthly_leaderboard():
    """Get monthly leaderboard"""
    url = f"{BASE_URL}/leaderboard"
    params = {
        "period": "month",
        "limit": 50
    }
    response = requests.get(url, params=params)
    return response.json()

weekly = get_weekly_leaderboard()
monthly = get_monthly_leaderboard()

print(f"Top Weekly Trader: {weekly[0]['address'][:10]}... (${weekly[0]['pnl']:.2f})")
print(f"Top Monthly Trader: {monthly[0]['address'][:10]}... (${monthly[0]['pnl']:.2f})")
```

**TypeScript**:
```typescript
async function getWeeklyLeaderboard() {
  const url = `${BASE_URL}/leaderboard`;
  const params = new URLSearchParams({
    period: 'week',
    limit: '50',
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

async function getMonthlyLeaderboard() {
  const url = `${BASE_URL}/leaderboard`;
  const params = new URLSearchParams({
    period: 'month',
    limit: '50',
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const weekly = await getWeeklyLeaderboard();
const monthly = await getMonthlyLeaderboard();

console.log(`Top Weekly Trader: ${weekly[0].address.substring(0, 10)}... ($${weekly[0].pnl.toFixed(2)})`);
console.log(`Top Monthly Trader: ${monthly[0].address.substring(0, 10)}... ($${monthly[0].pnl.toFixed(2)})`);
```

## User Rankings

### Get User Rank
**Python**:
```python
def get_user_rank(address):
    """Get rank for specific user"""
    url = f"{BASE_URL}/leaderboard/rank"
    params = {"user": address}
    response = requests.get(url, params=params)
    return response.json()

address = "0x1234567890abcdef1234567890abcdef12345678"
rank_data = get_user_rank(address)

print(f"Global Rank: #{rank_data['rank']}")
print(f"P&L: ${rank_data['pnl']:.2f}")
print(f"Volume: ${rank_data['volume']:.2f}")
print(f"Win Rate: {rank_data['win_rate']:.1f}%")
```

**TypeScript**:
```typescript
async function getUserRank(address: string) {
  const url = `${BASE_URL}/leaderboard/rank`;
  const params = new URLSearchParams({ user: address });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const address = '0x1234567890abcdef1234567890abcdef12345678';
const rankData = await getUserRank(address);

console.log(`Global Rank: #${rankData.rank}`);
console.log(`P&L: $${rankData.pnl.toFixed(2)}`);
console.log(`Volume: $${rankData.volume.toFixed(2)}`);
console.log(`Win Rate: ${rankData.win_rate.toFixed(1)}%`);
```

### Compare Users
**Python**:
```python
def compare_traders(address1, address2):
    """Compare two traders"""
    rank1 = get_user_rank(address1)
    rank2 = get_user_rank(address2)

    comparison = {
        'trader1': {
            'address': address1,
            'rank': rank1['rank'],
            'pnl': rank1['pnl'],
            'volume': rank1['volume'],
            'roi': rank1['roi']
        },
        'trader2': {
            'address': address2,
            'rank': rank2['rank'],
            'pnl': rank2['pnl'],
            'volume': rank2['volume'],
            'roi': rank2['roi']
        },
        'better_rank': address1 if rank1['rank'] < rank2['rank'] else address2,
        'higher_pnl': address1 if rank1['pnl'] > rank2['pnl'] else address2,
        'higher_volume': address1 if rank1['volume'] > rank2['volume'] else address2,
        'higher_roi': address1 if rank1['roi'] > rank2['roi'] else address2
    }

    return comparison

comparison = compare_traders(
    "0x1111111111111111111111111111111111111111",
    "0x2222222222222222222222222222222222222222"
)

print(f"Better Rank: {comparison['better_rank'][:10]}...")
print(f"Higher P&L: {comparison['higher_pnl'][:10]}...")
print(f"Higher Volume: {comparison['higher_volume'][:10]}...")
print(f"Higher ROI: {comparison['higher_roi'][:10]}...")
```

**TypeScript**:
```typescript
async function compareTraders(address1: string, address2: string) {
  const rank1 = await getUserRank(address1);
  const rank2 = await getUserRank(address2);

  return {
    trader1: {
      address: address1,
      rank: rank1.rank,
      pnl: rank1.pnl,
      volume: rank1.volume,
      roi: rank1.roi,
    },
    trader2: {
      address: address2,
      rank: rank2.rank,
      pnl: rank2.pnl,
      volume: rank2.volume,
      roi: rank2.roi,
    },
    betterRank: rank1.rank < rank2.rank ? address1 : address2,
    higherPnl: rank1.pnl > rank2.pnl ? address1 : address2,
    higherVolume: rank1.volume > rank2.volume ? address1 : address2,
    higherRoi: rank1.roi > rank2.roi ? address1 : address2,
  };
}

const comparison = await compareTraders(
  '0x1111111111111111111111111111111111111111',
  '0x2222222222222222222222222222222222222222'
);

console.log(`Better Rank: ${comparison.betterRank.substring(0, 10)}...`);
console.log(`Higher P&L: ${comparison.higherPnl.substring(0, 10)}...`);
console.log(`Higher Volume: ${comparison.higherVolume.substring(0, 10)}...`);
console.log(`Higher ROI: ${comparison.higherRoi.substring(0, 10)}...`);
```

## Leaderboard Analytics

### Top Performers Analysis
**Python**:
```python
def analyze_top_performers(limit=100):
    """Analyze characteristics of top traders"""
    leaderboard = get_leaderboard(limit)

    total_pnl = sum(t['pnl'] for t in leaderboard)
    total_volume = sum(t['volume'] for t in leaderboard)
    avg_trades = sum(t['trade_count'] for t in leaderboard) / len(leaderboard)

    profitable_traders = [t for t in leaderboard if t['pnl'] > 0]
    avg_roi = sum(t['roi'] for t in leaderboard) / len(leaderboard)

    return {
        'total_traders': len(leaderboard),
        'total_pnl': total_pnl,
        'total_volume': total_volume,
        'avg_trades_per_trader': avg_trades,
        'profitable_count': len(profitable_traders),
        'profitable_percentage': len(profitable_traders) / len(leaderboard) * 100,
        'avg_roi': avg_roi,
        'top_pnl': leaderboard[0]['pnl'],
        'top_volume': max(t['volume'] for t in leaderboard)
    }

analysis = analyze_top_performers(100)

print(f"Top 100 Traders Analysis:")
print(f"Total P&L: ${analysis['total_pnl']:,.2f}")
print(f"Total Volume: ${analysis['total_volume']:,.2f}")
print(f"Avg Trades per Trader: {analysis['avg_trades_per_trader']:.0f}")
print(f"Profitable: {analysis['profitable_count']} ({analysis['profitable_percentage']:.1f}%)")
print(f"Avg ROI: {analysis['avg_roi']:.2f}%")
print(f"Top P&L: ${analysis['top_pnl']:,.2f}")
```

**TypeScript**:
```typescript
async function analyzeTopPerformers(limit: number = 100) {
  const leaderboard = await getLeaderboard(limit);

  const totalPnl = leaderboard.reduce((sum: number, t: any) => sum + t.pnl, 0);
  const totalVolume = leaderboard.reduce((sum: number, t: any) => sum + t.volume, 0);
  const avgTrades = leaderboard.reduce((sum: number, t: any) => sum + t.trade_count, 0) / leaderboard.length;

  const profitableTraders = leaderboard.filter((t: any) => t.pnl > 0);
  const avgRoi = leaderboard.reduce((sum: number, t: any) => sum + t.roi, 0) / leaderboard.length;

  return {
    totalTraders: leaderboard.length,
    totalPnl,
    totalVolume,
    avgTradesPerTrader: avgTrades,
    profitableCount: profitableTraders.length,
    profitablePercentage: (profitableTraders.length / leaderboard.length) * 100,
    avgRoi,
    topPnl: leaderboard[0].pnl,
    topVolume: Math.max(...leaderboard.map((t: any) => t.volume)),
  };
}

const analysis = await analyzeTopPerformers(100);

console.log('Top 100 Traders Analysis:');
console.log(`Total P&L: $${analysis.totalPnl.toLocaleString('en-US', {minimumFractionDigits: 2})}`);
console.log(`Total Volume: $${analysis.totalVolume.toLocaleString('en-US', {minimumFractionDigits: 2})}`);
console.log(`Avg Trades per Trader: ${analysis.avgTradesPerTrader.toFixed(0)}`);
console.log(`Profitable: ${analysis.profitableCount} (${analysis.profitablePercentage.toFixed(1)}%)`);
console.log(`Avg ROI: ${analysis.avgRoi.toFixed(2)}%`);
console.log(`Top P&L: $${analysis.topPnl.toLocaleString('en-US', {minimumFractionDigits: 2})}`);
```

## Market-Specific Leaderboards

### Top Traders per Market
**Python**:
```python
def get_market_leaderboard(market_id, limit=50):
    """Get leaderboard for specific market"""
    url = f"{BASE_URL}/leaderboard/market"
    params = {
        "market": market_id,
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

market_id = "0xmarket123..."
market_leaders = get_market_leaderboard(market_id)

print(f"Top traders in market {market_id[:10]}...:")
for idx, trader in enumerate(market_leaders[:10], 1):
    print(f"#{idx} {trader['address'][:10]}... - P&L: ${trader['pnl']:.2f}")
```

**TypeScript**:
```typescript
async function getMarketLeaderboard(marketId: string, limit: number = 50) {
  const url = `${BASE_URL}/leaderboard/market`;
  const params = new URLSearchParams({
    market: marketId,
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const marketId = '0xmarket123...';
const marketLeaders = await getMarketLeaderboard(marketId);

console.log(`Top traders in market ${marketId.substring(0, 10)}...:`);
marketLeaders.slice(0, 10).forEach((trader: any, idx: number) => {
  console.log(`#${idx + 1} ${trader.address.substring(0, 10)}... - P&L: $${trader.pnl.toFixed(2)}`);
});
```

## Response Format

### Leaderboard Entry
```json
{
  "rank": 1,
  "address": "0x1234567890abcdef1234567890abcdef12345678",
  "pnl": 12500.50,
  "volume": 250000.00,
  "trade_count": 1523,
  "roi": 15.75,
  "win_rate": 62.5,
  "avg_trade_size": 164.15,
  "markets_traded": 87,
  "total_fees": 1250.25
}
```

## Leaderboard Widget Example

### Simple Leaderboard Display
**Python**:
```python
def display_leaderboard_widget(limit=10):
    """Display formatted leaderboard"""
    leaderboard = get_leaderboard(limit)

    print("=" * 70)
    print(f"{'RANK':<6} {'ADDRESS':<15} {'P&L':<12} {'VOLUME':<12} {'ROI':<8}")
    print("=" * 70)

    for trader in leaderboard:
        address_short = f"{trader['address'][:6]}...{trader['address'][-4:]}"
        print(
            f"#{trader['rank']:<5} "
            f"{address_short:<15} "
            f"${trader['pnl']:>10,.2f} "
            f"${trader['volume']:>10,.2f} "
            f"{trader['roi']:>6.2f}%"
        )

    print("=" * 70)

display_leaderboard_widget(10)
```

**TypeScript**:
```typescript
async function displayLeaderboardWidget(limit: number = 10) {
  const leaderboard = await getLeaderboard(limit);

  console.log('='.repeat(70));
  console.log('RANK   ADDRESS          P&L          VOLUME       ROI    ');
  console.log('='.repeat(70));

  leaderboard.forEach((trader: any) => {
    const addressShort = `${trader.address.substring(0, 6)}...${trader.address.slice(-4)}`;
    console.log(
      `#${trader.rank.toString().padEnd(5)} ` +
      `${addressShort.padEnd(15)} ` +
      `$${trader.pnl.toFixed(2).padStart(10)} ` +
      `$${trader.volume.toFixed(2).padStart(10)} ` +
      `${trader.roi.toFixed(2).padStart(6)}%`
    );
  });

  console.log('='.repeat(70));
}

await displayLeaderboardWidget(10);
```

## Best Practices
1. **Cache leaderboard data** - Updates infrequently (hourly/daily)
2. **Paginate large lists** - Don't fetch thousands of entries
3. **Filter by timeframe** - Focus on relevant periods
4. **Anonymize addresses** - Truncate for privacy
5. **Track changes** - Monitor rank movements over time

## Resources
- [Leaderboard API Docs](https://docs.polymarket.com/developers/data-api/leaderboard)
- [Rankings System](https://docs.polymarket.com/developers/data-api/rankings)
