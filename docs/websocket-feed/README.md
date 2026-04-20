# WebSocket Real-Time Feed

## Overview

The WebSocket feed delivers live market data, order status updates, and account events to connected clients. It powers trading UIs, bots, and integrations without polling the REST API.

---

## Connecting

Clients connect to `ws://localhost/ws`. Private channels (orders, positions, account balances) require authentication via session token passed during the WebSocket handshake.

---

## Subscribing to Channels

After connecting, clients send a subscribe message specifying which channels and products they want:

```json
{
  "type": "subscribe",
  "channels": [
    { "name": "ticker", "product_ids": ["BTC-USDT"] },
    { "name": "level2", "product_ids": ["BTC-USDT"] },
    { "name": "user",   "product_ids": ["BTC-USDT"] }
  ]
}
```

Clients can subscribe and unsubscribe at any time. Multiple products can be subscribed in a single message.

---

## Message Types

### Order Updates (private — requires auth)

| Event | When it fires |
|---|---|
| `received` | The exchange has acknowledged the order |
| `open` | The order is now live on the order book |
| `match` | The order was partially or fully filled |
| `done` | The order is no longer on the book (fully filled or cancelled) |

A typical LIMIT order lifecycle: `received` → `open` → `match` (one or more) → `done`.
A MARKET order that fills fully: `received` → `match` → `done` (no `open`).

---

### Position Updates (private — requires auth)

| Event | When it fires |
|---|---|
| `position_open` | A new futures position has been opened |
| `position_update` | Position size, margin, or leverage changed |
| `position_close` | Position fully closed (manually or by Take-Profit) |
| `liquidation` | Position was force-liquidated |

---

### Account Events (private — requires auth)

| Event | When it fires |
|---|---|
| `funds` | Account balance changed (deposit, withdrawal, fill, fee) |
| `funding` | Funding rate settlement applied to a position |
| `adl` | A position was partially closed due to Auto-Deleveraging |

---

### Market Data (public)

| Channel | What it delivers |
|---|---|
| `ticker` | Latest price, 24h open/high/low/volume, updated on every trade |
| `level2` | Full order book depth snapshot on subscribe, then incremental updates |
| `candle` | OHLCV candle data for configured intervals (1m, 5m, 15m, 1h, 4h, 1d) |
| `mark_price` | Mark price updates sourced from Binance/OKX/Bybit (futures only) |

---

### Order Book (L2) Feed

The order book channel uses a snapshot + diff pattern to minimize bandwidth:

1. On subscribe, the client receives a **full snapshot** of all price levels with their sizes.
2. After that, only **changed levels** are sent — if a price level's size drops to zero, it means that level was removed.
3. Clients apply the updates to their local book to keep it in sync.

This pattern is the same as used by Coinbase Advanced Trade and other major exchanges.

---

### System Events

| Event | Description |
|---|---|
| `heartbeat` | Sent periodically with a sequence number so clients can detect gaps |
| `pong` | Response to a client `ping` message |

---

## Channel Summary

| Channel | Auth required | Description |
|---|:---:|---|
| `ticker` | No | Last price and 24h stats |
| `level2` | No | Order book depth (snapshot + incremental) |
| `candle` | No | OHLCV candle stream |
| `mark_price` | No | Mark price feed (futures) |
| `heartbeat` | No | Sequence keepalive |
| `user` | Yes | Orders, positions, account balance, funding, ADL |
