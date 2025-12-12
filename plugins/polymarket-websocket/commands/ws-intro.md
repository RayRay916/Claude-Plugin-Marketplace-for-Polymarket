# Polymarket WebSocket API - Introduction

## Overview
Polymarket provides WebSocket APIs for real-time streaming of market data, orderbook updates, trades, and user-specific events. Use WebSockets to build responsive trading interfaces and live data dashboards.

## WebSocket URLs
- **CLOB/Trading**: `wss://ws-subscriptions-clob.polymarket.com/ws/`
- **Market Data**: `wss://ws-live-data.polymarket.com`

## Key Features
- **Real-time Orderbook**: Live bid/ask updates
- **Trade Streams**: Instant trade execution notifications
- **User Updates**: Order fills, cancellations, balance changes
- **Market Prices**: Live price updates for all markets
- **Low Latency**: Sub-second update delivery

## Why WebSockets?
Instead of polling REST APIs every few seconds:
- ✅ **Lower Latency**: Instant updates when events occur
- ✅ **Reduced Load**: Server pushes updates instead of constant polling
- ✅ **Better UX**: Real-time responsiveness
- ✅ **Efficient**: Less bandwidth and API calls

## Quick Start

### Python
```python
import websocket
import json

WS_URL = "wss://ws-subscriptions-clob.polymarket.com/ws/"

def on_message(ws, message):
    """Handle incoming messages"""
    data = json.loads(message)
    print(f"Received: {data}")

def on_error(ws, error):
    print(f"Error: {error}")

def on_close(ws, close_status_code, close_msg):
    print("Connection closed")

def on_open(ws):
    """Subscribe to market updates on connection"""
    subscribe_msg = {
        "auth": {},
        "type": "subscribe",
        "channel": "market",
        "market": "0x1234..."  # Market condition ID
    }
    ws.send(json.dumps(subscribe_msg))
    print("Subscribed to market updates")

# Create WebSocket connection
ws = websocket.WebSocketApp(
    WS_URL,
    on_open=on_open,
    on_message=on_message,
    on_error=on_error,
    on_close=on_close
)

# Start connection (blocking)
ws.run_forever()
```

### TypeScript
```typescript
const WS_URL = 'wss://ws-subscriptions-clob.polymarket.com/ws/';

// Create WebSocket connection
const ws = new WebSocket(WS_URL);

ws.onopen = () => {
  console.log('Connected to WebSocket');

  // Subscribe to market updates
  const subscribeMsg = {
    auth: {},
    type: 'subscribe',
    channel: 'market',
    market: '0x1234...',  // Market condition ID
  };

  ws.send(JSON.stringify(subscribeMsg));
  console.log('Subscribed to market updates');
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

ws.onclose = () => {
  console.log('Connection closed');
};
```

## Message Format

### Subscribe Message
```json
{
  "auth": {
    "apiKey": "YOUR_API_KEY",
    "secret": "YOUR_SECRET",
    "passphrase": "YOUR_PASSPHRASE"
  },
  "type": "subscribe",
  "channel": "market",
  "market": "0x1234..."
}
```

### Unsubscribe Message
```json
{
  "type": "unsubscribe",
  "channel": "market",
  "market": "0x1234..."
}
```

## Available Channels

### 1. Market Channel
Real-time market data (orderbook, trades, prices):
```json
{
  "type": "subscribe",
  "channel": "market",
  "market": "0x1234..."
}
```

### 2. User Channel
User-specific updates (authenticated):
```json
{
  "auth": {
    "apiKey": "YOUR_API_KEY",
    "secret": "YOUR_SECRET",
    "passphrase": "YOUR_PASSPHRASE"
  },
  "type": "subscribe",
  "channel": "user"
}
```

### 3. Ticker Channel
Price tickers for all markets:
```json
{
  "type": "subscribe",
  "channel": "ticker"
}
```

## Update Types

### Orderbook Updates
```json
{
  "event_type": "book",
  "asset_id": "71321...",
  "bids": [
    {"price": "0.52", "size": "100.0"}
  ],
  "asks": [
    {"price": "0.54", "size": "200.0"}
  ],
  "timestamp": 1234567890
}
```

### Trade Updates
```json
{
  "event_type": "trade",
  "asset_id": "71321...",
  "side": "BUY",
  "price": "0.53",
  "size": "50.0",
  "timestamp": 1234567890
}
```

### User Order Updates
```json
{
  "event_type": "order",
  "order_id": "0xabc...",
  "status": "FILLED",
  "filled_size": "100.0",
  "timestamp": 1234567890
}
```

## Connection Management

### Reconnection Logic
**Python**:
```python
import time

def connect_with_retry(max_retries=5):
    """Connect with exponential backoff"""
    retries = 0

    while retries < max_retries:
        try:
            ws = websocket.WebSocketApp(
                WS_URL,
                on_message=on_message,
                on_error=on_error,
                on_close=on_close,
                on_open=on_open
            )
            ws.run_forever()
        except Exception as e:
            retries += 1
            wait_time = min(2 ** retries, 60)  # Max 60 seconds
            print(f"Connection failed. Retrying in {wait_time}s...")
            time.sleep(wait_time)

connect_with_retry()
```

**TypeScript**:
```typescript
function connectWithRetry(maxRetries: number = 5) {
  let retries = 0;

  function connect() {
    const ws = new WebSocket(WS_URL);

    ws.onopen = () => {
      console.log('Connected');
      retries = 0;  // Reset on successful connection
    };

    ws.onclose = () => {
      if (retries < maxRetries) {
        retries++;
        const waitTime = Math.min(2 ** retries * 1000, 60000);
        console.log(`Reconnecting in ${waitTime/1000}s...`);
        setTimeout(connect, waitTime);
      }
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  connect();
}

connectWithRetry();
```

### Heartbeat/Ping
**Python**:
```python
import threading

def send_ping(ws):
    """Send periodic ping to keep connection alive"""
    def ping_loop():
        while True:
            time.sleep(30)  # Ping every 30 seconds
            try:
                ws.send(json.dumps({"type": "ping"}))
            except:
                break

    thread = threading.Thread(target=ping_loop, daemon=True)
    thread.start()

def on_open(ws):
    send_ping(ws)
    # Subscribe to channels...
```

**TypeScript**:
```typescript
function startHeartbeat(ws: WebSocket) {
  const interval = setInterval(() => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(JSON.stringify({ type: 'ping' }));
    } else {
      clearInterval(interval);
    }
  }, 30000);  // Ping every 30 seconds

  return interval;
}

ws.onopen = () => {
  const heartbeat = startHeartbeat(ws);
  // Subscribe to channels...
};
```

## Best Practices
1. **Implement reconnection** - Handle disconnects gracefully
2. **Send heartbeats** - Keep connection alive
3. **Validate messages** - Check message structure
4. **Rate limit subscriptions** - Don't subscribe to too many channels
5. **Handle errors** - Log and recover from errors
6. **Buffer updates** - Process updates in batches for UI

## Libraries

### Python
```bash
pip install websocket-client
```

### TypeScript/JavaScript
Native `WebSocket` API available in browsers and Node.js (v20+).

For Node.js < 20:
```bash
npm install ws
```

## Available Commands
- `/ws-intro` - This overview
- `/ws-user-channel` - User-specific streams
- `/ws-market-channel` - Market data streams

## Resources
- [WebSocket Docs](https://docs.polymarket.com/developers/websocket)
- [WebSocket Spec](https://tools.ietf.org/html/rfc6455)
