# WebSocket API

The Fibe WebSocket API provides real-time streaming data for market updates, orderbook changes, and trade events.

## Connection

### WebSocket URL

```
wss://api.fibe.com/ws
```

### Using the TypeScript SDK

```typescript
import { FibeWebSocket } from '@fibe/ts-sdk';

const ws = new FibeWebSocket({
  url: 'wss://api.fibe.com/ws',
  reconnect: true
});
```

### Raw WebSocket Connection

```typescript
const ws = new WebSocket('wss://api.fibe.com/ws');

ws.onopen = () => {
  console.log('Connected');
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

ws.onclose = () => {
  console.log('Disconnected');
};
```

## Subscription Channels

### L2 Orderbook Updates

Subscribe to real-time orderbook updates for a specific market.

```typescript
// Using SDK
ws.subscribe({
  type: 'l2Book',
  marketIndex: 0,
  priceStep: 100
});

// Raw message
ws.send(JSON.stringify({
  method: 'subscribe',
  channel: 'l2Book',
  params: {
    marketIndex: 0,
    priceStep: 100
  }
}));
```

**Message Format:**
```json
{
  "channel": "l2Book",
  "data": {
    "market": 0,
    "ps": 100,
    "bids": [
      {"px": "50000.00", "sz": "1.5"}
    ],
    "asks": [
      {"px": "50001.00", "sz": "1.2"}
    ]
  }
}
```

### Trade Stream

Subscribe to real-time trade updates.

```typescript
// Using SDK
ws.subscribe({
  type: 'trades',
  marketIndex: 0
});

// Raw message
ws.send(JSON.stringify({
  method: 'subscribe',
  channel: 'trades',
  params: {
    marketIndex: 0
  }
}));
```

**Message Format:**
```json
{
  "channel": "trades",
  "data": {
    "market": 0,
    "px": "50000.00",
    "sz": "1.5",
    "side": "B",
    "time": 1640000000000,
    "oid": "order123",
    "txId": "tx456"
  }
}
```

### Candle Updates

Subscribe to real-time candle updates.

```typescript
// Using SDK
ws.subscribe({
  type: 'candles',
  marketIndex: 0,
  interval: '1m'
});

// Raw message
ws.send(JSON.stringify({
  method: 'subscribe',
  channel: 'candles',
  params: {
    marketIndex: 0,
    interval: '1m'
  }
}));
```

**Message Format:**
```json
{
  "channel": "candles",
  "data": {
    "market": 0,
    "t": 1640000000000,
    "T": 1640000060000,
    "o": "50000.00",
    "c": "50100.00",
    "h": "50200.00",
    "l": "49900.00",
    "bv": "123.45",
    "qv": "6172500.00",
    "n": 150
  }
}
```

### All Markets Mid Prices

Subscribe to mid price updates for all markets.

```typescript
// Using SDK
ws.subscribe({
  type: 'allMids'
});

// Raw message
ws.send(JSON.stringify({
  method: 'subscribe',
  channel: 'allMids'
}));
```

**Message Format:**
```json
{
  "channel": "allMids",
  "data": {
    "BTC-USDC": "50000.50",
    "ETH-USDC": "3000.25",
    "SOL-USDC": "100.50"
  }
}
```

## Unsubscribe

```typescript
// Using SDK
ws.unsubscribe({
  type: 'l2Book',
  marketIndex: 0
});

// Raw message
ws.send(JSON.stringify({
  method: 'unsubscribe',
  channel: 'l2Book',
  params: {
    marketIndex: 0
  }
}));
```

## Event Handlers (SDK)

```typescript
import { FibeWebSocket } from '@fibe/ts-sdk';

const ws = new FibeWebSocket();

// Connection opened
ws.on('open', () => {
  console.log('WebSocket connected');
});

// Message received
ws.on('message', (data) => {
  console.log('Received:', data);
});

// Connection closed
ws.on('close', () => {
  console.log('WebSocket disconnected');
});

// Error occurred
ws.on('error', (error) => {
  console.error('WebSocket error:', error);
});

// Specific channel updates
ws.on('l2Book', (data) => {
  console.log('Orderbook update:', data);
});

ws.on('trades', (data) => {
  console.log('New trade:', data);
});
```

## Connection Management

### Auto Reconnect

The SDK automatically reconnects if the connection is lost:

```typescript
const ws = new FibeWebSocket({
  reconnect: true,
  reconnectInterval: 5000, // ms
  maxReconnectAttempts: 10
});
```

### Manual Connection Control

```typescript
// Connect
await ws.connect();

// Disconnect
ws.disconnect();

// Check connection status
if (ws.isConnected()) {
  console.log('Connected');
}
```

## Complete Example

```typescript
import { FibeWebSocket } from '@fibe/ts-sdk';

const ws = new FibeWebSocket({
  url: 'wss://api.fibe.com/ws',
  reconnect: true
});

// Connection handlers
ws.on('open', () => {
  console.log('Connected to Fibe WebSocket');

  // Subscribe to multiple channels
  ws.subscribe({ type: 'l2Book', marketIndex: 0, priceStep: 100 });
  ws.subscribe({ type: 'trades', marketIndex: 0 });
  ws.subscribe({ type: 'allMids' });
});

// Handle orderbook updates
ws.on('l2Book', (data) => {
  console.log('Best bid:', data.bids[0]);
  console.log('Best ask:', data.asks[0]);
});

// Handle trades
ws.on('trades', (trade) => {
  console.log(`Trade: ${trade.side === 'B' ? 'BUY' : 'SELL'} ${trade.sz} @ ${trade.px}`);
});

// Handle mid price updates
ws.on('allMids', (mids) => {
  console.log('BTC mid price:', mids['BTC-USDC']);
});

// Error handling
ws.on('error', (error) => {
  console.error('WebSocket error:', error);
});

// Connect
await ws.connect();
```

## Rate Limits

WebSocket connections are subject to the following limits:
- Maximum 10 concurrent connections per IP
- Maximum 100 subscriptions per connection
- Messages are throttled to 1000/second per connection

## Best Practices

1. **Subscribe to only what you need** - Each subscription consumes resources
2. **Handle reconnections gracefully** - Use the SDK's built-in reconnect feature
3. **Process messages asynchronously** - Don't block the message handler
4. **Implement exponential backoff** - For reconnection attempts
5. **Validate message data** - Always check message structure before processing
