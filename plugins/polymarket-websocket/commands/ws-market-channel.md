# Polymarket WebSocket API - Market Channel

## Overview
The market channel provides real-time updates for market data including orderbook changes, trades, and price updates.

**No Authentication Required**: Market data is public.

## Subscribe to Market Channel

### Python
```python
import websocket
import json

WS_URL = "wss://ws-subscriptions-clob.polymarket.com/ws/"

def on_message(ws, message):
    """Handle market updates"""
    data = json.loads(message)

    event_type = data.get('event_type')

    if event_type == 'book':
        # Orderbook update
        print(f"Orderbook Update for {data['asset_id'][:10]}...")
        print(f"  Best Bid: {data['bids'][0] if data['bids'] else 'N/A'}")
        print(f"  Best Ask: {data['asks'][0] if data['asks'] else 'N/A'}")
    elif event_type == 'trade':
        # Trade execution
        print(f"Trade: {data['side']} {data['size']} @ {data['price']}")
    elif event_type == 'price':
        # Price update
        print(f"Price: {data['asset_id'][:10]}... = ${data['price']}")

def on_open(ws):
    """Subscribe to market on connection"""
    token_id = "71321045679252212594626385532706912750332728571942532289631379312455583992563"

    subscribe_msg = {
        "auth": {},
        "type": "subscribe",
        "channel": "market",
        "market": token_id
    }
    ws.send(json.dumps(subscribe_msg))
    print(f"Subscribed to market: {token_id[:10]}...")

ws = websocket.WebSocketApp(
    WS_URL,
    on_open=on_open,
    on_message=on_message
)

ws.run_forever()
```

### TypeScript
```typescript
const WS_URL = 'wss://ws-subscriptions-clob.polymarket.com/ws/';

const ws = new WebSocket(WS_URL);

ws.onopen = () => {
  const tokenId = '71321045679252212594626385532706912750332728571942532289631379312455583992563';

  const subscribeMsg = {
    auth: {},
    type: 'subscribe',
    channel: 'market',
    market: tokenId,
  };

  ws.send(JSON.stringify(subscribeMsg));
  console.log(`Subscribed to market: ${tokenId.substring(0, 10)}...`);
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  const eventType = data.event_type;

  if (eventType === 'book') {
    console.log(`Orderbook Update for ${data.asset_id.substring(0, 10)}...`);
    console.log(`  Best Bid: ${data.bids[0] || 'N/A'}`);
    console.log(`  Best Ask: ${data.asks[0] || 'N/A'}`);
  } else if (eventType === 'trade') {
    console.log(`Trade: ${data.side} ${data.size} @ ${data.price}`);
  } else if (eventType === 'price') {
    console.log(`Price: ${data.asset_id.substring(0, 10)}... = $${data.price}`);
  }
};
```

## Event Types

### Orderbook Events
Real-time orderbook updates (bids and asks).

**Event Format**:
```json
{
  "event_type": "book",
  "asset_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "market": "0x1234...",
  "bids": [
    {"price": "0.52", "size": "100.0"},
    {"price": "0.51", "size": "200.0"}
  ],
  "asks": [
    {"price": "0.54", "size": "150.0"},
    {"price": "0.55", "size": "250.0"}
  ],
  "timestamp": 1234567890
}
```

### Trade Events
Real-time trade execution notifications.

**Event Format**:
```json
{
  "event_type": "trade",
  "asset_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "market": "0x1234...",
  "side": "BUY",
  "price": "0.53",
  "size": "50.0",
  "timestamp": 1234567890,
  "maker_address": "0xabc...",
  "taker_address": "0xdef..."
}
```

### Price Events
Price ticker updates.

**Event Format**:
```json
{
  "event_type": "price",
  "asset_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "price": "0.53",
  "bid": "0.52",
  "ask": "0.54",
  "timestamp": 1234567890
}
```

## Advanced Usage

### Multi-Market Subscription
**Python**:
```python
def subscribe_to_markets(ws, market_ids):
    """Subscribe to multiple markets"""
    for market_id in market_ids:
        subscribe_msg = {
            "auth": {},
            "type": "subscribe",
            "channel": "market",
            "market": market_id
        }
        ws.send(json.dumps(subscribe_msg))
        print(f"Subscribed to: {market_id[:10]}...")

def on_open(ws):
    markets = [
        "71321045679252212594626385532706912750332728571942532289631379312455583992563",
        "52114319501245915516055106046884209969926127482827954674443846427813813222426"
    ]
    subscribe_to_markets(ws, markets)

ws = websocket.WebSocketApp(WS_URL, on_open=on_open, on_message=on_message)
```

**TypeScript**:
```typescript
function subscribeToMarkets(ws: WebSocket, marketIds: string[]) {
  marketIds.forEach(marketId => {
    const subscribeMsg = {
      auth: {},
      type: 'subscribe',
      channel: 'market',
      market: marketId,
    };

    ws.send(JSON.stringify(subscribeMsg));
    console.log(`Subscribed to: ${marketId.substring(0, 10)}...`);
  });
}

ws.onopen = () => {
  const markets = [
    '71321045679252212594626385532706912750332728571942532289631379312455583992563',
    '52114319501245915516055106046884209969926127482827954674443846427813813222426'
  ];

  subscribeToMarkets(ws, markets);
};
```

### Orderbook Manager
**Python**:
```python
class OrderbookManager:
    def __init__(self):
        self.orderbooks = {}

    def update_orderbook(self, data):
        """Update local orderbook state"""
        asset_id = data['asset_id']

        self.orderbooks[asset_id] = {
            'bids': data['bids'],
            'asks': data['asks'],
            'timestamp': data['timestamp']
        }

    def get_spread(self, asset_id):
        """Calculate bid-ask spread"""
        book = self.orderbooks.get(asset_id)
        if not book or not book['bids'] or not book['asks']:
            return None

        best_bid = float(book['bids'][0]['price'])
        best_ask = float(book['asks'][0]['price'])

        return best_ask - best_bid

    def get_midpoint(self, asset_id):
        """Calculate midpoint price"""
        book = self.orderbooks.get(asset_id)
        if not book or not book['bids'] or not book['asks']:
            return None

        best_bid = float(book['bids'][0]['price'])
        best_ask = float(book['asks'][0]['price'])

        return (best_bid + best_ask) / 2

manager = OrderbookManager()

def on_message(ws, message):
    data = json.loads(message)

    if data.get('event_type') == 'book':
        manager.update_orderbook(data)

        asset_id = data['asset_id']
        spread = manager.get_spread(asset_id)
        midpoint = manager.get_midpoint(asset_id)

        print(f"Asset: {asset_id[:10]}...")
        print(f"  Midpoint: {midpoint:.4f}")
        print(f"  Spread: {spread:.4f}")
```

**TypeScript**:
```typescript
class OrderbookManager {
  private orderbooks: Map<string, any> = new Map();

  updateOrderbook(data: any) {
    const assetId = data.asset_id;

    this.orderbooks.set(assetId, {
      bids: data.bids,
      asks: data.asks,
      timestamp: data.timestamp,
    });
  }

  getSpread(assetId: string): number | null {
    const book = this.orderbooks.get(assetId);
    if (!book || !book.bids.length || !book.asks.length) {
      return null;
    }

    const bestBid = parseFloat(book.bids[0].price);
    const bestAsk = parseFloat(book.asks[0].price);

    return bestAsk - bestBid;
  }

  getMidpoint(assetId: string): number | null {
    const book = this.orderbooks.get(assetId);
    if (!book || !book.bids.length || !book.asks.length) {
      return null;
    }

    const bestBid = parseFloat(book.bids[0].price);
    const bestAsk = parseFloat(book.asks[0].price);

    return (bestBid + bestAsk) / 2;
  }
}

const manager = new OrderbookManager();

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);

  if (data.event_type === 'book') {
    manager.updateOrderbook(data);

    const assetId = data.asset_id;
    const spread = manager.getSpread(assetId);
    const midpoint = manager.getMidpoint(assetId);

    console.log(`Asset: ${assetId.substring(0, 10)}...`);
    console.log(`  Midpoint: ${midpoint?.toFixed(4)}`);
    console.log(`  Spread: ${spread?.toFixed(4)}`);
  }
};
```

### Trade Volume Tracker
**Python**:
```python
from collections import defaultdict
from datetime import datetime, timedelta

class VolumeTracker:
    def __init__(self, window_minutes=60):
        self.trades = defaultdict(list)
        self.window = timedelta(minutes=window_minutes)

    def add_trade(self, asset_id, trade_data):
        """Add trade to tracker"""
        self.trades[asset_id].append({
            'size': float(trade_data['size']),
            'price': float(trade_data['price']),
            'timestamp': trade_data['timestamp']
        })

        # Clean old trades
        cutoff = datetime.now().timestamp() - self.window.total_seconds()
        self.trades[asset_id] = [
            t for t in self.trades[asset_id]
            if t['timestamp'] >= cutoff
        ]

    def get_volume(self, asset_id):
        """Get volume in time window"""
        return sum(t['size'] for t in self.trades[asset_id])

    def get_vwap(self, asset_id):
        """Get volume-weighted average price"""
        trades = self.trades[asset_id]
        if not trades:
            return None

        total_value = sum(t['size'] * t['price'] for t in trades)
        total_volume = sum(t['size'] for t in trades)

        return total_value / total_volume if total_volume > 0 else None

volume_tracker = VolumeTracker(window_minutes=60)

def on_message(ws, message):
    data = json.loads(message)

    if data.get('event_type') == 'trade':
        asset_id = data['asset_id']
        volume_tracker.add_trade(asset_id, data)

        volume = volume_tracker.get_volume(asset_id)
        vwap = volume_tracker.get_vwap(asset_id)

        print(f"1h Volume: {volume:.2f}")
        print(f"1h VWAP: {vwap:.4f}")
```

**TypeScript**:
```typescript
class VolumeTracker {
  private trades: Map<string, any[]> = new Map();
  private windowMs: number;

  constructor(windowMinutes: number = 60) {
    this.windowMs = windowMinutes * 60 * 1000;
  }

  addTrade(assetId: string, tradeData: any) {
    if (!this.trades.has(assetId)) {
      this.trades.set(assetId, []);
    }

    const trades = this.trades.get(assetId)!;
    trades.push({
      size: parseFloat(tradeData.size),
      price: parseFloat(tradeData.price),
      timestamp: tradeData.timestamp * 1000,  // Convert to ms
    });

    // Clean old trades
    const cutoff = Date.now() - this.windowMs;
    this.trades.set(
      assetId,
      trades.filter(t => t.timestamp >= cutoff)
    );
  }

  getVolume(assetId: string): number {
    const trades = this.trades.get(assetId) || [];
    return trades.reduce((sum, t) => sum + t.size, 0);
  }

  getVWAP(assetId: string): number | null {
    const trades = this.trades.get(assetId) || [];
    if (trades.length === 0) return null;

    const totalValue = trades.reduce((sum, t) => sum + t.size * t.price, 0);
    const totalVolume = trades.reduce((sum, t) => sum + t.size, 0);

    return totalVolume > 0 ? totalValue / totalVolume : null;
  }
}

const volumeTracker = new VolumeTracker(60);

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);

  if (data.event_type === 'trade') {
    const assetId = data.asset_id;
    volumeTracker.addTrade(assetId, data);

    const volume = volumeTracker.getVolume(assetId);
    const vwap = volumeTracker.getVWAP(assetId);

    console.log(`1h Volume: ${volume.toFixed(2)}`);
    console.log(`1h VWAP: ${vwap?.toFixed(4)}`);
  }
};
```

## Unsubscribe from Market

### Python
```python
def unsubscribe_from_market(ws, market_id):
    """Unsubscribe from market updates"""
    unsubscribe_msg = {
        "type": "unsubscribe",
        "channel": "market",
        "market": market_id
    }
    ws.send(json.dumps(unsubscribe_msg))
    print(f"Unsubscribed from: {market_id[:10]}...")
```

### TypeScript
```typescript
function unsubscribeFromMarket(ws: WebSocket, marketId: string) {
  const unsubscribeMsg = {
    type: 'unsubscribe',
    channel: 'market',
    market: marketId,
  };

  ws.send(JSON.stringify(unsubscribeMsg));
  console.log(`Unsubscribed from: ${marketId.substring(0, 10)}...`);
}
```

## Best Practices
1. **Throttle UI updates** - Don't update UI on every message
2. **Buffer updates** - Process in batches for performance
3. **Maintain local state** - Keep orderbook in memory
4. **Clean old data** - Remove stale data periodically
5. **Handle reconnection** - Resubscribe to markets on reconnect
6. **Validate data** - Check for required fields

## Resources
- [Market Channel Docs](https://docs.polymarket.com/developers/websocket/market)
- [WebSocket Best Practices](https://docs.polymarket.com/developers/websocket/best-practices)
