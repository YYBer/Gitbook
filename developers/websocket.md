# WebSocket API

The Fibe WebSocket API provides real-time streaming data for market updates, orderbook changes, trade events, and user order updates.

## Connection

### WebSocket URLs

| Environment | URL |
|-------------|-----|
| **Mainnet** | `wss://api.fibe.com/ws` |
| **Testnet** | `wss://api.fibe-testnet.com/ws` |

### Using the SDK (Recommended)

```typescript
import { api } from '<package-name>';

const subscriptions = await api.WebSocketSubscriptions.create(
  'wss://api.fibe.com/ws',
  (error) => console.error('WebSocket error:', error)
);
```

### Raw WebSocket Connection

```typescript
const ws = new WebSocket('wss://api.fibe.com/ws');

ws.onopen = () => {
  console.log('Connected');
};

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log('Received:', message);
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

ws.onclose = () => {
  console.log('Disconnected');
};
```

---

## Subscription Channels

### L2 Orderbook

Subscribe to real-time orderbook updates for a specific market.

**SDK:**
```typescript
const subscription = subscriptions.subscribeToL2Book(
  0,      // marketIndex
  0.01,   // priceStep
  (data) => {
    console.log('Orderbook update:', data);
    console.log('Best bid:', data.bids[0]);
    console.log('Best ask:', data.asks[0]);
  }
);

// Unsubscribe when done
subscription.unsubscribe();
```

**Raw WebSocket:**
```typescript
// Subscribe
ws.send(JSON.stringify({
  method: 'subscribe',
  subscription: {
    type: 'l2Book',
    market: 0,
    ps: 1000000  // scaled price step
  }
}));

// Unsubscribe
ws.send(JSON.stringify({
  method: 'unsubscribe',
  subscription: {
    type: 'l2Book',
    market: 0,
    ps: 1000000
  }
}));
```

**Message Format:**
```json
{
  "channel": "l2Book",
  "data": {
    "market": 0,
    "ps": 1000000,
    "bids": [
      { "px": "103.44", "sz": "125.5" },
      { "px": "103.43", "sz": "250.0" }
    ],
    "asks": [
      { "px": "103.46", "sz": "100.0" },
      { "px": "103.47", "sz": "200.0" }
    ]
  }
}
```

---

### Trades

Subscribe to real-time trade events for a market.

**SDK:**
```typescript
const subscription = subscriptions.subscribeToTrades(
  0,  // marketIndex
  (trades) => {
    trades.forEach(trade => {
      console.log(`${trade.side === 'B' ? 'BUY' : 'SELL'} ${trade.sz} @ ${trade.px}`);
    });
  }
);
```

**Raw WebSocket:**
```typescript
ws.send(JSON.stringify({
  method: 'subscribe',
  subscription: {
    type: 'trades',
    market: 0
  }
}));
```

**Message Format:**
```json
{
  "channel": "trades",
  "data": [
    {
      "market": 0,
      "px": "103.45",
      "sz": "10.5",
      "side": "B",
      "time": 1704067200000,
      "oid": "123456789",
      "txId": "5KQw..."
    }
  ]
}
```

---

### All Mids (Price Updates)

Subscribe to mid price updates for all markets.

**SDK:**
```typescript
const subscription = subscriptions.subscribeToAllMids((mids) => {
  console.log('SOL-USDC price:', mids['0']);
});
```

**Raw WebSocket:**
```typescript
ws.send(JSON.stringify({
  method: 'subscribe',
  subscription: {
    type: 'allMids'
  }
}));
```

**Message Format:**
```json
{
  "channel": "allMids",
  "data": {
    "0": "103.45",
    "1": "0.00001234"
  }
}
```

---

### Candles

Subscribe to real-time candle updates.

**SDK:**
```typescript
import { api } from '<package-name>';

const subscription = subscriptions.subscribeToCandle(
  0,  // marketIndex
  api.CandleInterval.Min1,  // interval
  (candle) => {
    console.log(`Open: ${candle.o}, Close: ${candle.c}, High: ${candle.h}, Low: ${candle.l}`);
  }
);
```

**Raw WebSocket:**
```typescript
ws.send(JSON.stringify({
  method: 'subscribe',
  subscription: {
    type: 'candle',
    market: 0,
    interval: '1m'
  }
}));
```

**Message Format:**
```json
{
  "channel": "candle",
  "data": {
    "market": 0,
    "t": 1704067200000,
    "T": 1704067259999,
    "o": "103.00",
    "h": "103.50",
    "l": "102.80",
    "c": "103.45",
    "bv": "125.5",
    "qv": "12984.67",
    "n": 23
  }
}
```

---

### Order Updates

Subscribe to real-time updates for a user's orders.

**SDK:**
```typescript
const subscription = subscriptions.subscribeToOrderUpdates(
  'user_wallet_address',
  (data) => {
    console.log(`User: ${data.user}`);
    data.orders.forEach(order => {
      console.log(`Order ${order.oid}: ${order.status}`);
    });
  }
);
```

**Raw WebSocket:**
```typescript
ws.send(JSON.stringify({
  method: 'subscribe',
  subscription: {
    type: 'orderUpdates',
    user: 'user_wallet_address'
  }
}));
```

**Message Format:**
```json
{
  "channel": "orderUpdates",
  "data": {
    "user": "7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU",
    "orders": [
      {
        "market": 0,
        "oid": "1704067200000",
        "type": "L",
        "px": "100.00",
        "side": "B",
        "origSz": "10.0",
        "sz": "5.0",
        "status": "O",
        "txId": "5KQw...",
        "createdAt": 1704067200000,
        "updatedAt": 1704067300000
      }
    ]
  }
}
```

---

## Connection Management

### Ping/Pong

The SDK automatically handles ping/pong to keep the connection alive. If using raw WebSocket:

```typescript
// Send ping every 15 seconds
setInterval(() => {
  if (ws.readyState === WebSocket.OPEN) {
    ws.send(JSON.stringify({ method: 'ping' }));
  }
}, 15000);

// Handle pong response
ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  if (message.channel === 'pong') {
    // Connection is alive
  }
};
```

### Auto Reconnection

The SDK automatically reconnects on disconnection with exponential backoff:

- Initial delay: 1 second
- Max delay: 30 seconds
- Max attempts: 5

When using the SDK, subscriptions are automatically restored after reconnection.

### Manual Reconnection (Raw WebSocket)

```typescript
function connect() {
  const ws = new WebSocket('wss://api.fibe.com/ws');

  ws.onclose = () => {
    console.log('Disconnected, reconnecting...');
    setTimeout(connect, 5000);
  };

  ws.onopen = () => {
    // Resubscribe to channels
    ws.send(JSON.stringify({
      method: 'subscribe',
      subscription: { type: 'allMids' }
    }));
  };
}

connect();
```

---

## Error Handling

### Error Messages

```json
{
  "channel": "error",
  "detail": "Invalid subscription type"
}
```

### SDK Error Handling

```typescript
const subscriptions = await api.WebSocketSubscriptions.create(
  'wss://api.fibe.com/ws',
  (error) => {
    console.error('WebSocket error:', error);
    // Handle error (e.g., show notification, retry logic)
  }
);
```

---

## Complete Example

```typescript
import { api } from '<package-name>';

async function main() {
  // Create WebSocket subscriptions
  const subscriptions = await api.WebSocketSubscriptions.create(
    'wss://api.fibe.com/ws',
    (error) => console.error('Error:', error)
  );

  // Track best prices
  let bestBid = '0';
  let bestAsk = '0';

  // Subscribe to orderbook
  const orderbookSub = subscriptions.subscribeToL2Book(0, 0.01, (data) => {
    if (data.bids.length > 0) bestBid = data.bids[0].px;
    if (data.asks.length > 0) bestAsk = data.asks[0].px;
    const spread = parseFloat(bestAsk) - parseFloat(bestBid);
    console.log(`Bid: ${bestBid} | Ask: ${bestAsk} | Spread: ${spread.toFixed(4)}`);
  });

  // Subscribe to trades
  const tradesSub = subscriptions.subscribeToTrades(0, (trades) => {
    trades.forEach(trade => {
      const side = trade.side === 'B' ? 'BUY ' : 'SELL';
      console.log(`[TRADE] ${side} ${trade.sz} @ ${trade.px}`);
    });
  });

  // Subscribe to all mid prices
  const midsSub = subscriptions.subscribeToAllMids((mids) => {
    console.log('[MIDS]', mids);
  });

  // Run for 60 seconds then cleanup
  setTimeout(() => {
    console.log('Unsubscribing...');
    orderbookSub.unsubscribe();
    tradesSub.unsubscribe();
    midsSub.unsubscribe();
  }, 60000);
}

main().catch(console.error);
```

---

## Rate Limits

| Resource | Limit |
|----------|-------|
| WebSocket connections | 100 per IP |
| WebSocket subscriptions | 1000 per IP |
| Messages sent to server | 2000 per minute (across all connections) |
| Inflight POST messages | 100 simultaneous |
| User-specific subscriptions | 10 unique users |

---

## Best Practices

1. **Subscribe only to what you need** - Each subscription consumes server resources

2. **Handle disconnections gracefully** - Use the SDK's built-in reconnection or implement your own

3. **Process messages asynchronously** - Don't block the message handler with heavy computation

4. **Validate message data** - Always check message structure before processing

5. **Use the SDK** - It handles reconnection, ping/pong, and subscription management automatically

6. **Implement exponential backoff** - When manually handling reconnection

```typescript
let reconnectAttempt = 0;

function getReconnectDelay() {
  const delay = Math.min(1000 * Math.pow(2, reconnectAttempt), 30000);
  reconnectAttempt++;
  return delay;
}

function onConnected() {
  reconnectAttempt = 0;  // Reset on successful connection
}
```
