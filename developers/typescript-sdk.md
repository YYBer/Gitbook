# TypeScript SDK

The official Fibe TypeScript SDK provides a complete interface for interacting with the Fibe decentralized exchange on Solana.

## Installation

```bash
npm install <package-name>
# or
yarn add <package-name>
```

### Peer Dependencies

The SDK requires the following Solana dependencies:

```bash
npm install @solana/web3.js @coral-xyz/anchor @solana/spl-token
```

## SDK Structure

The SDK exports two main modules:

```typescript
import { api, exchange } from '<package-name>';
```

| Module | Purpose |
|--------|---------|
| `api` | REST API client for market data (read-only) |
| `exchange` | Exchange client for trading operations (requires wallet) |

## API Client

The API client provides access to market data endpoints.

### Creating the Client

```typescript
import { api } from '<package-name>';

const apiClient = api.createApiClient('https://api.fibe.com');
```

### Available Methods

#### getMarkets()

Get all available markets.

```typescript
const markets = await apiClient.getMarkets();
// Returns: Market[]
```

#### getMarket(params)

Get a specific market by index.

```typescript
const market = await apiClient.getMarket({ marketIndex: 0 });
// Returns: Market
```

#### getAllMids()

Get mid prices for all markets.

```typescript
const mids = await apiClient.getAllMids();
// Returns: { "0": "103.45", "1": "0.00001234" }
```

#### getMarketStats(params)

Get 24-hour statistics for a market.

```typescript
const stats = await apiClient.getMarketStats({ marketIndex: 0 });
// Returns: { bv24h, qv24h, px, px24hAgo }
```

#### getL2Book(params)

Get Level 2 orderbook data.

```typescript
const orderbook = await apiClient.getL2Book({
  marketIndex: 0,
  priceStep: 0.01  // Price aggregation step
});
// Returns: L2Book[]
```

#### getCandles(params)

Get OHLCV candlestick data.

```typescript
import { api } from '<package-name>';

const candles = await apiClient.getCandles({
  marketIndex: 0,
  startTimestamp: Date.now() - 86400000, // 24 hours ago
  endTimestamp: Date.now(),
  interval: api.CandleInterval.Hour1
});
// Returns: Candle[]
```

**Available Intervals:**
```typescript
enum CandleInterval {
  Min1 = "1m",
  Min3 = "3m",
  Min5 = "5m",
  Min15 = "15m",
  Min30 = "30m",
  Hour1 = "1h",
  Hour2 = "2h",
  Hour4 = "4h",
  Hour8 = "8h",
  Hour12 = "12h",
  Day1 = "1d",
  Day3 = "3d",
  Week1 = "1w",
  Month1 = "1M"
}
```

#### getRecentMarketTrades(params)

Get recent trades for a market.

```typescript
const trades = await apiClient.getRecentMarketTrades({ marketIndex: 0 });
// Returns: Trade[]
```

#### getOpenOrders(params)

Get user's open orders.

```typescript
const orders = await apiClient.getOpenOrders({
  user: 'wallet_address_here'
});
// Returns: Order[]
```

#### getHistoricalOrders(params)

Get user's historical orders with pagination.

```typescript
const history = await apiClient.getHistoricalOrders({
  user: 'wallet_address_here',
  page: 0,
  pageSize: 50
});
// Returns: Order[]
```

---

## Exchange Client

The Exchange client handles all trading operations on the Fibe smart contract.

### Creating the Client

```typescript
import { api, exchange } from '<package-name>';
import { Connection } from '@solana/web3.js';

// Create API client first
const apiClient = api.createApiClient('https://api.fibe.com');

// Create connection to Solana
const connection = new Connection('https://api.mainnet-beta.solana.com');

// Create exchange client
const exchangeClient = new exchange.ExchangeClient(
  connection,
  'confirmed',  // Commitment level
  apiClient
);
```

### Wallet Interface

Trading operations require a wallet that implements the `Wallet` interface:

```typescript
interface Wallet {
  publicKey: PublicKey;
  signTransaction(tx: VersionedTransaction): Promise<VersionedTransaction>;
}
```

You can use any Solana wallet adapter or create one from a Keypair:

```typescript
import { Keypair } from '@solana/web3.js';

const keypair = Keypair.fromSecretKey(yourSecretKey);

const wallet = {
  publicKey: keypair.publicKey,
  signTransaction: async (tx) => {
    tx.sign([keypair]);
    return tx;
  }
};
```

---

### Place Limit Order

```typescript
const tx = await exchangeClient.placeLimitOrder({
  owner: wallet.publicKey,
  marketIndex: 0,
  price: '100.50',           // Price as string
  orderId: BigInt(Date.now()), // Unique order ID
  side: exchange.Side.Bid,   // Bid (buy) or Ask (sell)
  baseQuantity: '1.5',       // Quantity in base token
});

// Sign and send
await exchangeClient.sendAndConfirmTx(tx, wallet);
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `owner` | PublicKey | Yes | Wallet public key |
| `marketIndex` | number | Yes | Market index |
| `price` | string | Yes | Order price |
| `orderId` | bigint | Yes | Unique order ID |
| `side` | Side | Yes | `Side.Bid` or `Side.Ask` |
| `baseQuantity` | string | Yes* | Quantity in base token |
| `quoteQuantity` | string | Yes* | Quantity in quote token |
| `timeInForce` | TimeInForce | No | Default: `Gtc` |
| `referrer` | PublicKey | No | Referrer address |

*Either `baseQuantity` or `quoteQuantity` must be provided, not both.

---

### Place Market Order

```typescript
const tx = await exchangeClient.placeMarketOrder({
  owner: wallet.publicKey,
  marketIndex: 0,
  orderId: BigInt(Date.now()),
  side: exchange.Side.Bid,
  baseQuantity: '1.5',
  slippage: 0.05,  // 5% slippage tolerance (optional, default 5%)
});

await exchangeClient.sendAndConfirmTx(tx, wallet);
```

Market orders use `TimeInForce.Ioc` (Immediate Or Cancel) automatically.

---

### Cancel Order

```typescript
const tx = await exchangeClient.cancelOrder({
  signer: wallet.publicKey,
  marketIndex: 0,
  orderId: BigInt('1704067200000'),  // The order ID to cancel
});

await exchangeClient.sendAndConfirmTx(tx, wallet);
```

---

### Modify Order

Replace an existing order with new parameters:

```typescript
const tx = await exchangeClient.modifyOrder({
  owner: wallet.publicKey,
  marketIndex: 0,
  oldOrderId: BigInt('1704067200000'),
  newOrderId: BigInt(Date.now()),
  newPrice: '101.00',
  newSide: exchange.Side.Bid,
  newQuantity: '2.0',
  newTimeInForce: exchange.TimeInForce.Gtc,
});

await exchangeClient.sendAndConfirmTx(tx, wallet);
```

---

### Fee Estimation

Estimate fees before placing orders:

```typescript
// Estimate buy order fees (input quote, output base)
const bidEstimate = await exchangeClient.estimateBidMinInputQuoteAndFee(
  wallet.publicKey,
  0,           // marketIndex
  '1.5',       // baseQuantity
  exchange.TimeInForce.Ioc,
  '103.50',    // price (optional)
  0.05         // slippage (optional)
);
console.log('Min input:', bidEstimate.minInput);
console.log('Fee:', bidEstimate.fee);

// Estimate sell order fees (input base, output quote)
const askEstimate = await exchangeClient.estimateAskMaxOutputQuoteAndFee(
  wallet.publicKey,
  0,           // marketIndex
  '1.5',       // baseQuantity
  exchange.TimeInForce.Ioc
);
console.log('Max output:', askEstimate.output);
console.log('Fee:', askEstimate.fee);
```

---

## Types

### Side

```typescript
enum Side {
  Bid = "B",  // Buy
  Ask = "A"   // Sell
}
```

### TimeInForce

```typescript
enum TimeInForce {
  Gtc,       // Good Till Cancelled
  Ioc,       // Immediate Or Cancel (market orders)
  PostOnly,  // Only add liquidity
  Fok        // Fill Or Kill
}
```

### Market

```typescript
interface Market {
  market: string;
  marketIndex: number;
  baseMint: string;
  quoteMint: string;
  baseDecimals: number;
  quoteDecimals: number;
  tickSizeInQuoteBaseUnits: number;
  lotSizeInBaseBaseUnits: number;
  label?: string;
  lookupTable?: string;
}
```

### Order

```typescript
interface Order {
  market: number;
  oid: string;
  type: OrderType;  // "M" or "L"
  px: string;
  side: Side;
  origSz: string;
  sz: string;
  status: OrderStatus;  // "O", "F", or "C"
  txId: string;
  createdAt: number;
  updatedAt: number;
}
```

### L2Book

```typescript
interface L2Book {
  market: number;
  ps: number;
  bids: BookLevel[];
  asks: BookLevel[];
}

interface BookLevel {
  px: string;
  sz: string;
}
```

---

## Error Handling

Always wrap SDK calls in try-catch blocks:

```typescript
try {
  const tx = await exchangeClient.placeLimitOrder({
    owner: wallet.publicKey,
    marketIndex: 0,
    price: '100.00',
    orderId: BigInt(Date.now()),
    side: exchange.Side.Bid,
    baseQuantity: '1.5',
  });

  await exchangeClient.sendAndConfirmTx(tx, wallet);
  console.log('Order placed successfully');
} catch (error) {
  if (error.message.includes('insufficient funds')) {
    console.error('Not enough balance');
  } else if (error.message.includes('order not found')) {
    console.error('Order does not exist');
  } else {
    console.error('Error:', error.message);
  }
}
```

---

## Complete Example

```typescript
import { api, exchange } from '<package-name>';
import { Connection, Keypair } from '@solana/web3.js';

async function main() {
  // Setup
  const apiClient = api.createApiClient('https://api.fibe.com');
  const connection = new Connection('https://api.mainnet-beta.solana.com');
  const exchangeClient = new exchange.ExchangeClient(connection, 'confirmed', apiClient);

  // Load wallet (use your own method)
  const keypair = Keypair.fromSecretKey(/* your secret key */);
  const wallet = {
    publicKey: keypair.publicKey,
    signTransaction: async (tx) => { tx.sign([keypair]); return tx; }
  };

  // Get market info
  const markets = await apiClient.getMarkets();
  console.log('Available markets:', markets.map(m => m.market));

  // Get current price
  const mids = await apiClient.getAllMids();
  const currentPrice = parseFloat(mids['0']);
  console.log('SOL-USDC mid price:', currentPrice);

  // Place a limit buy order 1% below current price
  const buyPrice = (currentPrice * 0.99).toFixed(2);
  const orderId = BigInt(Date.now());

  const tx = await exchangeClient.placeLimitOrder({
    owner: wallet.publicKey,
    marketIndex: 0,
    price: buyPrice,
    orderId,
    side: exchange.Side.Bid,
    baseQuantity: '0.1',
  });

  await exchangeClient.sendAndConfirmTx(tx, wallet);
  console.log('Order placed at price:', buyPrice);

  // Check open orders
  const openOrders = await apiClient.getOpenOrders({
    user: wallet.publicKey.toBase58()
  });
  console.log('Open orders:', openOrders.length);
}

main().catch(console.error);
```

---

## Next Steps

- [WebSocket API](websocket.md) - Real-time data streaming
- [Examples](examples.md) - More code examples
- [Info API Reference](api-reference.md) - Complete REST API documentation
