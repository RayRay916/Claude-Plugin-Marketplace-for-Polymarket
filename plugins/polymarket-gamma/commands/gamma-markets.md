# Polymarket Gamma API - Markets

## Overview
Access detailed market data, filter markets by various criteria, and build market discovery features.

## Get Markets

### All Markets
**Python**:
```python
import requests

GAMMA_BASE_URL = "https://gamma-api.polymarket.com"

def get_markets(limit=100, offset=0):
    """Get markets with pagination"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "limit": limit,
        "offset": offset
    }
    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()

markets = get_markets(limit=50)

for market in markets:
    print(f"Question: {market['question']}")
    print(f"Category: {market['category']}")
    print(f"Volume: ${market['volume']:.2f}")
    print(f"Liquidity: ${market['liquidity']:.2f}")
    print("---")
```

**TypeScript**:
```typescript
const GAMMA_BASE_URL = 'https://gamma-api.polymarket.com';

async function getMarkets(limit: number = 100, offset: number = 0) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    limit: limit.toString(),
    offset: offset.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  if (!response.ok) throw new Error('Failed to fetch markets');

  return response.json();
}

const markets = await getMarkets(50);

markets.forEach((market: any) => {
  console.log('Question:', market.question);
  console.log('Category:', market.category);
  console.log('Volume:', market.volume.toFixed(2));
  console.log('Liquidity:', market.liquidity.toFixed(2));
  console.log('---');
});
```

### Get Market by ID
**Python**:
```python
def get_market(condition_id):
    """Get specific market by condition ID"""
    url = f"{GAMMA_BASE_URL}/markets/{condition_id}"
    response = requests.get(url)
    response.raise_for_status()
    return response.json()

market = get_market("0x1234...")

print(f"Question: {market['question']}")
print(f"Description: {market['description']}")
print(f"Outcomes: {', '.join(market['outcomes'])}")
print(f"Category: {market['category']}")
print(f"Volume: ${market['volume']:.2f}")
print(f"End Date: {market['end_date']}")
```

**TypeScript**:
```typescript
async function getMarket(conditionId: string) {
  const url = `${GAMMA_BASE_URL}/markets/${conditionId}`;

  const response = await fetch(url);
  if (!response.ok) throw new Error('Failed to fetch market');

  return response.json();
}

const market = await getMarket('0x1234...');

console.log('Question:', market.question);
console.log('Description:', market.description);
console.log('Outcomes:', market.outcomes.join(', '));
console.log('Category:', market.category);
console.log('Volume:', market.volume.toFixed(2));
console.log('End Date:', market.end_date);
```

### Get Market by Slug
**Python**:
```python
def get_market_by_slug(slug):
    """Get market by URL-friendly slug"""
    url = f"{GAMMA_BASE_URL}/markets/slug/{slug}"
    response = requests.get(url)
    return response.json()

market = get_market_by_slug("will-bitcoin-reach-100k-2024")
```

**TypeScript**:
```typescript
async function getMarketBySlug(slug: string) {
  const url = `${GAMMA_BASE_URL}/markets/slug/${slug}`;
  const response = await fetch(url);
  return response.json();
}

const market = await getMarketBySlug('will-bitcoin-reach-100k-2024');
```

## Filter Markets

### By Status
**Python**:
```python
def get_active_markets(limit=50):
    """Get active, tradeable markets"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "active": "true",
        "closed": "false",
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

def get_closed_markets(limit=50):
    """Get closed markets awaiting resolution"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "active": "false",
        "closed": "true",
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

active = get_active_markets()
closed = get_closed_markets()

print(f"Active markets: {len(active)}")
print(f"Closed markets: {len(closed)}")
```

**TypeScript**:
```typescript
async function getActiveMarkets(limit: number = 50) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    active: 'true',
    closed: 'false',
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

async function getClosedMarkets(limit: number = 50) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    active: 'false',
    closed: 'true',
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const active = await getActiveMarkets();
const closed = await getClosedMarkets();

console.log('Active markets:', active.length);
console.log('Closed markets:', closed.length);
```

### By Category
**Python**:
```python
def get_markets_by_category(category, limit=50):
    """Get markets in specific category"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "category": category,
        "active": "true",
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

# Get markets by category
politics = get_markets_by_category("Politics")
crypto = get_markets_by_category("Crypto")
sports = get_markets_by_category("Sports")

print(f"Politics markets: {len(politics)}")
print(f"Crypto markets: {len(crypto)}")
print(f"Sports markets: {len(sports)}")
```

**TypeScript**:
```typescript
async function getMarketsByCategory(category: string, limit: number = 50) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    category,
    active: 'true',
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const politics = await getMarketsByCategory('Politics');
const crypto = await getMarketsByCategory('Crypto');
const sports = await getMarketsByCategory('Sports');

console.log('Politics markets:', politics.length);
console.log('Crypto markets:', crypto.length);
console.log('Sports markets:', sports.length);
```

### By Volume
**Python**:
```python
def get_top_volume_markets(limit=20):
    """Get highest volume markets"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "order": "volume",
        "ascending": "false",
        "active": "true",
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

top_markets = get_top_volume_markets(10)

for idx, market in enumerate(top_markets, 1):
    print(f"#{idx}: {market['question']}")
    print(f"   Volume: ${market['volume']:,.2f}")
```

**TypeScript**:
```typescript
async function getTopVolumeMarkets(limit: number = 20) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    order: 'volume',
    ascending: 'false',
    active: 'true',
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const topMarkets = await getTopVolumeMarkets(10);

topMarkets.forEach((market: any, idx: number) => {
  console.log(`#${idx + 1}: ${market.question}`);
  console.log(`   Volume: $${market.volume.toLocaleString()}`);
});
```

### By Tag
**Python**:
```python
def get_markets_by_tag(tag, limit=50):
    """Get markets with specific tag"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "tag": tag,
        "active": "true",
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

# Get markets by tag
election_markets = get_markets_by_tag("election")
tech_markets = get_markets_by_tag("technology")
```

**TypeScript**:
```typescript
async function getMarketsByTag(tag: string, limit: number = 50) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    tag,
    active: 'true',
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const electionMarkets = await getMarketsByTag('election');
const techMarkets = await getMarketsByTag('technology');
```

## Market Details

### Get Market with Prices
**Python**:
```python
def get_market_with_prices(condition_id):
    """Get market with current outcome prices"""
    market = get_market(condition_id)

    # Market includes token IDs and current prices
    outcomes = []
    for idx, outcome in enumerate(market['outcomes']):
        outcomes.append({
            'name': outcome,
            'token_id': market['tokens'][idx]['token_id'],
            'price': market['tokens'][idx]['price'],
            'volume': market['tokens'][idx]['volume']
        })

    return {
        'question': market['question'],
        'outcomes': outcomes,
        'total_volume': market['volume']
    }

market_data = get_market_with_prices("0x1234...")

print(f"Question: {market_data['question']}\n")
for outcome in market_data['outcomes']:
    print(f"{outcome['name']}: ${outcome['price']:.3f} ({outcome['price']*100:.1f}%)")
```

**TypeScript**:
```typescript
async function getMarketWithPrices(conditionId: string) {
  const market = await getMarket(conditionId);

  const outcomes = market.outcomes.map((outcome: string, idx: number) => ({
    name: outcome,
    tokenId: market.tokens[idx].token_id,
    price: market.tokens[idx].price,
    volume: market.tokens[idx].volume,
  }));

  return {
    question: market.question,
    outcomes,
    totalVolume: market.volume,
  };
}

const marketData = await getMarketWithPrices('0x1234...');

console.log(`Question: ${marketData.question}\n`);
marketData.outcomes.forEach((outcome: any) => {
  console.log(`${outcome.name}: $${outcome.price.toFixed(3)} (${(outcome.price * 100).toFixed(1)}%)`);
});
```

### Get Market Tags
**Python**:
```python
def get_market_tags(condition_id):
    """Get tags for a market"""
    market = get_market(condition_id)
    return market.get('tags', [])

tags = get_market_tags("0x1234...")
print(f"Tags: {', '.join(tags)}")
```

**TypeScript**:
```typescript
async function getMarketTags(conditionId: string) {
  const market = await getMarket(conditionId);
  return market.tags || [];
}

const tags = await getMarketTags('0x1234...');
console.log('Tags:', tags.join(', '));
```

## Market Analytics

### Calculate Market Statistics
**Python**:
```python
def analyze_market(condition_id):
    """Get comprehensive market statistics"""
    market = get_market(condition_id)

    # Calculate outcome distribution
    tokens = market.get('tokens', [])
    total_volume = market.get('volume', 0)

    outcome_stats = []
    for token in tokens:
        outcome_stats.append({
            'outcome': token['outcome'],
            'price': token['price'],
            'probability': token['price'] * 100,
            'volume': token.get('volume', 0),
            'volume_pct': (token.get('volume', 0) / total_volume * 100) if total_volume > 0 else 0
        })

    return {
        'question': market['question'],
        'category': market['category'],
        'total_volume': total_volume,
        'liquidity': market.get('liquidity', 0),
        'end_date': market.get('end_date'),
        'outcomes': outcome_stats,
        'most_likely': max(outcome_stats, key=lambda x: x['probability']) if outcome_stats else None
    }

stats = analyze_market("0x1234...")

print(f"Question: {stats['question']}")
print(f"Total Volume: ${stats['total_volume']:,.2f}")
print(f"\nOutcomes:")
for outcome in stats['outcomes']:
    print(f"  {outcome['outcome']}: {outcome['probability']:.1f}% probability")
    print(f"    Volume: ${outcome['volume']:,.2f} ({outcome['volume_pct']:.1f}%)")

if stats['most_likely']:
    print(f"\nMost Likely: {stats['most_likely']['outcome']} ({stats['most_likely']['probability']:.1f}%)")
```

**TypeScript**:
```typescript
async function analyzeMarket(conditionId: string) {
  const market = await getMarket(conditionId);

  const tokens = market.tokens || [];
  const totalVolume = market.volume || 0;

  const outcomeStats = tokens.map((token: any) => ({
    outcome: token.outcome,
    price: token.price,
    probability: token.price * 100,
    volume: token.volume || 0,
    volumePct: totalVolume > 0 ? (token.volume / totalVolume * 100) : 0,
  }));

  const mostLikely = outcomeStats.length > 0
    ? outcomeStats.reduce((max, curr) => curr.probability > max.probability ? curr : max)
    : null;

  return {
    question: market.question,
    category: market.category,
    totalVolume,
    liquidity: market.liquidity || 0,
    endDate: market.end_date,
    outcomes: outcomeStats,
    mostLikely,
  };
}

const stats = await analyzeMarket('0x1234...');

console.log('Question:', stats.question);
console.log('Total Volume:', stats.totalVolume.toLocaleString());
console.log('\nOutcomes:');
stats.outcomes.forEach((outcome: any) => {
  console.log(`  ${outcome.outcome}: ${outcome.probability.toFixed(1)}% probability`);
  console.log(`    Volume: $${outcome.volume.toLocaleString()} (${outcome.volumePct.toFixed(1)}%)`);
});

if (stats.mostLikely) {
  console.log(`\nMost Likely: ${stats.mostLikely.outcome} (${stats.mostLikely.probability.toFixed(1)}%)`);
}
```

## Response Format

### Market Object
```json
{
  "condition_id": "0x1234...",
  "slug": "will-bitcoin-reach-100k-2024",
  "question": "Will Bitcoin reach $100,000 in 2024?",
  "description": "Resolves YES if Bitcoin...",
  "category": "Crypto",
  "outcomes": ["Yes", "No"],
  "tokens": [
    {
      "token_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
      "outcome": "Yes",
      "price": 0.67,
      "volume": 150000.00
    },
    {
      "token_id": "52114319501245915516055106046884209969926127482827954674443846427813813222426",
      "outcome": "No",
      "price": 0.33,
      "volume": 100000.00
    }
  ],
  "volume": 250000.00,
  "liquidity": 50000.00,
  "start_date": "2024-01-01T00:00:00Z",
  "end_date": "2024-12-31T23:59:59Z",
  "resolution_source": "Official price feeds",
  "tags": ["bitcoin", "crypto", "price"],
  "active": true,
  "closed": false,
  "resolved": false,
  "image": "https://..."
}
```

## Best Practices
1. **Use slugs for URLs** - Better for SEO and UX
2. **Cache market data** - Update every 5-10 minutes
3. **Show probabilities** - Display prices as percentages
4. **Filter client-side when possible** - Reduce API calls
5. **Paginate results** - Don't load thousands of markets

## Resources
- [Markets API Docs](https://docs.polymarket.com/developers/gamma/markets)
- [Market Schema](https://docs.polymarket.com/developers/gamma/market-schema)
