# Polymarket WebSocket API - User Channel

## Overview
The user channel provides real-time updates for user-specific events including order fills, cancellations, balance changes, and position updates.

**Requires Authentication**: User channel requires API credentials.

## Subscribe to User Channel

### Python
```python
import websocket
import json
import os

WS_URL = "wss://ws-subscriptions-clob.polymarket.com/ws/"

def on_message(ws, message):
    """Handle user updates"""
    data = json.loads(message)

    event_type = data.get('event_type')

    if event_type == 'order':
        print(f"Order Update: {data['order_id']}")
        print(f"  Status: {data['status']}")
        print(f"  Filled: {data['filled_size']}/{data['size']}")
    elif event_type == 'trade':
        print(f"Trade Executed: {data['side']} {data['size']} @ {data['price']}")
    elif event_type == 'balance':
        print(f"Balance Update: ${data['balance']:.2f}")

def on_open(ws):
    """Subscribe to user channel with authentication"""
    subscribe_msg = {
        "auth": {
            "apiKey": os.getenv("POLYMARKET_API_KEY"),
            "secret": os.getenv("POLYMARKET_SECRET"),
            "passphrase": os.getenv("POLYMARKET_PASSPHRASE")
        },
        "type": "subscribe",
        "channel": "user"
    }
    ws.send(json.dumps(subscribe_msg))
    print("Subscribed to user updates")

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
  // Subscribe to user channel with authentication
  const subscribeMsg = {
    auth: {
      apiKey: process.env.POLYMARKET_API_KEY,
      secret: process.env.POLYMARKET_SECRET,
      passphrase: process.env.POLYMARKET_PASSPHRASE,
    },
    type: 'subscribe',
    channel: 'user',
  };

  ws.send(JSON.stringify(subscribeMsg));
  console.log('Subscribed to user updates');
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  const eventType = data.event_type;

  if (eventType === 'order') {
    console.log(`Order Update: ${data.order_id}`);
    console.log(`  Status: ${data.status}`);
    console.log(`  Filled: ${data.filled_size}/${data.size}`);
  } else if (eventType === 'trade') {
    console.log(`Trade Executed: ${data.side} ${data.size} @ ${data.price}`);
  } else if (eventType === 'balance') {
    console.log(`Balance Update: $${data.balance.toFixed(2)}`);
  }
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};
```

## Event Types

### Order Events
Receive updates when your orders change status.

**Event Format**:
```json
{
  "event_type": "order",
  "order_id": "0xabc123...",
  "market": "0x1234...",
  "asset_id": "71321...",
  "side": "BUY",
  "price": "0.52",
  "size": "100.0",
  "filled_size": "50.0",
  "status": "PARTIAL",
  "timestamp": 1234567890
}
```

**Order Statuses**:
- `LIVE`: Order active on orderbook
- `PARTIAL`: Partially filled
- `FILLED`: Completely filled
- `CANCELLED`: Order cancelled

### Trade Events
Receive notifications when your orders are filled.

**Event Format**:
```json
{
  "event_type": "trade",
  "trade_id": "0xdef456...",
  "order_id": "0xabc123...",
  "market": "0x1234...",
  "asset_id": "71321...",
  "side": "BUY",
  "price": "0.52",
  "size": "25.0",
  "fee": "0.25",
  "timestamp": 1234567890
}
```

### Balance Events
Receive updates when your balance changes.

**Event Format**:
```json
{
  "event_type": "balance",
  "asset": "USDC",
  "balance": "1000.50",
  "available": "950.25",
  "locked": "50.25",
  "timestamp": 1234567890
}
```

### Position Events
Receive updates when your positions change.

**Event Format**:
```json
{
  "event_type": "position",
  "market": "0x1234...",
  "asset_id": "71321...",
  "outcome": "Yes",
  "size": "100.0",
  "avg_cost": "0.52",
  "current_price": "0.55",
  "pnl": "3.00",
  "timestamp": 1234567890
}
```

## Advanced Usage

### Order Tracking
**Python**:
```python
class OrderTracker:
    def __init__(self):
        self.active_orders = {}

    def handle_message(self, message):
        data = json.loads(message)

        if data.get('event_type') == 'order':
            order_id = data['order_id']
            status = data['status']

            if status in ['LIVE', 'PARTIAL']:
                self.active_orders[order_id] = data
            elif status in ['FILLED', 'CANCELLED']:
                self.active_orders.pop(order_id, None)

            print(f"Active orders: {len(self.active_orders)}")

tracker = OrderTracker()

def on_message(ws, message):
    tracker.handle_message(message)

# Use in WebSocket
ws = websocket.WebSocketApp(WS_URL, on_message=on_message)
```

**TypeScript**:
```typescript
class OrderTracker {
  private activeOrders: Map<string, any> = new Map();

  handleMessage(message: string) {
    const data = JSON.parse(message);

    if (data.event_type === 'order') {
      const orderId = data.order_id;
      const status = data.status;

      if (['LIVE', 'PARTIAL'].includes(status)) {
        this.activeOrders.set(orderId, data);
      } else if (['FILLED', 'CANCELLED'].includes(status)) {
        this.activeOrders.delete(orderId);
      }

      console.log(`Active orders: ${this.activeOrders.size}`);
    }
  }

  getActiveOrders() {
    return Array.from(this.activeOrders.values());
  }
}

const tracker = new OrderTracker();

ws.onmessage = (event) => {
  tracker.handleMessage(event.data);
};
```

### P&L Tracking
**Python**:
```python
class PnLTracker:
    def __init__(self):
        self.total_pnl = 0
        self.trades = []

    def handle_trade(self, trade_data):
        """Track realized P&L from trades"""
        self.trades.append(trade_data)

        # Calculate realized P&L
        # (Simplified - actual calculation depends on position)
        if trade_data['side'] == 'SELL':
            # Assume profit on sell
            profit = float(trade_data['size']) * (float(trade_data['price']) - 0.5)  # Simplified
            self.total_pnl += profit
            print(f"Realized P&L: ${self.total_pnl:.2f}")

pnl_tracker = PnLTracker()

def on_message(ws, message):
    data = json.loads(message)
    if data.get('event_type') == 'trade':
        pnl_tracker.handle_trade(data)
```

**TypeScript**:
```typescript
class PnLTracker {
  private totalPnl: number = 0;
  private trades: any[] = [];

  handleTrade(tradeData: any) {
    this.trades.push(tradeData);

    // Calculate realized P&L
    if (tradeData.side === 'SELL') {
      const profit = parseFloat(tradeData.size) * (parseFloat(tradeData.price) - 0.5);
      this.totalPnl += profit;
      console.log(`Realized P&L: $${this.totalPnl.toFixed(2)}`);
    }
  }

  getTotalPnL() {
    return this.totalPnl;
  }
}

const pnlTracker = new PnLTracker();

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.event_type === 'trade') {
    pnlTracker.handleTrade(data);
  }
};
```

### Alert System
**Python**:
```python
def create_alert_handler(alert_callback):
    """Create handler that triggers alerts"""
    def on_message(ws, message):
        data = json.loads(message)

        # Alert on order fill
        if data.get('event_type') == 'order' and data.get('status') == 'FILLED':
            alert_callback(f"Order filled: {data['order_id']}")

        # Alert on large trade
        if data.get('event_type') == 'trade':
            size = float(data['size'])
            if size > 1000:
                alert_callback(f"Large trade: {size} shares @ ${data['price']}")

    return on_message

def send_alert(message):
    print(f"🔔 ALERT: {message}")
    # Could send email, SMS, push notification, etc.

ws = websocket.WebSocketApp(
    WS_URL,
    on_message=create_alert_handler(send_alert)
)
```

**TypeScript**:
```typescript
function createAlertHandler(alertCallback: (msg: string) => void) {
  return (event: MessageEvent) => {
    const data = JSON.parse(event.data);

    // Alert on order fill
    if (data.event_type === 'order' && data.status === 'FILLED') {
      alertCallback(`Order filled: ${data.order_id}`);
    }

    // Alert on large trade
    if (data.event_type === 'trade') {
      const size = parseFloat(data.size);
      if (size > 1000) {
        alertCallback(`Large trade: ${size} shares @ $${data.price}`);
      }
    }
  };
}

function sendAlert(message: string) {
  console.log(`🔔 ALERT: ${message}`);
  // Could send email, SMS, push notification, etc.
}

ws.onmessage = createAlertHandler(sendAlert);
```

## Authentication

### API Credentials
User channel requires API credentials:
```json
{
  "auth": {
    "apiKey": "YOUR_API_KEY",
    "secret": "YOUR_SECRET",
    "passphrase": "YOUR_PASSPHRASE"
  }
}
```

### Environment Variables
**Python**:
```python
import os
from dotenv import load_dotenv

load_dotenv()

auth = {
    "apiKey": os.getenv("POLYMARKET_API_KEY"),
    "secret": os.getenv("POLYMARKET_SECRET"),
    "passphrase": os.getenv("POLYMARKET_PASSPHRASE")
}
```

**TypeScript**:
```typescript
import * as dotenv from 'dotenv';
dotenv.config();

const auth = {
  apiKey: process.env.POLYMARKET_API_KEY!,
  secret: process.env.POLYMARKET_SECRET!,
  passphrase: process.env.POLYMARKET_PASSPHRASE!,
};
```

## Best Practices
1. **Store credentials securely** - Use environment variables
2. **Handle reconnection** - Resubscribe on reconnect
3. **Validate events** - Check event_type before processing
4. **Track order state** - Maintain local order book
5. **Log events** - Keep audit trail of important events
6. **Rate limit actions** - Don't spam orders based on events

## Resources
- [User Channel Docs](https://docs.polymarket.com/developers/websocket/user)
- [Authentication Guide](https://docs.polymarket.com/developers/CLOB/authentication)
