# API Overview

Welcome to the Fibe API documentation. The Fibe API allows developers to programmatically access market data and execute trades on the Fibe decentralized exchange built on Solana.

## Base URLs

| Environment | REST API | WebSocket |
|-------------|----------|-----------|
| **Mainnet** | `https://api.fibe.com` | `wss://api.fibe.com/ws` |
| **Testnet** | `https://api.fibe-testnet.com` | `wss://api.fibe-testnet.com/ws` |

## Architecture Overview

The Fibe API consists of two main components:

### 1. Info API (REST)
Read-only endpoints for retrieving market data, orderbooks, candles, and user orders. These are standard HTTP GET requests that don't require authentication.

### 2. Exchange Client (On-chain)
Trading operations (place, cancel, modify orders) are executed directly on the Solana blockchain through the Fibe smart contract. These require a Solana wallet for signing transactions.

```
┌─────────────────────────────────────────────────────────────┐
│                        Your Application                      │
└─────────────────────────────────────────────────────────────┘
                │                           │
                ▼                           ▼
┌─────────────────────────┐   ┌─────────────────────────────┐
│      Info API (REST)    │   │    Exchange Client (Solana)  │
│  - Market Data          │   │  - Place Orders              │
│  - Orderbook            │   │  - Cancel Orders             │
│  - Candles              │   │  - Modify Orders             │
│  - User Orders          │   │  - Requires Wallet Signature │
└─────────────────────────┘   └─────────────────────────────┘
```

## Quick Start

### Installation

```bash
npm install @anthropic-ai/fibe-sdk
# or
yarn add @anthropic-ai/fibe-sdk
```

### Basic Usage

```typescript
import { api, exchange } from '@anthropic-ai/fibe-sdk';
import { Connection } from '@solana/web3.js';

// 1. Create API client for market data
const apiClient = api.createApiClient('https://api.fibe.com');

// Get all markets
const markets = await apiClient.getMarkets();
console.log(markets);

// Get mid prices
const mids = await apiClient.getAllMids();
console.log(mids);

// 2. Create Exchange client for trading
const connection = new Connection('https://api.mainnet-beta.solana.com');
const exchangeClient = new exchange.ExchangeClient(
  connection,
  'confirmed',
  apiClient
);
```

## Key Concepts

### Market Index

Each trading pair on Fibe is identified by a unique `marketIndex` (integer). Use the `/api/v1/markets` endpoint to get the list of all markets and their indices.

```typescript
const markets = await apiClient.getMarkets();
// [{ market: "SOL-USDC", marketIndex: 0, ... }, ...]
```

### Tick Size and Lot Size

- **Tick Size** (`tickSizeInQuoteBaseUnits`): The minimum price increment for orders
- **Lot Size** (`lotSizeInBaseBaseUnits`): The minimum quantity increment for orders

All order prices and quantities must be multiples of these values.

### Order ID

Orders are identified by a unique `orderId` (bigint). When placing orders, you must generate a unique order ID. A common pattern is to use the current timestamp:

```typescript
const orderId = BigInt(Date.now());
```

### Side

- `Side.Bid` (`"B"`) - Buy order
- `Side.Ask` (`"A"`) - Sell order

### Time in Force

- `TimeInForce.Gtc` - Good Till Cancelled (default for limit orders)
- `TimeInForce.Ioc` - Immediate Or Cancel (market orders)
- `TimeInForce.PostOnly` - Only add liquidity, cancel if would match
- `TimeInForce.Fok` - Fill Or Kill

## Response Format

### Success Response

All successful API responses return the data directly:

```json
{
  "market": "SOL-USDC",
  "marketIndex": 0,
  "baseMint": "So11111111111111111111111111111111111111112",
  "quoteMint": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"
}
```

### Error Response

Error responses include an `error` field:

```json
{
  "error": "Market not found"
}
```

## Rate Limits

| Endpoint Type | Rate Limit |
|---------------|------------|
| Public endpoints | 100 requests/minute per IP |
| WebSocket connections | 10 concurrent per IP |
| WebSocket subscriptions | 100 per connection |

## Next Steps

- [Info API Reference](api-reference.md) - Complete REST API documentation
- [Exchange API](exchange-api.md) - Trading operations documentation
- [TypeScript SDK](typescript-sdk.md) - Full SDK guide
- [WebSocket API](websocket.md) - Real-time data streaming
- [Examples](examples.md) - Code examples for common use cases
