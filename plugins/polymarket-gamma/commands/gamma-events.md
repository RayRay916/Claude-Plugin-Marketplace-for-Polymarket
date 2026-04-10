---
description: "Events and series endpoints"
---

# Polymarket Gamma API - Events & Series

## Overview
Events are high-level prediction questions (e.g., "Who will win the 2024 Presidential Election?"). Series are collections of related events (e.g., "Elections 2024").

## Get Events

### All Events
**Python**:
```python
import requests

GAMMA_BASE_URL = "https://gamma-api.polymarket.com"

def get_all_events(limit=50):
    """Get all events"""
    url = f"{GAMMA_BASE_URL}/events"
    params = {"limit": limit}
    response = requests.get(url, params=params)
    response.raise_for_status()
    return response.json()

events = get_all_events()

for event in events:
    print(f"Title: {event['title']}")
    print(f"Category: {event['category']}")
    print(f"Markets: {event['market_count']}")
    print(f"Volume: ${event['volume']:.2f}")
    print("---")
```

**TypeScript**:
```typescript
const GAMMA_BASE_URL = 'https://gamma-api.polymarket.com';

async function getAllEvents(limit: number = 50) {
  const url = `${GAMMA_BASE_URL}/events`;
  const params = new URLSearchParams({ limit: limit.toString() });

  const response = await fetch(`${url}?${params}`);
  if (!response.ok) throw new Error('Failed to fetch events');

  return response.json();
}

const events = await getAllEvents();

events.forEach((event: any) => {
  console.log('Title:', event.title);
  console.log('Category:', event.category);
  console.log('Markets:', event.market_count);
  console.log('Volume:', event.volume.toFixed(2));
  console.log('---');
});
```

### Get Event by ID
**Python**:
```python
def get_event(event_id):
    """Get specific event by ID"""
    url = f"{GAMMA_BASE_URL}/events/{event_id}"
    response = requests.get(url)
    response.raise_for_status()
    return response.json()

event = get_event("0x1234...")

print(f"Title: {event['title']}")
print(f"Description: {event['description']}")
print(f"Category: {event['category']}")
print(f"Start: {event['start_date']}")
print(f"End: {event['end_date']}")
print(f"Markets: {event['market_count']}")
```

**TypeScript**:
```typescript
async function getEvent(eventId: string) {
  const url = `${GAMMA_BASE_URL}/events/${eventId}`;

  const response = await fetch(url);
  if (!response.ok) throw new Error('Failed to fetch event');

  return response.json();
}

const event = await getEvent('0x1234...');

console.log('Title:', event.title);
console.log('Description:', event.description);
console.log('Category:', event.category);
console.log('Start:', event.start_date);
console.log('End:', event.end_date);
console.log('Markets:', event.market_count);
```

### Get Event by Slug
**Python**:
```python
def get_event_by_slug(slug):
    """Get event by URL-friendly slug"""
    url = f"{GAMMA_BASE_URL}/events/slug/{slug}"
    response = requests.get(url)
    return response.json()

# Example: "presidential-election-2024"
event = get_event_by_slug("presidential-election-2024")
```

**TypeScript**:
```typescript
async function getEventBySlug(slug: string) {
  const url = `${GAMMA_BASE_URL}/events/slug/${slug}`;
  const response = await fetch(url);
  return response.json();
}

const event = await getEventBySlug('presidential-election-2024');
```

### Filter Events
**Python**:
```python
def filter_events(category=None, active=True, closed=False):
    """Filter events by criteria"""
    params = {}

    if category:
        params['category'] = category
    if active is not None:
        params['active'] = str(active).lower()
    if closed is not None:
        params['closed'] = str(closed).lower()

    url = f"{GAMMA_BASE_URL}/events"
    response = requests.get(url, params=params)
    return response.json()

# Get active politics events
politics_events = filter_events(category="Politics", active=True)

# Get closed events
closed_events = filter_events(active=False, closed=True)
```

**TypeScript**:
```typescript
async function filterEvents(options: {
  category?: string;
  active?: boolean;
  closed?: boolean;
}) {
  const params = new URLSearchParams();

  if (options.category) params.append('category', options.category);
  if (options.active !== undefined) params.append('active', String(options.active));
  if (options.closed !== undefined) params.append('closed', String(options.closed));

  const url = `${GAMMA_BASE_URL}/events?${params}`;
  const response = await fetch(url);
  return response.json();
}

// Get active politics events
const politicsEvents = await filterEvents({
  category: 'Politics',
  active: true,
});

// Get closed events
const closedEvents = await filterEvents({
  active: false,
  closed: true,
});
```

## Get Markets for Event

### All Markets in Event
**Python**:
```python
def get_event_markets(event_id):
    """Get all markets for an event"""
    url = f"{GAMMA_BASE_URL}/events/{event_id}/markets"
    response = requests.get(url)
    return response.json()

markets = get_event_markets("0x1234...")

for market in markets:
    print(f"Question: {market['question']}")
    print(f"Outcomes: {', '.join(market['outcomes'])}")
    print(f"Volume: ${market['volume']:.2f}")
    print("---")
```

**TypeScript**:
```typescript
async function getEventMarkets(eventId: string) {
  const url = `${GAMMA_BASE_URL}/events/${eventId}/markets`;
  const response = await fetch(url);
  return response.json();
}

const markets = await getEventMarkets('0x1234...');

markets.forEach((market: any) => {
  console.log('Question:', market.question);
  console.log('Outcomes:', market.outcomes.join(', '));
  console.log('Volume:', market.volume.toFixed(2));
  console.log('---');
});
```

## Series

### Get All Series
**Python**:
```python
def get_all_series():
    """Get all series collections"""
    url = f"{GAMMA_BASE_URL}/series"
    response = requests.get(url)
    return response.json()

series = get_all_series()

for s in series:
    print(f"Title: {s['title']}")
    print(f"Events: {s['event_count']}")
    print(f"Description: {s['description']}")
    print("---")
```

**TypeScript**:
```typescript
async function getAllSeries() {
  const url = `${GAMMA_BASE_URL}/series`;
  const response = await fetch(url);
  return response.json();
}

const series = await getAllSeries();

series.forEach((s: any) => {
  console.log('Title:', s.title);
  console.log('Events:', s.event_count);
  console.log('Description:', s.description);
  console.log('---');
});
```

### Get Series by ID
**Python**:
```python
def get_series(series_id):
    """Get specific series"""
    url = f"{GAMMA_BASE_URL}/series/{series_id}"
    response = requests.get(url)
    return response.json()

series = get_series("elections-2024")
print(f"Title: {series['title']}")
print(f"Events: {series['event_count']}")
```

**TypeScript**:
```typescript
async function getSeries(seriesId: string) {
  const url = `${GAMMA_BASE_URL}/series/${seriesId}`;
  const response = await fetch(url);
  return response.json();
}

const series = await getSeries('elections-2024');
console.log('Title:', series.title);
console.log('Events:', series.event_count);
```

### Get Events in Series
**Python**:
```python
def get_series_events(series_id):
    """Get all events in a series"""
    url = f"{GAMMA_BASE_URL}/series/{series_id}/events"
    response = requests.get(url)
    return response.json()

events = get_series_events("elections-2024")

for event in events:
    print(f"- {event['title']}")
```

**TypeScript**:
```typescript
async function getSeriesEvents(seriesId: string) {
  const url = `${GAMMA_BASE_URL}/series/${seriesId}/events`;
  const response = await fetch(url);
  return response.json();
}

const events = await getSeriesEvents('elections-2024');

events.forEach((event: any) => {
  console.log('-', event.title);
});
```

## Event Analytics

### Calculate Event Statistics
**Python**:
```python
def get_event_stats(event_id):
    """Calculate statistics for an event"""
    event = get_event(event_id)
    markets = get_event_markets(event_id)

    total_volume = sum(m.get('volume', 0) for m in markets)
    total_liquidity = sum(m.get('liquidity', 0) for m in markets)
    avg_volume = total_volume / len(markets) if markets else 0

    active_markets = [m for m in markets if m.get('active', False)]

    return {
        'title': event['title'],
        'total_markets': len(markets),
        'active_markets': len(active_markets),
        'total_volume': total_volume,
        'total_liquidity': total_liquidity,
        'avg_market_volume': avg_volume
    }

stats = get_event_stats("0x1234...")

print(f"Event: {stats['title']}")
print(f"Total Markets: {stats['total_markets']}")
print(f"Active Markets: {stats['active_markets']}")
print(f"Total Volume: ${stats['total_volume']:,.2f}")
print(f"Avg Market Volume: ${stats['avg_market_volume']:,.2f}")
```

**TypeScript**:
```typescript
async function getEventStats(eventId: string) {
  const event = await getEvent(eventId);
  const markets = await getEventMarkets(eventId);

  const totalVolume = markets.reduce((sum: number, m: any) => sum + (m.volume || 0), 0);
  const totalLiquidity = markets.reduce((sum: number, m: any) => sum + (m.liquidity || 0), 0);
  const avgVolume = markets.length > 0 ? totalVolume / markets.length : 0;

  const activeMarkets = markets.filter((m: any) => m.active);

  return {
    title: event.title,
    totalMarkets: markets.length,
    activeMarkets: activeMarkets.length,
    totalVolume,
    totalLiquidity,
    avgMarketVolume: avgVolume,
  };
}

const stats = await getEventStats('0x1234...');

console.log('Event:', stats.title);
console.log('Total Markets:', stats.totalMarkets);
console.log('Active Markets:', stats.activeMarkets);
console.log('Total Volume:', stats.totalVolume.toLocaleString());
console.log('Avg Market Volume:', stats.avgMarketVolume.toLocaleString());
```

## Response Formats

### Event Object
```json
{
  "id": "0x1234...",
  "slug": "presidential-election-2024",
  "title": "2024 Presidential Election",
  "description": "Who will win the 2024 US Presidential Election?",
  "category": "Politics",
  "start_date": "2024-01-01T00:00:00Z",
  "end_date": "2024-11-05T00:00:00Z",
  "market_count": 15,
  "volume": 5000000.00,
  "liquidity": 250000.00,
  "image": "https://...",
  "active": true,
  "closed": false
}
```

### Series Object
```json
{
  "id": "elections-2024",
  "title": "Elections 2024",
  "description": "All 2024 election markets",
  "event_count": 25,
  "total_volume": 10000000.00,
  "slug": "elections-2024",
  "created_at": "2024-01-01T00:00:00Z"
}
```

## Building Event Pages

### Event Detail Page
**Python**:
```python
def render_event_page(event_slug):
    """Get all data needed for an event detail page"""
    event = get_event_by_slug(event_slug)
    markets = get_event_markets(event['id'])

    # Sort markets by volume
    markets.sort(key=lambda m: m.get('volume', 0), reverse=True)

    return {
        'event': event,
        'markets': markets,
        'top_markets': markets[:5],
        'total_volume': sum(m.get('volume', 0) for m in markets)
    }

page_data = render_event_page("presidential-election-2024")
```

**TypeScript**:
```typescript
async function renderEventPage(eventSlug: string) {
  const event = await getEventBySlug(eventSlug);
  const markets = await getEventMarkets(event.id);

  // Sort markets by volume
  markets.sort((a: any, b: any) => (b.volume || 0) - (a.volume || 0));

  return {
    event,
    markets,
    topMarkets: markets.slice(0, 5),
    totalVolume: markets.reduce((sum: number, m: any) => sum + (m.volume || 0), 0),
  };
}

const pageData = await renderEventPage('presidential-election-2024');
```

## Best Practices
1. **Use slugs for URLs** - More user-friendly than IDs
2. **Cache event data** - Events don't change frequently
3. **Load markets lazily** - Don't fetch all markets upfront
4. **Show top markets first** - Sort by volume or activity
5. **Filter by status** - Show active markets prominently

## Resources
- [Events API Docs](https://docs.polymarket.com/developers/gamma/events)
- [Series API Docs](https://docs.polymarket.com/developers/gamma/series)
