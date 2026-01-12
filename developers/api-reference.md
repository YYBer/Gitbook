# API Reference

This page provides an interactive API reference powered by Swagger UI. You can explore all endpoints, view request/response schemas, and test API calls directly from your browser.

## Interactive Documentation

{% embed url="api-explorer.html" %}

Alternatively, you can view the raw OpenAPI specification:

- [Download OpenAPI JSON](../openapi.json)

## Main Endpoints

### Market Data Endpoints

#### GET /api/v1/markets
Get a list of all available markets.

**Response Example:**
```json
[
  {
    "market": "BTC-USDC",
    "marketIndex": 0,
    "baseMint": "...",
    "quoteMint": "...",
    "baseDecimals": 9,
    "quoteDecimals": 6
  }
]
```

#### GET /api/v1/all-mids
Get current mid prices for all markets.

**Response Example:**
```json
{
  "BTC-USDC": "50000.50",
  "ETH-USDC": "3000.25"
}
```

#### GET /api/v1/market
Get detailed information about a specific market.

**Parameters:**
- `marketIndex` (integer, required) - The market index

**Response Example:**
```json
{
  "market": "BTC-USDC",
  "marketIndex": 0,
  "baseMint": "...",
  "quoteMint": "...",
  "tickSizeInQuoteBaseUnits": 100,
  "lotSizeInBaseBaseUnits": 1000
}
```

#### GET /api/v1/market-stats
Get 24-hour statistics for a specific market.

**Parameters:**
- `marketIndex` (integer, required) - The market index

**Response Example:**
```json
{
  "bv24h": "1234567.89",
  "qv24h": "98765432.10",
  "px": "50000.50",
  "px24hAgo": "49500.00"
}
```

#### GET /api/v1/l2book
Get Level 2 orderbook data.

**Parameters:**
- `marketIndex` (integer, required) - The market index
- `priceStep` (integer, required) - Price aggregation step

**Response Example:**
```json
[
  {
    "market": 0,
    "ps": 100,
    "bids": [
      {"px": "50000.00", "sz": "1.5"},
      {"px": "49999.00", "sz": "2.0"}
    ],
    "asks": [
      {"px": "50001.00", "sz": "1.2"},
      {"px": "50002.00", "sz": "0.8"}
    ]
  }
]
```

#### GET /api/v1/candles
Get candlestick/OHLCV data.

**Parameters:**
- `marketIndex` (integer, required) - The market index
- `startTimestamp` (integer, required) - Start time in Unix timestamp
- `endTimestamp` (integer, required) - End time in Unix timestamp
- `interval` (string, required) - Candle interval: `1m`, `5m`, `15m`, `1h`, `4h`, `1d`, etc.

**Response Example:**
```json
[
  {
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
]
```

#### GET /api/v1/recent-market-trades
Get recent trades for a market.

**Parameters:**
- `marketIndex` (integer, required) - The market index

**Response Example:**
```json
[
  {
    "market": 0,
    "px": "50000.00",
    "sz": "1.5",
    "side": "B",
    "time": 1640000000000,
    "oid": "order123",
    "txId": "tx456"
  }
]
```

### Trading Endpoints

#### GET /api/v1/open-orders
Get open orders for a user.

**Parameters:**
- `user` (string, required) - User's wallet address

**Response Example:**
```json
[
  {
    "market": 0,
    "oid": "order123",
    "type": "L",
    "px": "50000.00",
    "side": "B",
    "origSz": "1.0",
    "sz": "1.0",
    "status": "O",
    "txId": "tx456",
    "createdAt": 1640000000000,
    "updatedAt": 1640000000000
  }
]
```

#### GET /api/v1/historical-orders
Get historical orders for a user.

**Parameters:**
- `user` (string, required) - User's wallet address
- `page` (integer, optional) - Page number for pagination
- `pageSize` (integer, optional) - Number of results per page

**Response Example:**
```json
[
  {
    "market": 0,
    "oid": "order123",
    "type": "M",
    "px": "50000.00",
    "side": "A",
    "origSz": "1.0",
    "sz": "0.0",
    "status": "F",
    "txId": "tx456",
    "createdAt": 1640000000000,
    "updatedAt": 1640000100000
  }
]
```

## Data Models

### Order Types
- `M` - Market Order
- `L` - Limit Order

### Order Status
- `O` - Open
- `F` - Filled
- `C` - Cancelled

### Side
- `B` - Bid (Buy)
- `A` - Ask (Sell)

### Candle Intervals
`1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `2h`, `4h`, `8h`, `12h`, `1d`, `3d`, `1w`, `1M`

## Error Handling

All errors follow this format:

```json
{
  "error": "Error description"
}
```

Common HTTP status codes:
- `200` - Success
- `400` - Bad Request (invalid parameters)
- `500` - Internal Server Error
