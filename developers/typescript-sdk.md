# TypeScript SDK

The official Fibe TypeScript SDK provides a simple and intuitive way to interact with the Fibe API.

## Installation

```bash
npm install @fibe/ts-sdk
# or
yarn add @fibe/ts-sdk
```

## Quick Start

### Initialize the Client

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

// Initialize with default settings
const api = new FibeAPI();

// Or with custom configuration
const api = new FibeAPI({
  baseUrl: 'https://api.fibe.com',
  timeout: 10000
});
```

## API Methods

### Market Data

#### Get All Markets

```typescript
const markets = await api.getMarkets();
console.log(markets);
```

**Response:**
```typescript
Array<{
  market: string;
  marketIndex: number;
  baseMint: string;
  quoteMint: string;
  baseDecimals: number;
  quoteDecimals: number;
}>
```

#### Get All Mid Prices

```typescript
const mids = await api.getAllMids();
console.log(mids);
```

**Response:**
```typescript
{
  [market: string]: string;
}
```

#### Get Market Info

```typescript
const market = await api.getMarket({ marketIndex: 0 });
console.log(market);
```

#### Get Market Statistics

```typescript
const stats = await api.getMarketStats({ marketIndex: 0 });
console.log(stats);
// {
//   bv24h: "1234567.89",
//   qv24h: "98765432.10",
//   px: "50000.50",
//   px24hAgo: "49500.00"
// }
```

#### Get L2 Orderbook

```typescript
const orderbook = await api.getL2Book({
  marketIndex: 0,
  priceStep: 100
});
console.log(orderbook);
```

**Response:**
```typescript
Array<{
  market: number;
  ps: number;
  bids: Array<{ px: string; sz: string }>;
  asks: Array<{ px: string; sz: string }>;
}>
```

#### Get Candles (OHLCV)

```typescript
const candles = await api.getCandles({
  marketIndex: 0,
  startTimestamp: Date.now() - 86400000, // 24 hours ago
  endTimestamp: Date.now(),
  interval: '1h'
});
console.log(candles);
```

**Available Intervals:**
- `1m`, `3m`, `5m`, `15m`, `30m`
- `1h`, `2h`, `4h`, `8h`, `12h`
- `1d`, `3d`, `1w`, `1M`

#### Get Recent Trades

```typescript
const trades = await api.getRecentMarketTrades({ marketIndex: 0 });
console.log(trades);
```

### Trading

#### Get Open Orders

```typescript
const openOrders = await api.getOpenOrders({
  user: 'wallet_address_here'
});
console.log(openOrders);
```

#### Get Historical Orders

```typescript
const historicalOrders = await api.getHistoricalOrders({
  user: 'wallet_address_here',
  page: 0,
  pageSize: 50
});
console.log(historicalOrders);
```

## WebSocket Integration

For real-time data streaming, see the [WebSocket API](websocket.md) documentation.

```typescript
import { FibeWebSocket } from '@fibe/ts-sdk';

const ws = new FibeWebSocket();

// Subscribe to orderbook updates
ws.subscribe({
  type: 'l2Book',
  marketIndex: 0
});

ws.on('message', (data) => {
  console.log('Orderbook update:', data);
});
```

## Error Handling

The SDK throws errors for failed requests. Always wrap API calls in try-catch blocks:

```typescript
try {
  const markets = await api.getMarkets();
} catch (error) {
  if (error.response) {
    // API error response
    console.error('API Error:', error.response.data.error);
  } else {
    // Network or other error
    console.error('Error:', error.message);
  }
}
```

## TypeScript Types

The SDK includes full TypeScript type definitions:

```typescript
import type {
  Market,
  Order,
  Trade,
  Candle,
  L2Book
} from '@fibe/ts-sdk';

const market: Market = await api.getMarket({ marketIndex: 0 });
```

## Advanced Usage

### Custom Fetch Configuration

```typescript
const api = new FibeAPI({
  baseUrl: 'https://api.fibe.com',
  headers: {
    'X-Custom-Header': 'value'
  },
  timeout: 15000
});
```

### Retry Logic

```typescript
async function fetchWithRetry(fn: () => Promise<any>, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}

const markets = await fetchWithRetry(() => api.getMarkets());
```

## Examples

For complete code examples, see the [Examples](examples.md) page.

## Source Code

The TypeScript SDK is open source and available on GitHub:
- [GitHub Repository](https://github.com/fibe-io/ts-sdk)
- [Report Issues](https://github.com/fibe-io/ts-sdk/issues)
