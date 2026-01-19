# Code Examples

Practical examples for common use cases with the Fibe SDK.

## Setup

All examples assume the following setup:

```typescript
import { api, exchange } from '@anthropic-ai/fibe-sdk';
import { Connection, Keypair } from '@solana/web3.js';

// API client for market data
const apiClient = api.createApiClient('https://api.fibe.com');

// Connection to Solana
const connection = new Connection('https://api.mainnet-beta.solana.com');

// Exchange client for trading
const exchangeClient = new exchange.ExchangeClient(
  connection,
  'confirmed',
  apiClient
);

// Wallet setup (use your own method to load keys)
const keypair = Keypair.fromSecretKey(/* your secret key */);
const wallet = {
  publicKey: keypair.publicKey,
  signTransaction: async (tx) => {
    tx.sign([keypair]);
    return tx;
  }
};
```

---

## Market Data

### Get All Markets

```typescript
async function listMarkets() {
  const markets = await apiClient.getMarkets();

  console.log('Available Markets:\n');
  markets.forEach((market) => {
    console.log(`${market.market}`);
    console.log(`  Index: ${market.marketIndex}`);
    console.log(`  Base: ${market.baseMint}`);
    console.log(`  Quote: ${market.quoteMint}`);
    console.log(`  Tick Size: ${market.tickSizeInQuoteBaseUnits}`);
    console.log(`  Lot Size: ${market.lotSizeInBaseBaseUnits}`);
    console.log('');
  });
}

listMarkets();
```

### Get Current Prices

```typescript
async function getCurrentPrices() {
  const mids = await apiClient.getAllMids();
  const markets = await apiClient.getMarkets();

  console.log('Current Prices:\n');
  markets.forEach((market) => {
    const price = mids[market.marketIndex.toString()];
    if (price) {
      console.log(`${market.market}: $${price}`);
    }
  });
}

getCurrentPrices();
```

### Get 24h Market Stats

```typescript
async function getMarketStats(marketIndex: number) {
  const market = await apiClient.getMarket({ marketIndex });
  const stats = await apiClient.getMarketStats({ marketIndex });

  console.log(`\n${market.market} - 24h Stats`);
  console.log('─'.repeat(30));

  if (stats.px && stats.px24hAgo) {
    const current = parseFloat(stats.px);
    const dayAgo = parseFloat(stats.px24hAgo);
    const change = ((current - dayAgo) / dayAgo) * 100;

    console.log(`Current Price: $${current.toFixed(4)}`);
    console.log(`24h Change: ${change >= 0 ? '+' : ''}${change.toFixed(2)}%`);
  }

  console.log(`24h Base Volume: ${stats.bv24h}`);
  console.log(`24h Quote Volume: $${parseFloat(stats.qv24h).toLocaleString()}`);
}

getMarketStats(0);
```

---

## Orderbook Analysis

### Get Best Bid/Ask and Spread

```typescript
async function getSpread(marketIndex: number) {
  const orderbook = await apiClient.getL2Book({
    marketIndex,
    priceStep: 0.01
  });

  if (orderbook.length === 0) {
    console.log('No orderbook data');
    return;
  }

  const book = orderbook[0];
  const bestBid = book.bids[0];
  const bestAsk = book.asks[0];

  if (!bestBid || !bestAsk) {
    console.log('Orderbook is empty');
    return;
  }

  const bidPrice = parseFloat(bestBid.px);
  const askPrice = parseFloat(bestAsk.px);
  const spread = askPrice - bidPrice;
  const spreadPercent = (spread / bidPrice) * 100;

  console.log(`Best Bid: $${bidPrice.toFixed(4)} (${bestBid.sz} size)`);
  console.log(`Best Ask: $${askPrice.toFixed(4)} (${bestAsk.sz} size)`);
  console.log(`Spread: $${spread.toFixed(4)} (${spreadPercent.toFixed(4)}%)`);
}

getSpread(0);
```

### Calculate Orderbook Depth

```typescript
async function calculateDepth(marketIndex: number, levels: number = 10) {
  const orderbook = await apiClient.getL2Book({
    marketIndex,
    priceStep: 0.01
  });

  if (orderbook.length === 0) return;

  const book = orderbook[0];

  // Calculate bid depth
  const bidDepth = book.bids
    .slice(0, levels)
    .reduce((sum, level) => sum + parseFloat(level.sz), 0);

  // Calculate ask depth
  const askDepth = book.asks
    .slice(0, levels)
    .reduce((sum, level) => sum + parseFloat(level.sz), 0);

  console.log(`\nOrderbook Depth (Top ${levels} levels)`);
  console.log('─'.repeat(30));
  console.log(`Bid Depth: ${bidDepth.toFixed(4)}`);
  console.log(`Ask Depth: ${askDepth.toFixed(4)}`);
  console.log(`Ratio (Bid/Ask): ${(bidDepth / askDepth).toFixed(2)}`);
}

calculateDepth(0, 10);
```

---

## Historical Data

### Get and Analyze Candles

```typescript
async function analyzeCandles(marketIndex: number, hours: number = 24) {
  const endTime = Date.now();
  const startTime = endTime - hours * 60 * 60 * 1000;

  const candles = await apiClient.getCandles({
    marketIndex,
    startTimestamp: startTime,
    endTimestamp: endTime,
    interval: api.CandleInterval.Hour1
  });

  if (candles.length === 0) {
    console.log('No candle data');
    return;
  }

  const closes = candles.map(c => parseFloat(c.c));
  const highs = candles.map(c => parseFloat(c.h));
  const lows = candles.map(c => parseFloat(c.l));
  const volumes = candles.map(c => parseFloat(c.bv));

  const high = Math.max(...highs);
  const low = Math.min(...lows);
  const open = parseFloat(candles[0].o);
  const close = parseFloat(candles[candles.length - 1].c);
  const totalVolume = volumes.reduce((a, b) => a + b, 0);
  const change = ((close - open) / open) * 100;

  console.log(`\n${hours}h Analysis`);
  console.log('─'.repeat(30));
  console.log(`Open: $${open.toFixed(4)}`);
  console.log(`High: $${high.toFixed(4)}`);
  console.log(`Low: $${low.toFixed(4)}`);
  console.log(`Close: $${close.toFixed(4)}`);
  console.log(`Change: ${change >= 0 ? '+' : ''}${change.toFixed(2)}%`);
  console.log(`Volume: ${totalVolume.toFixed(2)}`);
}

analyzeCandles(0, 24);
```

### Simple Moving Average

```typescript
async function calculateSMA(marketIndex: number, period: number = 20) {
  const endTime = Date.now();
  const startTime = endTime - (period + 5) * 60 * 60 * 1000;

  const candles = await apiClient.getCandles({
    marketIndex,
    startTimestamp: startTime,
    endTimestamp: endTime,
    interval: api.CandleInterval.Hour1
  });

  if (candles.length < period) {
    console.log(`Not enough data for ${period}-period SMA`);
    return;
  }

  const closes = candles.slice(-period).map(c => parseFloat(c.c));
  const sma = closes.reduce((a, b) => a + b, 0) / period;
  const currentPrice = closes[closes.length - 1];
  const position = currentPrice > sma ? 'above' : 'below';

  console.log(`\n${period}-Period SMA Analysis`);
  console.log('─'.repeat(30));
  console.log(`SMA(${period}): $${sma.toFixed(4)}`);
  console.log(`Current: $${currentPrice.toFixed(4)}`);
  console.log(`Price is ${position} SMA`);
}

calculateSMA(0, 20);
```

---

## Trading

### Place Limit Buy Order

```typescript
async function placeLimitBuy(
  marketIndex: number,
  price: string,
  quantity: string
) {
  const orderId = BigInt(Date.now());

  console.log(`Placing limit buy: ${quantity} @ $${price}`);

  const tx = await exchangeClient.placeLimitOrder({
    owner: wallet.publicKey,
    marketIndex,
    price,
    orderId,
    side: exchange.Side.Bid,
    baseQuantity: quantity,
  });

  await exchangeClient.sendAndConfirmTx(tx, wallet);
  console.log(`Order placed! ID: ${orderId}`);

  return orderId;
}

// Example: Buy 0.1 SOL at $100
placeLimitBuy(0, '100.00', '0.1');
```

### Place Market Buy Order

```typescript
async function placeMarketBuy(marketIndex: number, quantity: string) {
  const orderId = BigInt(Date.now());

  console.log(`Placing market buy: ${quantity}`);

  const tx = await exchangeClient.placeMarketOrder({
    owner: wallet.publicKey,
    marketIndex,
    orderId,
    side: exchange.Side.Bid,
    baseQuantity: quantity,
    slippage: 0.01,  // 1% slippage tolerance
  });

  await exchangeClient.sendAndConfirmTx(tx, wallet);
  console.log(`Market order executed! ID: ${orderId}`);
}

placeMarketBuy(0, '0.1');
```

### Cancel Order

```typescript
async function cancelOrder(marketIndex: number, orderId: bigint) {
  console.log(`Cancelling order: ${orderId}`);

  const tx = await exchangeClient.cancelOrder({
    signer: wallet.publicKey,
    marketIndex,
    orderId,
  });

  await exchangeClient.sendAndConfirmTx(tx, wallet);
  console.log('Order cancelled!');
}

cancelOrder(0, BigInt('1704067200000'));
```

### Cancel All Orders

```typescript
async function cancelAllOrders(marketIndex: number) {
  const orders = await apiClient.getOpenOrders({
    user: wallet.publicKey.toBase58()
  });

  const marketOrders = orders.filter(o => o.market === marketIndex);

  console.log(`Cancelling ${marketOrders.length} orders...`);

  for (const order of marketOrders) {
    const tx = await exchangeClient.cancelOrder({
      signer: wallet.publicKey,
      marketIndex,
      orderId: BigInt(order.oid),
    });
    await exchangeClient.sendAndConfirmTx(tx, wallet);
    console.log(`Cancelled: ${order.oid}`);
  }

  console.log('All orders cancelled!');
}

cancelAllOrders(0);
```

---

## Real-time Data

### Price Alert System

```typescript
async function priceAlert(marketIndex: number, targetPrice: number, above: boolean) {
  const subscriptions = await api.WebSocketSubscriptions.create(
    'wss://api.fibe.com/ws'
  );

  console.log(`Watching for price ${above ? 'above' : 'below'} $${targetPrice}...`);

  const subscription = subscriptions.subscribeToAllMids((mids) => {
    const price = parseFloat(mids[marketIndex.toString()]);

    if ((above && price >= targetPrice) || (!above && price <= targetPrice)) {
      console.log(`\nALERT! Price hit $${price}`);
      subscription.unsubscribe();
    }
  });
}

// Alert when SOL goes above $110
priceAlert(0, 110, true);
```

### Trade Volume Tracker

```typescript
async function trackVolume(marketIndex: number, durationMs: number = 60000) {
  const subscriptions = await api.WebSocketSubscriptions.create(
    'wss://api.fibe.com/ws'
  );

  let buyVolume = 0;
  let sellVolume = 0;
  let tradeCount = 0;

  console.log('Tracking trade volume...\n');

  const subscription = subscriptions.subscribeToTrades(marketIndex, (trades) => {
    trades.forEach(trade => {
      const size = parseFloat(trade.sz);
      if (trade.side === 'B') {
        buyVolume += size;
      } else {
        sellVolume += size;
      }
      tradeCount++;

      console.log(
        `${trade.side === 'B' ? 'BUY ' : 'SELL'} ${trade.sz.padStart(10)} @ $${trade.px}`
      );
    });
  });

  // Stop after duration
  setTimeout(() => {
    subscription.unsubscribe();
    console.log('\n' + '─'.repeat(40));
    console.log(`Total Trades: ${tradeCount}`);
    console.log(`Buy Volume: ${buyVolume.toFixed(4)}`);
    console.log(`Sell Volume: ${sellVolume.toFixed(4)}`);
    console.log(`Net: ${(buyVolume - sellVolume).toFixed(4)}`);
  }, durationMs);
}

trackVolume(0, 60000);  // Track for 1 minute
```

### Order Book Imbalance Monitor

```typescript
async function monitorImbalance(marketIndex: number) {
  const subscriptions = await api.WebSocketSubscriptions.create(
    'wss://api.fibe.com/ws'
  );

  const subscription = subscriptions.subscribeToL2Book(
    marketIndex,
    0.1,
    (book) => {
      // Calculate top 5 level depth
      const bidDepth = book.bids.slice(0, 5)
        .reduce((sum, l) => sum + parseFloat(l.sz), 0);
      const askDepth = book.asks.slice(0, 5)
        .reduce((sum, l) => sum + parseFloat(l.sz), 0);

      const imbalance = (bidDepth - askDepth) / (bidDepth + askDepth);
      const bar = '█'.repeat(Math.abs(Math.round(imbalance * 20)));

      if (imbalance > 0) {
        console.log(`BID  [${bar.padEnd(20)}] ${(imbalance * 100).toFixed(1)}%`);
      } else {
        console.log(`ASK  [${''.padStart(20 - bar.length)}${bar}] ${(imbalance * 100).toFixed(1)}%`);
      }
    }
  );

  // Run for 30 seconds
  setTimeout(() => subscription.unsubscribe(), 30000);
}

monitorImbalance(0);
```

---

## Account Management

### View Open Orders

```typescript
async function viewOpenOrders() {
  const orders = await apiClient.getOpenOrders({
    user: wallet.publicKey.toBase58()
  });

  if (orders.length === 0) {
    console.log('No open orders');
    return;
  }

  console.log(`\nOpen Orders (${orders.length})`);
  console.log('─'.repeat(60));

  orders.forEach(order => {
    const side = order.side === 'B' ? 'BUY ' : 'SELL';
    const filled = parseFloat(order.origSz) - parseFloat(order.sz);
    const fillPercent = (filled / parseFloat(order.origSz) * 100).toFixed(0);

    console.log(
      `${side} ${order.sz}/${order.origSz} (${fillPercent}%) @ $${order.px} | Market: ${order.market}`
    );
  });
}

viewOpenOrders();
```

### Order History

```typescript
async function viewOrderHistory(page: number = 0) {
  const orders = await apiClient.getHistoricalOrders({
    user: wallet.publicKey.toBase58(),
    page,
    pageSize: 20
  });

  console.log(`\nOrder History (Page ${page + 1})`);
  console.log('─'.repeat(60));

  orders.forEach(order => {
    const side = order.side === 'B' ? 'BUY ' : 'SELL';
    const status = order.status === 'F' ? 'FILLED' :
                   order.status === 'C' ? 'CANCELLED' : 'OPEN';
    const date = new Date(order.createdAt).toLocaleString();

    console.log(`${date} | ${side} ${order.origSz} @ $${order.px} | ${status}`);
  });
}

viewOrderHistory(0);
```

---

## Error Handling Pattern

```typescript
async function safeApiCall<T>(
  fn: () => Promise<T>,
  retries: number = 3
): Promise<T | null> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error.message);

      if (i === retries - 1) {
        console.error('All retries exhausted');
        return null;
      }

      // Exponential backoff
      const delay = 1000 * Math.pow(2, i);
      await new Promise(r => setTimeout(r, delay));
    }
  }
  return null;
}

// Usage
async function robustMarketData() {
  const markets = await safeApiCall(() => apiClient.getMarkets());
  if (markets) {
    console.log(`Found ${markets.length} markets`);
  }
}

robustMarketData();
```

---

## Next Steps

- [API Reference](api-reference.md) - Complete endpoint documentation
- [TypeScript SDK](typescript-sdk.md) - Full SDK reference
- [WebSocket API](websocket.md) - Real-time streaming guide
