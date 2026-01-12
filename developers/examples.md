# Code Examples

This page provides practical examples for common use cases with the Fibe API.

## Basic Market Data

### Get All Markets and Display Info

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function displayMarkets() {
  try {
    const markets = await api.getMarkets();

    console.log('Available Markets:\n');
    markets.forEach((market, index) => {
      console.log(`${index + 1}. ${market.market}`);
      console.log(`   Market Index: ${market.marketIndex}`);
      console.log(`   Base: ${market.baseMint.slice(0, 8)}...`);
      console.log(`   Quote: ${market.quoteMint.slice(0, 8)}...`);
      console.log('');
    });
  } catch (error) {
    console.error('Error fetching markets:', error);
  }
}

displayMarkets();
```

### Get Current Prices

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function getCurrentPrices() {
  const mids = await api.getAllMids();

  console.log('Current Market Prices:\n');
  Object.entries(mids).forEach(([market, price]) => {
    console.log(`${market}: $${price}`);
  });
}

getCurrentPrices();
```

### Get Market Statistics with 24h Change

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function getMarketStats(marketIndex: number) {
  const stats = await api.getMarketStats({ marketIndex });

  if (stats.px && stats.px24hAgo) {
    const currentPrice = parseFloat(stats.px);
    const price24hAgo = parseFloat(stats.px24hAgo);
    const change = ((currentPrice - price24hAgo) / price24hAgo) * 100;

    console.log('Market Statistics:');
    console.log(`Current Price: $${currentPrice}`);
    console.log(`24h Change: ${change.toFixed(2)}%`);
    console.log(`24h Volume (Base): ${stats.bv24h}`);
    console.log(`24h Volume (Quote): $${stats.qv24h}`);
  }
}

getMarketStats(0);
```

## Orderbook Analysis

### Get Best Bid and Ask

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function getBestPrices(marketIndex: number) {
  const orderbook = await api.getL2Book({
    marketIndex,
    priceStep: 100
  });

  if (orderbook.length > 0) {
    const book = orderbook[0];
    const bestBid = book.bids[0];
    const bestAsk = book.asks[0];

    console.log('Best Prices:');
    console.log(`Bid: $${bestBid.px} (Size: ${bestBid.sz})`);
    console.log(`Ask: $${bestAsk.px} (Size: ${bestAsk.sz})`);

    const spread = parseFloat(bestAsk.px) - parseFloat(bestBid.px);
    const spreadPercent = (spread / parseFloat(bestBid.px)) * 100;

    console.log(`Spread: $${spread.toFixed(2)} (${spreadPercent.toFixed(4)}%)`);
  }
}

getBestPrices(0);
```

### Calculate Orderbook Depth

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function calculateDepth(marketIndex: number, levels: number = 10) {
  const orderbook = await api.getL2Book({
    marketIndex,
    priceStep: 100
  });

  if (orderbook.length > 0) {
    const book = orderbook[0];

    const bidDepth = book.bids
      .slice(0, levels)
      .reduce((sum, level) => sum + parseFloat(level.sz), 0);

    const askDepth = book.asks
      .slice(0, levels)
      .reduce((sum, level) => sum + parseFloat(level.sz), 0);

    console.log(`Orderbook Depth (Top ${levels} levels):`);
    console.log(`Bid Side: ${bidDepth.toFixed(4)}`);
    console.log(`Ask Side: ${askDepth.toFixed(4)}`);
    console.log(`Total: ${(bidDepth + askDepth).toFixed(4)}`);
  }
}

calculateDepth(0, 10);
```

## Historical Data

### Get and Analyze Candles

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function analyzeCandleData(marketIndex: number) {
  const endTime = Date.now();
  const startTime = endTime - 24 * 60 * 60 * 1000; // 24 hours ago

  const candles = await api.getCandles({
    marketIndex,
    startTimestamp: startTime,
    endTimestamp: endTime,
    interval: '1h'
  });

  if (candles.length > 0) {
    const prices = candles.map(c => parseFloat(c.c));
    const high = Math.max(...prices);
    const low = Math.min(...prices);
    const first = parseFloat(candles[0].o);
    const last = parseFloat(candles[candles.length - 1].c);
    const change = ((last - first) / first) * 100;

    console.log('24h Analysis:');
    console.log(`High: $${high.toFixed(2)}`);
    console.log(`Low: $${low.toFixed(2)}`);
    console.log(`Open: $${first.toFixed(2)}`);
    console.log(`Close: $${last.toFixed(2)}`);
    console.log(`Change: ${change.toFixed(2)}%`);
  }
}

analyzeCandleData(0);
```

### Calculate Simple Moving Average

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function calculateSMA(marketIndex: number, period: number = 20) {
  const endTime = Date.now();
  const startTime = endTime - period * 60 * 60 * 1000; // hours ago

  const candles = await api.getCandles({
    marketIndex,
    startTimestamp: startTime,
    endTimestamp: endTime,
    interval: '1h'
  });

  if (candles.length >= period) {
    const prices = candles.slice(-period).map(c => parseFloat(c.c));
    const sma = prices.reduce((sum, price) => sum + price, 0) / period;

    console.log(`${period}-hour SMA: $${sma.toFixed(2)}`);
  }
}

calculateSMA(0, 20);
```

## Real-time Data with WebSocket

### Monitor Orderbook Updates

```typescript
import { FibeWebSocket } from '@fibe/ts-sdk';

const ws = new FibeWebSocket();

ws.on('open', () => {
  console.log('Connected to WebSocket');
  ws.subscribe({ type: 'l2Book', marketIndex: 0, priceStep: 100 });
});

ws.on('l2Book', (data) => {
  const bestBid = data.bids[0];
  const bestAsk = data.asks[0];
  const spread = parseFloat(bestAsk.px) - parseFloat(bestBid.px);

  console.log(`Bid: $${bestBid.px} | Ask: $${bestAsk.px} | Spread: $${spread.toFixed(2)}`);
});

ws.connect();
```

### Track Trade Volume

```typescript
import { FibeWebSocket } from '@fibe/ts-sdk';

const ws = new FibeWebSocket();
let totalVolume = 0;

ws.on('open', () => {
  console.log('Tracking trade volume...');
  ws.subscribe({ type: 'trades', marketIndex: 0 });
});

ws.on('trades', (trade) => {
  const volume = parseFloat(trade.sz);
  totalVolume += volume;

  console.log(`New trade: ${trade.side === 'B' ? 'BUY' : 'SELL'} ${trade.sz} @ $${trade.px}`);
  console.log(`Total volume tracked: ${totalVolume.toFixed(4)}`);
});

ws.connect();
```

### Price Alert System

```typescript
import { FibeWebSocket } from '@fibe/ts-sdk';

const ws = new FibeWebSocket();
const alertPrice = 50000; // Set your alert price

ws.on('open', () => {
  console.log('Price alert system active');
  ws.subscribe({ type: 'allMids' });
});

ws.on('allMids', (mids) => {
  const btcPrice = parseFloat(mids['BTC-USDC']);

  if (btcPrice >= alertPrice) {
    console.log(`🚨 ALERT: BTC reached $${btcPrice}!`);
    // Send notification, email, etc.
  }
});

ws.connect();
```

## Account Data

### Get User Orders

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();
const userAddress = 'your_wallet_address';

async function getUserOrders() {
  // Get open orders
  const openOrders = await api.getOpenOrders({ user: userAddress });
  console.log(`Open Orders: ${openOrders.length}`);

  openOrders.forEach((order) => {
    console.log(`- ${order.side === 'B' ? 'BUY' : 'SELL'} ${order.sz} @ $${order.px}`);
  });

  // Get order history
  const history = await api.getHistoricalOrders({
    user: userAddress,
    page: 0,
    pageSize: 10
  });
  console.log(`\nRecent Orders: ${history.length}`);
}

getUserOrders();
```

## Advanced: Multi-Market Monitor

```typescript
import { FibeAPI, FibeWebSocket } from '@fibe/ts-sdk';

const api = new FibeAPI();
const ws = new FibeWebSocket();

async function monitorMultipleMarkets() {
  // Get all markets
  const markets = await api.getMarkets();

  ws.on('open', () => {
    console.log('Monitoring all markets...\n');

    // Subscribe to all markets
    markets.forEach((market) => {
      ws.subscribe({
        type: 'trades',
        marketIndex: market.marketIndex
      });
    });
  });

  ws.on('trades', async (trade) => {
    const market = markets.find(m => m.marketIndex === trade.market);
    console.log(`[${market?.market}] ${trade.side === 'B' ? 'BUY' : 'SELL'} ${trade.sz} @ $${trade.px}`);
  });

  await ws.connect();
}

monitorMultipleMarkets();
```

## Error Handling Pattern

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();

async function robustAPICall<T>(
  apiCall: () => Promise<T>,
  retries = 3
): Promise<T | null> {
  for (let i = 0; i < retries; i++) {
    try {
      return await apiCall();
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error);

      if (i === retries - 1) {
        console.error('All retries exhausted');
        return null;
      }

      // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
  return null;
}

// Usage
async function example() {
  const markets = await robustAPICall(() => api.getMarkets());
  if (markets) {
    console.log('Successfully fetched markets:', markets.length);
  }
}

example();
```

## Next Steps

- Explore the [API Reference](api-reference.md) for complete endpoint documentation
- Check out the [TypeScript SDK](typescript-sdk.md) guide for more SDK features
- Learn about [WebSocket API](websocket.md) for real-time data streaming
