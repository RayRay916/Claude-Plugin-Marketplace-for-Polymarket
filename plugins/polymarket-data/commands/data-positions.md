# Polymarket Data API - Positions & Holdings

## Overview
Query user positions, portfolio holdings, and calculate P&L metrics.

## Get User Positions

### All Positions for User
**Python**:
```python
import requests

BASE_URL = "https://data-api.polymarket.com"

def get_user_positions(address):
    """Get all positions for a user address"""
    url = f"{BASE_URL}/positions"
    params = {"user": address}
    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()

address = "0x1234567890abcdef1234567890abcdef12345678"
positions = get_user_positions(address)

for position in positions:
    print(f"Market: {position['market']}")
    print(f"Outcome: {position['outcome']}")
    print(f"Shares: {position['size']}")
    print(f"Avg Cost: ${position['avg_cost']}")
    print(f"Current Value: ${position['value']}")
    print(f"P&L: ${position['pnl']}")
    print("---")
```

**TypeScript**:
```typescript
const BASE_URL = 'https://data-api.polymarket.com';

async function getUserPositions(address: string) {
  const url = `${BASE_URL}/positions`;
  const params = new URLSearchParams({ user: address });

  const response = await fetch(`${url}?${params}`);
  if (!response.ok) throw new Error('Failed to fetch positions');

  return response.json();
}

const address = '0x1234567890abcdef1234567890abcdef12345678';
const positions = await getUserPositions(address);

positions.forEach((position: any) => {
  console.log('Market:', position.market);
  console.log('Outcome:', position.outcome);
  console.log('Shares:', position.size);
  console.log('Avg Cost:', position.avg_cost);
  console.log('Current Value:', position.value);
  console.log('P&L:', position.pnl);
  console.log('---');
});
```

### Filter by Market
**Python**:
```python
def get_market_position(address, market_id):
    """Get position for specific market"""
    url = f"{BASE_URL}/positions"
    params = {
        "user": address,
        "market": market_id
    }
    response = requests.get(url, params=params)
    return response.json()

market_id = "0xmarket123..."
position = get_market_position(address, market_id)
```

**TypeScript**:
```typescript
async function getMarketPosition(address: string, marketId: string) {
  const url = `${BASE_URL}/positions`;
  const params = new URLSearchParams({
    user: address,
    market: marketId,
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const marketId = '0xmarket123...';
const position = await getMarketPosition(address, marketId);
```

### Open Positions Only
**Python**:
```python
def get_open_positions(address):
    """Get only open (non-zero) positions"""
    all_positions = get_user_positions(address)
    return [p for p in all_positions if float(p['size']) > 0]

open_positions = get_open_positions(address)
print(f"Open positions: {len(open_positions)}")
```

**TypeScript**:
```typescript
async function getOpenPositions(address: string) {
  const allPositions = await getUserPositions(address);
  return allPositions.filter((p: any) => parseFloat(p.size) > 0);
}

const openPositions = await getOpenPositions(address);
console.log('Open positions:', openPositions.length);
```

## Portfolio Analytics

### Calculate Total Portfolio Value
**Python**:
```python
def calculate_portfolio_value(positions):
    """Calculate total portfolio value and P&L"""
    total_value = 0
    total_cost = 0
    total_pnl = 0

    for position in positions:
        if float(position['size']) > 0:
            total_value += float(position['value'])
            total_cost += float(position['size']) * float(position['avg_cost'])
            total_pnl += float(position.get('pnl', 0))

    return {
        'total_value': total_value,
        'total_cost': total_cost,
        'total_pnl': total_pnl,
        'return_pct': (total_pnl / total_cost * 100) if total_cost > 0 else 0,
        'position_count': sum(1 for p in positions if float(p['size']) > 0)
    }

portfolio = calculate_portfolio_value(positions)
print(f"Portfolio Value: ${portfolio['total_value']:.2f}")
print(f"Total Cost: ${portfolio['total_cost']:.2f}")
print(f"Total P&L: ${portfolio['total_pnl']:.2f}")
print(f"Return: {portfolio['return_pct']:.2f}%")
print(f"Open Positions: {portfolio['position_count']}")
```

**TypeScript**:
```typescript
function calculatePortfolioValue(positions: any[]) {
  let totalValue = 0;
  let totalCost = 0;
  let totalPnl = 0;

  for (const position of positions) {
    if (parseFloat(position.size) > 0) {
      totalValue += parseFloat(position.value);
      totalCost += parseFloat(position.size) * parseFloat(position.avg_cost);
      totalPnl += parseFloat(position.pnl || '0');
    }
  }

  const positionCount = positions.filter(p => parseFloat(p.size) > 0).length;

  return {
    totalValue,
    totalCost,
    totalPnl,
    returnPct: totalCost > 0 ? (totalPnl / totalCost * 100) : 0,
    positionCount,
  };
}

const portfolio = calculatePortfolioValue(positions);
console.log(`Portfolio Value: $${portfolio.totalValue.toFixed(2)}`);
console.log(`Total Cost: $${portfolio.totalCost.toFixed(2)}`);
console.log(`Total P&L: $${portfolio.totalPnl.toFixed(2)}`);
console.log(`Return: ${portfolio.returnPct.toFixed(2)}%`);
console.log(`Open Positions: ${portfolio.positionCount}`);
```

### Position Concentration
**Python**:
```python
def analyze_concentration(positions):
    """Analyze portfolio concentration risk"""
    total_value = sum(float(p['value']) for p in positions if float(p['size']) > 0)

    position_weights = []
    for position in positions:
        if float(position['size']) > 0:
            weight = float(position['value']) / total_value if total_value > 0 else 0
            position_weights.append({
                'market': position['market'],
                'outcome': position['outcome'],
                'value': float(position['value']),
                'weight': weight * 100
            })

    # Sort by weight descending
    position_weights.sort(key=lambda x: x['weight'], reverse=True)

    # Top 5 positions
    top_5_weight = sum(p['weight'] for p in position_weights[:5])

    return {
        'positions': position_weights,
        'top_position_weight': position_weights[0]['weight'] if position_weights else 0,
        'top_5_weight': top_5_weight,
        'diversification_score': 100 - top_5_weight  # Higher = more diversified
    }

concentration = analyze_concentration(positions)
print(f"Top Position: {concentration['top_position_weight']:.1f}%")
print(f"Top 5 Positions: {concentration['top_5_weight']:.1f}%")
print(f"Diversification Score: {concentration['diversification_score']:.1f}")
```

**TypeScript**:
```typescript
function analyzeConcentration(positions: any[]) {
  const totalValue = positions
    .filter(p => parseFloat(p.size) > 0)
    .reduce((sum, p) => sum + parseFloat(p.value), 0);

  const positionWeights = positions
    .filter(p => parseFloat(p.size) > 0)
    .map(position => {
      const weight = totalValue > 0 ? parseFloat(position.value) / totalValue : 0;
      return {
        market: position.market,
        outcome: position.outcome,
        value: parseFloat(position.value),
        weight: weight * 100,
      };
    })
    .sort((a, b) => b.weight - a.weight);

  const top5Weight = positionWeights
    .slice(0, 5)
    .reduce((sum, p) => sum + p.weight, 0);

  return {
    positions: positionWeights,
    topPositionWeight: positionWeights[0]?.weight || 0,
    top5Weight,
    diversificationScore: 100 - top5Weight,
  };
}

const concentration = analyzeConcentration(positions);
console.log(`Top Position: ${concentration.topPositionWeight.toFixed(1)}%`);
console.log(`Top 5 Positions: ${concentration.top5Weight.toFixed(1)}%`);
console.log(`Diversification Score: ${concentration.diversificationScore.toFixed(1)}`);
```

## Holdings Data

### Get Token Holdings
**Python**:
```python
def get_token_holdings(address):
    """Get raw token holdings (shares per outcome)"""
    url = f"{BASE_URL}/holdings"
    params = {"user": address}
    response = requests.get(url, params=params)
    return response.json()

holdings = get_token_holdings(address)

for holding in holdings:
    print(f"Token ID: {holding['token_id']}")
    print(f"Balance: {holding['balance']}")
```

**TypeScript**:
```typescript
async function getTokenHoldings(address: string) {
  const url = `${BASE_URL}/holdings`;
  const params = new URLSearchParams({ user: address });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const holdings = await getTokenHoldings(address);

holdings.forEach((holding: any) => {
  console.log('Token ID:', holding.token_id);
  console.log('Balance:', holding.balance);
});
```

## Position Tracking

### Monitor Position Changes
**Python**:
```python
import time

def monitor_positions(address, interval=60):
    """Monitor position changes over time"""
    last_positions = {}

    while True:
        current_positions = get_user_positions(address)

        for position in current_positions:
            market = position['market']
            current_size = float(position['size'])

            if market in last_positions:
                last_size = last_positions[market]
                if current_size != last_size:
                    change = current_size - last_size
                    print(f"Position changed in {market}: {change:+.2f} shares")

            last_positions[market] = current_size

        time.sleep(interval)

# Monitor every 60 seconds
monitor_positions(address)
```

**TypeScript**:
```typescript
async function monitorPositions(address: string, interval: number = 60000) {
  const lastPositions = new Map<string, number>();

  while (true) {
    const currentPositions = await getUserPositions(address);

    for (const position of currentPositions) {
      const market = position.market;
      const currentSize = parseFloat(position.size);

      if (lastPositions.has(market)) {
        const lastSize = lastPositions.get(market)!;
        if (currentSize !== lastSize) {
          const change = currentSize - lastSize;
          console.log(`Position changed in ${market}: ${change >= 0 ? '+' : ''}${change.toFixed(2)} shares`);
        }
      }

      lastPositions.set(market, currentSize);
    }

    await new Promise(resolve => setTimeout(resolve, interval));
  }
}

// Monitor every 60 seconds
await monitorPositions(address);
```

## Response Format

### Position Object
```json
{
  "market": "0x1234...",
  "outcome": "Yes",
  "token_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "size": "100.50",
  "avg_cost": "0.52",
  "value": "54.00",
  "pnl": "1.74",
  "cost_basis": "52.26",
  "current_price": "0.537"
}
```

## Best Practices
1. **Filter zero positions** - Focus on active holdings
2. **Calculate metrics locally** - Reduce API calls
3. **Cache position data** - Update periodically
4. **Monitor concentration** - Assess risk exposure
5. **Track changes over time** - Identify trends

## Resources
- [Positions API Docs](https://docs.polymarket.com/developers/data-api/positions)
- [Portfolio Analytics](https://docs.polymarket.com/developers/data-api/analytics)
