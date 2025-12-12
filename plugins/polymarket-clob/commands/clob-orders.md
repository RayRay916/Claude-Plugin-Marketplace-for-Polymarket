# Polymarket CLOB API - Order Management

## Overview
Create, cancel, and manage orders on Polymarket's Central Limit Order Book.

**Authentication Required**: All order operations require L2 signature-based authentication (see `/clob-auth`).

## Order Types
- **GTC (Good Till Cancelled)**: Order remains active until filled or cancelled
- **FOK (Fill or Kill)**: Order must be filled immediately in full or cancelled
- **GTD (Good Till Date)**: Order expires at specified time

## Create Order

### Market Buy Order
**Python**:
```python
from py_clob_client.client import ClobClient
from py_clob_client.signer import Signer
from py_clob_client.order_builder.constants import BUY

# Initialize authenticated client
signer = Signer(private_key="YOUR_PRIVATE_KEY", chain_id=137)
client = ClobClient(
    host="https://clob.polymarket.com",
    chain_id=137,
    signer=signer
)

# Create a buy order
token_id = "71321045679252212594626385532706912750332728571942532289631379312455583992563"

order = client.create_order(
    token_id=token_id,
    price=0.52,  # Price in range [0.01, 0.99]
    size=100.0,  # Number of shares
    side=BUY,
    order_type="GTC"
)

print(f"Order ID: {order['orderID']}")
print(f"Status: {order['status']}")
```

**TypeScript**:
```typescript
import { ClobClient, Side, OrderType } from '@polymarket/clob-client';
import { Wallet } from 'ethers';

// Initialize authenticated client
const wallet = new Wallet('YOUR_PRIVATE_KEY');
const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  chainId: 137,
  signer: wallet,
});

const tokenId = '71321045679252212594626385532706912750332728571942532289631379312455583992563';

const order = await client.createOrder({
  tokenId,
  price: 0.52,
  size: 100.0,
  side: Side.BUY,
  orderType: OrderType.GTC,
});

console.log('Order ID:', order.orderID);
console.log('Status:', order.status);
```

### Market Sell Order
**Python**:
```python
from py_clob_client.order_builder.constants import SELL

sell_order = client.create_order(
    token_id=token_id,
    price=0.55,
    size=50.0,
    side=SELL,
    order_type="GTC"
)

print(f"Sell Order ID: {sell_order['orderID']}")
```

**TypeScript**:
```typescript
const sellOrder = await client.createOrder({
  tokenId,
  price: 0.55,
  size: 50.0,
  side: Side.SELL,
  orderType: OrderType.GTC,
});

console.log('Sell Order ID:', sellOrder.orderID);
```

### Fill-or-Kill Order
**Python**:
```python
# FOK order - must fill immediately in full
fok_order = client.create_order(
    token_id=token_id,
    price=0.53,
    size=25.0,
    side=BUY,
    order_type="FOK"
)
```

**TypeScript**:
```typescript
const fokOrder = await client.createOrder({
  tokenId,
  price: 0.53,
  size: 25.0,
  side: Side.BUY,
  orderType: OrderType.FOK,
});
```

## Get Orders

### Get All Orders for User
**Python**:
```python
# Get all orders for your account
orders = client.get_orders()

for order in orders:
    print(f"Order {order['id']}: {order['side']} {order['size']} @ {order['price']}")
    print(f"  Status: {order['status']}")
    print(f"  Filled: {order['size_matched']}")
```

**TypeScript**:
```typescript
const orders = await client.getOrders();

orders.forEach(order => {
  console.log(`Order ${order.id}: ${order.side} ${order.size} @ ${order.price}`);
  console.log(`  Status: ${order.status}`);
  console.log(`  Filled: ${order.size_matched}`);
});
```

### Get Order by ID
**Python**:
```python
order_id = "0x1234567890abcdef..."
order = client.get_order(order_id)

print(f"Order Status: {order['status']}")
print(f"Size: {order['size']}")
print(f"Filled: {order['size_matched']}")
print(f"Price: {order['price']}")
```

**TypeScript**:
```typescript
const orderId = '0x1234567890abcdef...';
const order = await client.getOrder(orderId);

console.log('Order Status:', order.status);
console.log('Size:', order.size);
console.log('Filled:', order.size_matched);
console.log('Price:', order.price);
```

### Get Orders for Market
**Python**:
```python
# Get all your orders for a specific market
market_orders = client.get_orders(market=token_id)

print(f"Active orders in market: {len(market_orders)}")
```

**TypeScript**:
```typescript
const marketOrders = await client.getOrders({ market: tokenId });
console.log('Active orders in market:', marketOrders.length);
```

## Cancel Orders

### Cancel Single Order
**Python**:
```python
# Cancel a specific order
result = client.cancel_order(order_id)

if result['success']:
    print(f"Order {order_id} cancelled successfully")
else:
    print(f"Failed to cancel: {result['error']}")
```

**TypeScript**:
```typescript
const result = await client.cancelOrder(orderId);

if (result.success) {
  console.log(`Order ${orderId} cancelled successfully`);
} else {
  console.log('Failed to cancel:', result.error);
}
```

### Cancel Multiple Orders
**Python**:
```python
# Cancel multiple orders at once
order_ids = ["0xabc...", "0xdef...", "0x123..."]
results = client.cancel_orders(order_ids)

for result in results:
    print(f"Order {result['orderID']}: {'Cancelled' if result['success'] else 'Failed'}")
```

**TypeScript**:
```typescript
const orderIds = ['0xabc...', '0xdef...', '0x123...'];
const results = await client.cancelOrders(orderIds);

results.forEach(result => {
  console.log(`Order ${result.orderID}: ${result.success ? 'Cancelled' : 'Failed'}`);
});
```

### Cancel All Orders
**Python**:
```python
# Cancel all active orders for your account
result = client.cancel_all_orders()
print(f"Cancelled {result['cancelled']} orders")
```

**TypeScript**:
```typescript
const result = await client.cancelAllOrders();
console.log(`Cancelled ${result.cancelled} orders`);
```

### Cancel Market Orders
**Python**:
```python
# Cancel all orders for a specific market
result = client.cancel_market_orders(market=token_id)
print(f"Cancelled {result['cancelled']} orders in market")
```

**TypeScript**:
```typescript
const result = await client.cancelMarketOrders({ market: tokenId });
console.log(`Cancelled ${result.cancelled} orders in market`);
```

## Advanced Order Management

### Replace Order (Cancel & Create)
**Python**:
```python
def replace_order(client, old_order_id, new_price, new_size):
    """Cancel existing order and create new one"""
    # Cancel old order
    client.cancel_order(old_order_id)

    # Get original order details
    old_order = client.get_order(old_order_id)

    # Create new order with updated params
    new_order = client.create_order(
        token_id=old_order['asset_id'],
        price=new_price,
        size=new_size,
        side=old_order['side'],
        order_type="GTC"
    )

    return new_order

# Replace order with new price
new_order = replace_order(client, "0xabc...", new_price=0.54, new_size=150)
```

**TypeScript**:
```typescript
async function replaceOrder(
  client: ClobClient,
  oldOrderId: string,
  newPrice: number,
  newSize: number
) {
  // Cancel old order
  await client.cancelOrder(oldOrderId);

  // Get original order details
  const oldOrder = await client.getOrder(oldOrderId);

  // Create new order with updated params
  const newOrder = await client.createOrder({
    tokenId: oldOrder.asset_id,
    price: newPrice,
    size: newSize,
    side: oldOrder.side,
    orderType: OrderType.GTC,
  });

  return newOrder;
}

const newOrder = await replaceOrder(client, '0xabc...', 0.54, 150);
```

### Monitor Order Status
**Python**:
```python
import time

def wait_for_fill(client, order_id, timeout=60):
    """Wait for order to be filled or timeout"""
    start_time = time.time()

    while time.time() - start_time < timeout:
        order = client.get_order(order_id)

        if order['status'] == 'FILLED':
            print(f"Order filled! Size: {order['size_matched']}")
            return True
        elif order['status'] == 'CANCELLED':
            print("Order was cancelled")
            return False

        print(f"Status: {order['status']}, Filled: {order['size_matched']}/{order['size']}")
        time.sleep(5)

    print("Timeout waiting for fill")
    return False

# Place order and wait for fill
order = client.create_order(token_id=token_id, price=0.52, size=100, side=BUY)
wait_for_fill(client, order['orderID'])
```

**TypeScript**:
```typescript
async function waitForFill(
  client: ClobClient,
  orderId: string,
  timeout: number = 60000
): Promise<boolean> {
  const startTime = Date.now();

  while (Date.now() - startTime < timeout) {
    const order = await client.getOrder(orderId);

    if (order.status === 'FILLED') {
      console.log('Order filled! Size:', order.size_matched);
      return true;
    } else if (order.status === 'CANCELLED') {
      console.log('Order was cancelled');
      return false;
    }

    console.log(`Status: ${order.status}, Filled: ${order.size_matched}/${order.size}`);
    await new Promise(resolve => setTimeout(resolve, 5000));
  }

  console.log('Timeout waiting for fill');
  return false;
}

const order = await client.createOrder({
  tokenId, price: 0.52, size: 100, side: Side.BUY
});
await waitForFill(client, order.orderID);
```

## Order Response Format
```json
{
  "orderID": "0x1234567890abcdef...",
  "status": "LIVE",
  "asset_id": "71321045679252212594626385532706912750332728571942532289631379312455583992563",
  "market": "0xabcd...",
  "side": "BUY",
  "price": "0.52",
  "size": "100.0",
  "size_matched": "0.0",
  "order_type": "GTC",
  "created_at": 1234567890,
  "expiration": null
}
```

## Best Practices
1. **Check balances first** - Ensure sufficient funds
2. **Validate prices** - Must be between 0.01 and 0.99
3. **Handle errors gracefully** - Network issues, insufficient liquidity
4. **Use FOK for price certainty** - When immediate execution is critical
5. **Monitor fills** - Don't assume instant execution
6. **Cancel stale orders** - Clean up unfilled orders regularly

## Resources
- [Order Management Docs](https://docs.polymarket.com/developers/CLOB/orders)
- [Order Types](https://docs.polymarket.com/developers/CLOB/order-types)
