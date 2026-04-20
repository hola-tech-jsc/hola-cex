# REST API

## Overview

The REST API is the primary interface for trading clients, admin tools, and integrations. It covers futures trading, spot trading, account management, wallet operations, and exchange administration. Full interactive documentation is available via Swagger UI at `http://localhost/swagger-ui/index.html` after deployment.

Base URL: `http://localhost/api`

---

## Authentication

Most endpoints require authentication via a session token or API key:

- **Session token** — obtained from `POST /api/sign-in`; passed as a cookie or `Authorization` header
- **API key** — created via `POST /api/apps`; used for programmatic access (bots, integrations)

Public endpoints (market data, products, tickers) require no authentication.

---

## Rate Limiting

API requests are rate-limited per user and per IP. Exceeding the limit returns HTTP 429. The `RateLimitingInterceptor` enforces limits before requests reach controllers.

---

## Endpoint Groups

### Futures Trading

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/orders` | Place a futures limit or market order |
| GET | `/api/orders` | List open and historical orders |
| DELETE | `/api/orders/{id}` | Cancel an open order |
| POST | `/api/conditional-orders` | Place a stop-loss or take-profit order |
| DELETE | `/api/conditional-orders/{id}` | Cancel a conditional order |
| GET | `/api/positions` | List open positions |
| POST | `/api/positions/{id}/leverage` | Adjust leverage for a position |
| GET | `/api/accounts` | Futures account balance |

### Spot Trading

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/spot/orders` | Place a spot order |
| GET | `/api/spot/orders` | List spot orders |
| DELETE | `/api/spot/orders/{id}` | Cancel a spot order |
| GET | `/api/spot/accounts` | Spot account balances |
| GET | `/api/spot/products` | List configured spot trading pairs |

### Market Data (public)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/products` | List all futures products |
| GET | `/api/products/{id}/candles` | OHLCV candle history |
| GET | `/api/products/{id}/trades` | Recent trade history |
| GET | `/api/products/{id}/ticker` | 24h ticker |
| GET | `/api/products/{id}/book` | Order book snapshot |

### Wallet & Transfers

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/wallet` | Funding wallet balance |
| POST | `/api/wallet/transfer` | Transfer between funding wallet and trading account |
| GET | `/api/wallet/transactions` | Transaction history |

### User & Auth

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/sign-up` | Register a new user |
| POST | `/api/sign-in` | Authenticate and receive session token |
| POST | `/api/change-password` | Update password |
| GET | `/api/profile` | Get user profile |
| POST | `/api/apps` | Create an API key |
| GET | `/api/apps` | List API keys |
| DELETE | `/api/apps/{id}` | Revoke an API key |

### Admin

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/admin/deposit` | Credit a deposit to a user's account |
| POST | `/api/admin/withdraw` | Process a withdrawal |
| POST | `/api/admin/products` | Create or update a trading product |
| GET | `/api/admin/users` | List all users |
| POST | `/api/admin/users/{id}/freeze` | Freeze a user account |

---

## Error Responses

All errors follow a consistent format:

```json
{
  "code": "ORDER_NOT_FOUND",
  "message": "Order with id abc123 does not exist"
}
```

Common error codes are defined in `ErrorCode.java`. HTTP status codes follow standard conventions: 400 for validation errors, 401 for auth failures, 404 for not found, 429 for rate limit exceeded, 500 for internal errors.

---

## Pagination

List endpoints return paginated results:

```json
{
  "items": [...],
  "total": 142,
  "page": 1,
  "pageSize": 20
}
```

Use `?page=2&pageSize=50` query parameters to navigate pages.
