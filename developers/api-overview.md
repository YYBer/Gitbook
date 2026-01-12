# API Overview

Welcome to the Fibe API documentation. Our API allows developers to integrate with Fibe's trading platform programmatically.

## Base URL

```
https://api.fibe.com/v1
```

## API Endpoints

Our API is organized around REST principles and returns JSON responses. All endpoints are documented using OpenAPI 3.1 specification.

### Available Endpoints

- **Market Data** - Get real-time market information, orderbook, and trading data
- **Trading** - Place and manage orders
- **Account** - Query account information and order history

## Quick Start

### 1. Install the TypeScript SDK

```bash
npm install @fibe/ts-sdk
```

### 2. Initialize the Client

```typescript
import { FibeAPI } from '@fibe/ts-sdk';

const api = new FibeAPI();
```

### 3. Make Your First Request

```typescript
// Get all markets
const markets = await api.getMarkets();
console.log(markets);

// Get mid prices
const mids = await api.getAllMids();
console.log(mids);
```

## Interactive API Reference

For a complete interactive API reference where you can test endpoints directly in your browser, see the [API Reference](api-reference.md) page.

## Response Format

All API responses follow a consistent format:

### Success Response
```json
{
  "data": { ... }
}
```

### Error Response
```json
{
  "error": "Error message"
}
```

## Rate Limits

API rate limits are applied per IP address. Current limits:
- Public endpoints: 100 requests per minute
- Authenticated endpoints: 200 requests per minute

## Support

If you have questions or need help:
- Check our [GitHub Repository](https://github.com/fibe-io/ts-sdk)
- Join our [Discord Community](#)
- Email us at api@fibe.com
