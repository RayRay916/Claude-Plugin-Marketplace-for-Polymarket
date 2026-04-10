---
description: "Search and discovery endpoints"
---

# Polymarket Gamma API - Search & Discovery

## Overview
Full-text search across markets and events, plus advanced discovery features.

## Search Markets

### Basic Search
**Python**:
```python
import requests

GAMMA_BASE_URL = "https://gamma-api.polymarket.com"

def search_markets(query, limit=20):
    """Search markets by keyword"""
    url = f"{GAMMA_BASE_URL}/search/markets"
    params = {
        "query": query,
        "limit": limit
    }
    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()

# Search for Bitcoin markets
bitcoin_markets = search_markets("bitcoin")

for market in bitcoin_markets:
    print(f"Question: {market['question']}")
    print(f"Category: {market['category']}")
    print(f"Volume: ${market['volume']:.2f}")
    print("---")
```

**TypeScript**:
```typescript
const GAMMA_BASE_URL = 'https://gamma-api.polymarket.com';

async function searchMarkets(query: string, limit: number = 20) {
  const url = `${GAMMA_BASE_URL}/search/markets`;
  const params = new URLSearchParams({
    query,
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  if (!response.ok) throw new Error('Search failed');

  return response.json();
}

// Search for Bitcoin markets
const bitcoinMarkets = await searchMarkets('bitcoin');

bitcoinMarkets.forEach((market: any) => {
  console.log('Question:', market.question);
  console.log('Category:', market.category);
  console.log('Volume:', market.volume.toFixed(2));
  console.log('---');
});
```

### Search with Filters
**Python**:
```python
def search_markets_filtered(query, category=None, active=True, limit=20):
    """Search markets with additional filters"""
    url = f"{GAMMA_BASE_URL}/search/markets"
    params = {
        "query": query,
        "limit": limit
    }

    if category:
        params['category'] = category
    if active is not None:
        params['active'] = str(active).lower()

    response = requests.get(url, params=params)
    return response.json()

# Search for active politics markets about elections
election_markets = search_markets_filtered(
    query="election",
    category="Politics",
    active=True
)

# Search for crypto markets about Ethereum
eth_markets = search_markets_filtered(
    query="ethereum",
    category="Crypto",
    active=True
)
```

**TypeScript**:
```typescript
async function searchMarketsFiltered(
  query: string,
  options: {
    category?: string;
    active?: boolean;
    limit?: number;
  } = {}
) {
  const url = `${GAMMA_BASE_URL}/search/markets`;
  const params = new URLSearchParams({ query });

  if (options.category) params.append('category', options.category);
  if (options.active !== undefined) params.append('active', String(options.active));
  params.append('limit', (options.limit || 20).toString());

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

// Search for active politics markets about elections
const electionMarkets = await searchMarketsFiltered('election', {
  category: 'Politics',
  active: true,
});

// Search for crypto markets about Ethereum
const ethMarkets = await searchMarketsFiltered('ethereum', {
  category: 'Crypto',
  active: true,
});
```

## Search Events

### Search Events by Query
**Python**:
```python
def search_events(query, limit=20):
    """Search events by keyword"""
    url = f"{GAMMA_BASE_URL}/search/events"
    params = {
        "query": query,
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

# Search for presidential events
pres_events = search_events("presidential")

for event in pres_events:
    print(f"Title: {event['title']}")
    print(f"Markets: {event['market_count']}")
    print(f"Volume: ${event['volume']:.2f}")
    print("---")
```

**TypeScript**:
```typescript
async function searchEvents(query: string, limit: number = 20) {
  const url = `${GAMMA_BASE_URL}/search/events`;
  const params = new URLSearchParams({
    query,
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const presEvents = await searchEvents('presidential');

presEvents.forEach((event: any) => {
  console.log('Title:', event.title);
  console.log('Markets:', event.market_count);
  console.log('Volume:', event.volume.toFixed(2));
  console.log('---');
});
```

## Advanced Search

### Multi-Keyword Search
**Python**:
```python
def search_multiple_keywords(keywords, combine_results=True):
    """Search for multiple keywords"""
    all_results = []

    for keyword in keywords:
        results = search_markets(keyword, limit=50)
        all_results.extend(results)

    if combine_results:
        # Deduplicate by condition_id
        seen = set()
        unique_results = []
        for market in all_results:
            if market['condition_id'] not in seen:
                seen.add(market['condition_id'])
                unique_results.append(market)
        return unique_results

    return all_results

# Search for multiple crypto keywords
crypto_keywords = ["bitcoin", "ethereum", "crypto"]
crypto_markets = search_multiple_keywords(crypto_keywords)

print(f"Found {len(crypto_markets)} unique crypto markets")
```

**TypeScript**:
```typescript
async function searchMultipleKeywords(
  keywords: string[],
  combineResults: boolean = true
) {
  const allResults: any[] = [];

  for (const keyword of keywords) {
    const results = await searchMarkets(keyword, 50);
    allResults.push(...results);
  }

  if (combineResults) {
    // Deduplicate by condition_id
    const seen = new Set();
    return allResults.filter(market => {
      if (seen.has(market.condition_id)) {
        return false;
      }
      seen.add(market.condition_id);
      return true;
    });
  }

  return allResults;
}

const cryptoKeywords = ['bitcoin', 'ethereum', 'crypto'];
const cryptoMarkets = await searchMultipleKeywords(cryptoKeywords);

console.log(`Found ${cryptoMarkets.length} unique crypto markets`);
```

### Search with Ranking
**Python**:
```python
def search_and_rank(query, rank_by='volume', limit=20):
    """Search markets and sort by custom ranking"""
    results = search_markets(query, limit=100)

    # Sort by specified field
    if rank_by == 'volume':
        results.sort(key=lambda m: m.get('volume', 0), reverse=True)
    elif rank_by == 'liquidity':
        results.sort(key=lambda m: m.get('liquidity', 0), reverse=True)
    elif rank_by == 'end_date':
        results.sort(key=lambda m: m.get('end_date', ''))

    return results[:limit]

# Search and rank by volume
top_btc_markets = search_and_rank("bitcoin", rank_by='volume', limit=10)

for idx, market in enumerate(top_btc_markets, 1):
    print(f"#{idx}: {market['question']}")
    print(f"   Volume: ${market['volume']:,.2f}")
```

**TypeScript**:
```typescript
async function searchAndRank(
  query: string,
  rankBy: 'volume' | 'liquidity' | 'end_date' = 'volume',
  limit: number = 20
) {
  const results = await searchMarkets(query, 100);

  // Sort by specified field
  results.sort((a: any, b: any) => {
    if (rankBy === 'volume' || rankBy === 'liquidity') {
      return (b[rankBy] || 0) - (a[rankBy] || 0);
    } else if (rankBy === 'end_date') {
      return (a.end_date || '').localeCompare(b.end_date || '');
    }
    return 0;
  });

  return results.slice(0, limit);
}

const topBtcMarkets = await searchAndRank('bitcoin', 'volume', 10);

topBtcMarkets.forEach((market: any, idx: number) => {
  console.log(`#${idx + 1}: ${market.question}`);
  console.log(`   Volume: $${market.volume.toLocaleString()}`);
});
```

## Get Tags

### All Available Tags
**Python**:
```python
def get_all_tags():
    """Get all available market tags"""
    url = f"{GAMMA_BASE_URL}/tags"
    response = requests.get(url)
    return response.json()

tags = get_all_tags()

print("Popular Tags:")
for tag in tags[:20]:  # Top 20 tags
    print(f"- {tag['name']} ({tag['market_count']} markets)")
```

**TypeScript**:
```typescript
async function getAllTags() {
  const url = `${GAMMA_BASE_URL}/tags`;
  const response = await fetch(url);
  return response.json();
}

const tags = await getAllTags();

console.log('Popular Tags:');
tags.slice(0, 20).forEach((tag: any) => {
  console.log(`- ${tag.name} (${tag.market_count} markets)`);
});
```

### Get Popular Tags
**Python**:
```python
def get_popular_tags(min_markets=5):
    """Get tags with minimum number of markets"""
    all_tags = get_all_tags()
    return [
        tag for tag in all_tags
        if tag.get('market_count', 0) >= min_markets
    ]

popular_tags = get_popular_tags(min_markets=10)
print(f"Found {len(popular_tags)} popular tags")
```

**TypeScript**:
```typescript
async function getPopularTags(minMarkets: number = 5) {
  const allTags = await getAllTags();
  return allTags.filter((tag: any) => (tag.market_count || 0) >= minMarkets);
}

const popularTags = await getPopularTags(10);
console.log(`Found ${popularTags.length} popular tags`);
```

## Discovery Features

### Trending Markets
**Python**:
```python
def get_trending_markets(limit=20):
    """Get markets with recent volume spike"""
    url = f"{GAMMA_BASE_URL}/markets/trending"
    params = {"limit": limit}
    response = requests.get(url, params=params)
    return response.json()

trending = get_trending_markets(10)

for market in trending:
    print(f"📈 {market['question']}")
    print(f"   24h Volume: ${market['volume_24h']:.2f}")
```

**TypeScript**:
```typescript
async function getTrendingMarkets(limit: number = 20) {
  const url = `${GAMMA_BASE_URL}/markets/trending`;
  const params = new URLSearchParams({ limit: limit.toString() });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const trending = await getTrendingMarkets(10);

trending.forEach((market: any) => {
  console.log(`📈 ${market.question}`);
  console.log(`   24h Volume: $${market.volume_24h.toFixed(2)}`);
});
```

### New Markets
**Python**:
```python
def get_new_markets(limit=20):
    """Get recently created markets"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "order": "created_at",
        "ascending": "false",
        "limit": limit
    }
    response = requests.get(url, params=params)
    return response.json()

new_markets = get_new_markets(10)

for market in new_markets:
    print(f"🆕 {market['question']}")
    print(f"   Created: {market['created_at']}")
```

**TypeScript**:
```typescript
async function getNewMarkets(limit: number = 20) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    order: 'created_at',
    ascending: 'false',
    limit: limit.toString(),
  });

  const response = await fetch(`${url}?${params}`);
  return response.json();
}

const newMarkets = await getNewMarkets(10);

newMarkets.forEach((market: any) => {
  console.log(`🆕 ${market.question}`);
  console.log(`   Created: ${market.created_at}`);
});
```

### Ending Soon
**Python**:
```python
from datetime import datetime, timedelta

def get_ending_soon(days=7, limit=20):
    """Get markets ending within N days"""
    url = f"{GAMMA_BASE_URL}/markets"
    params = {
        "active": "true",
        "order": "end_date",
        "ascending": "true",
        "limit": 100
    }
    response = requests.get(url, params=params)
    markets = response.json()

    # Filter to markets ending within days
    cutoff = datetime.now() + timedelta(days=days)
    ending_soon = []

    for market in markets:
        end_date = datetime.fromisoformat(market['end_date'].replace('Z', '+00:00'))
        if end_date <= cutoff:
            ending_soon.append(market)
        if len(ending_soon) >= limit:
            break

    return ending_soon

ending = get_ending_soon(days=7)

for market in ending:
    print(f"⏰ {market['question']}")
    print(f"   Ends: {market['end_date']}")
```

**TypeScript**:
```typescript
async function getEndingSoon(days: number = 7, limit: number = 20) {
  const url = `${GAMMA_BASE_URL}/markets`;
  const params = new URLSearchParams({
    active: 'true',
    order: 'end_date',
    ascending: 'true',
    limit: '100',
  });

  const response = await fetch(`${url}?${params}`);
  const markets = await response.json();

  const cutoff = new Date();
  cutoff.setDate(cutoff.getDate() + days);

  const endingSoon = [];
  for (const market of markets) {
    const endDate = new Date(market.end_date);
    if (endDate <= cutoff) {
      endingSoon.push(market);
    }
    if (endingSoon.length >= limit) {
      break;
    }
  }

  return endingSoon;
}

const ending = await getEndingSoon(7);

ending.forEach((market: any) => {
  console.log(`⏰ ${market.question}`);
  console.log(`   Ends: ${market.end_date}`);
});
```

## Building Search UI

### Search Autocomplete
**Python**:
```python
def get_search_suggestions(partial_query, limit=10):
    """Get search suggestions for autocomplete"""
    if len(partial_query) < 2:
        return []

    results = search_markets(partial_query, limit=limit)
    return [market['question'] for market in results]

suggestions = get_search_suggestions("bit")
print("Suggestions:", suggestions)
```

**TypeScript**:
```typescript
async function getSearchSuggestions(partialQuery: string, limit: number = 10): Promise<string[]> {
  if (partialQuery.length < 2) {
    return [];
  }

  const results = await searchMarkets(partialQuery, limit);
  return results.map((market: any) => market.question);
}

const suggestions = await getSearchSuggestions('bit');
console.log('Suggestions:', suggestions);
```

## Best Practices
1. **Debounce search input** - Wait for user to stop typing
2. **Show suggestions** - Implement autocomplete for better UX
3. **Cache popular searches** - Reduce API load
4. **Rank results** - Sort by relevance or volume
5. **Filter options** - Let users refine search by category/status

## Resources
- [Search API Docs](https://docs.polymarket.com/developers/gamma/search)
- [Tags API Docs](https://docs.polymarket.com/developers/gamma/tags)
