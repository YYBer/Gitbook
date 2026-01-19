# Info API Reference

The Info API provides read-only access to market data, orderbooks, and user information. All endpoints use HTTP GET requests and return JSON responses.

## Base URL

```
https://api.fibe.com/api/v1
```

## Endpoints

### Markets

#### GET /markets

Get a list of all available trading markets.

**Request:**
```bash
curl https://api.fibe.com/api/v1/markets
```

**Response:**
```json
[
  {
    "market": "SOL-USDC",
    "marketIndex": 0,
    "baseMint": "So11111111111111111111111111111111111111112",
    "quoteMint": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "baseDecimals": 9,
    "quoteDecimals": 6,
    "tickSizeInQuoteBaseUnits": 100,
    "lotSizeInBaseBaseUnits": 1000000,
    "label": "SPOT",
    "lookupTable": "..."
  }
]
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `market` | string | Market name (e.g., "SOL-USDC") |
| `marketIndex` | integer | Unique market identifier |
| `baseMint` | string | Solana address of base token mint |
| `quoteMint` | string | Solana address of quote token mint |
| `baseDecimals` | integer | Decimal places for base token |
| `quoteDecimals` | integer | Decimal places for quote token |
| `tickSizeInQuoteBaseUnits` | integer | Minimum price increment |
| `lotSizeInBaseBaseUnits` | integer | Minimum quantity increment |
| `label` | string | Market label/category |
| `lookupTable` | string | Address lookup table for transaction optimization |

---

#### GET /market

Get detailed information about a specific market.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `marketIndex` | integer | Yes | The market index |

**Request:**
```bash
curl "https://api.fibe.com/api/v1/market?marketIndex=0"
```

**Response:**
```json
{
  "market": "SOL-USDC",
  "marketIndex": 0,
  "baseMint": "So11111111111111111111111111111111111111112",
  "quoteMint": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
  "baseDecimals": 9,
  "quoteDecimals": 6,
  "tickSizeInQuoteBaseUnits": 100,
  "lotSizeInBaseBaseUnits": 1000000
}
```

---

### Prices

#### GET /all-mids

Get current mid prices for all markets.

**Request:**
```bash
curl https://api.fibe.com/api/v1/all-mids
```

**Response:**
```json
{
  "0": "103.45",
  "1": "0.00001234"
}
```

The keys are market indices (as strings), values are mid prices as decimal strings.

---

#### GET /market-stats

Get 24-hour statistics for a specific market.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `marketIndex` | integer | Yes | The market index |

**Request:**
```bash
curl "https://api.fibe.com/api/v1/market-stats?marketIndex=0"
```

**Response:**
```json
{
  "bv24h": "125432.567",
  "qv24h": "12984532.45",
  "px": "103.45",
  "px24hAgo": "101.23"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `bv24h` | string | 24-hour base volume |
| `qv24h` | string | 24-hour quote volume |
| `px` | string \| null | Current price |
| `px24hAgo` | string \| null | Price 24 hours ago |

---

### Orderbook

#### GET /l2book

Get Level 2 orderbook data for a market.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `marketIndex` | integer | Yes | The market index |
| `priceStep` | number | Yes | Price aggregation step (e.g., 0.01, 0.1, 1) |

**Request:**
```bash
curl "https://api.fibe.com/api/v1/l2book?marketIndex=0&priceStep=0.01"
```

**Response:**
```json
[
  {
    "market": 0,
    "ps": 1000000,
    "bids": [
      { "px": "103.44", "sz": "125.5" },
      { "px": "103.43", "sz": "250.0" },
      { "px": "103.42", "sz": "500.0" }
    ],
    "asks": [
      { "px": "103.46", "sz": "100.0" },
      { "px": "103.47", "sz": "200.0" },
      { "px": "103.48", "sz": "350.0" }
    ]
  }
]
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `market` | integer | Market index |
| `ps` | integer | Price step (scaled) |
| `bids` | array | Bid levels, sorted by price descending |
| `asks` | array | Ask levels, sorted by price ascending |
| `bids[].px` | string | Price level |
| `bids[].sz` | string | Total size at this level |

---

### Trades

#### GET /recent-market-trades

Get recent trades for a market.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `marketIndex` | integer | Yes | The market index |

**Request:**
```bash
curl "https://api.fibe.com/api/v1/recent-market-trades?marketIndex=0"
```

**Response:**
```json
[
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
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `market` | integer | Market index |
| `px` | string | Trade price |
| `sz` | string | Trade size |
| `side` | string | Taker side: `"B"` (buy) or `"A"` (sell) |
| `time` | integer | Unix timestamp in milliseconds |
| `oid` | string | Order ID |
| `txId` | string | Solana transaction signature |

---

### Candles

#### GET /candles

Get OHLCV candlestick data.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `marketIndex` | integer | Yes | The market index |
| `startTimestamp` | integer | Yes | Start time (Unix ms) |
| `endTimestamp` | integer | Yes | End time (Unix ms) |
| `interval` | string | Yes | Candle interval |

**Supported Intervals:**

| Interval | Description |
|----------|-------------|
| `1m` | 1 minute |
| `3m` | 3 minutes |
| `5m` | 5 minutes |
| `15m` | 15 minutes |
| `30m` | 30 minutes |
| `1h` | 1 hour |
| `2h` | 2 hours |
| `4h` | 4 hours |
| `8h` | 8 hours |
| `12h` | 12 hours |
| `1d` | 1 day |
| `3d` | 3 days |
| `1w` | 1 week |
| `1M` | 1 month |

**Request:**
```bash
curl "https://api.fibe.com/api/v1/candles?marketIndex=0&startTimestamp=1704067200000&endTimestamp=1704153600000&interval=1h"
```

**Response:**
```json
[
  {
    "market": 0,
    "t": 1704067200000,
    "T": 1704070799999,
    "o": "103.00",
    "h": "104.50",
    "l": "102.80",
    "c": "103.45",
    "bv": "12543.5",
    "qv": "1298453.67",
    "n": 523
  }
]
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `market` | integer | Market index |
| `t` | integer | Candle open time (Unix ms) |
| `T` | integer | Candle close time (Unix ms) |
| `o` | string | Open price |
| `h` | string | High price |
| `l` | string | Low price |
| `c` | string | Close price |
| `bv` | string | Base volume |
| `qv` | string | Quote volume |
| `n` | integer | Number of trades |

---

### User Orders

#### GET /open-orders

Get all open orders for a user.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `user` | string | Yes | User's Solana wallet address |

**Request:**
```bash
curl "https://api.fibe.com/api/v1/open-orders?user=7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU"
```

**Response:**
```json
[
  {
    "market": 0,
    "oid": "1704067200000",
    "type": "L",
    "px": "100.00",
    "side": "B",
    "origSz": "10.0",
    "sz": "10.0",
    "status": "O",
    "txId": "5KQw...",
    "createdAt": 1704067200000,
    "updatedAt": 1704067200000
  }
]
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `market` | integer | Market index |
| `oid` | string | Order ID |
| `type` | string | Order type: `"M"` (market) or `"L"` (limit) |
| `px` | string | Order price |
| `side` | string | Side: `"B"` (bid/buy) or `"A"` (ask/sell) |
| `origSz` | string | Original order size |
| `sz` | string | Remaining unfilled size |
| `status` | string | Order status |
| `txId` | string | Transaction signature |
| `createdAt` | integer | Creation timestamp (Unix ms) |
| `updatedAt` | integer | Last update timestamp (Unix ms) |

---

#### GET /historical-orders

Get historical orders for a user.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `user` | string | Yes | User's Solana wallet address |
| `page` | integer | No | Page number (default: 0) |
| `pageSize` | integer | No | Results per page (default: 50) |

**Request:**
```bash
curl "https://api.fibe.com/api/v1/historical-orders?user=7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU&page=0&pageSize=50"
```

**Response:**

Same format as `/open-orders`.

---

## Data Types

### Order Type

| Value | Description |
|-------|-------------|
| `M` | Market order |
| `L` | Limit order |

### Order Status

| Value | Description |
|-------|-------------|
| `O` | Open |
| `F` | Filled |
| `C` | Cancelled |

### Side

| Value | Description |
|-------|-------------|
| `B` | Bid (Buy) |
| `A` | Ask (Sell) |

---

## Error Handling

All errors return a JSON object with an `error` field:

```json
{
  "error": "Invalid market index"
}
```

**HTTP Status Codes:**

| Code | Description |
|------|-------------|
| `200` | Success |
| `400` | Bad Request - Invalid parameters |
| `404` | Not Found - Resource doesn't exist |
| `429` | Too Many Requests - Rate limit exceeded |
| `500` | Internal Server Error |

---

## SDK Usage

Using the TypeScript SDK:

```typescript
import { api } from '<package-name>';

const client = api.createApiClient('https://api.fibe.com');

// Get all markets
const markets = await client.getMarkets();

// Get orderbook
const orderbook = await client.getL2Book({
  marketIndex: 0,
  priceStep: 0.01
});

// Get user's open orders
const orders = await client.getOpenOrders({
  user: '7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU'
});
```
