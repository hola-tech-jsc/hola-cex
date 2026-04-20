# Spot Trading

## Overview

The spot trading engine is a fully independent order book running in parallel with the futures engine. Users buy and sell assets directly at market or limit prices — no leverage, no margin, no liquidation. Balances are held in both the base asset (e.g., BTC) and the quote asset (e.g., USDT).

---

## Placing an Order

1. User funds their spot account with the base or quote asset via a deposit.
2. User submits a buy or sell order with a price (LIMIT) or without (MARKET).
3. The engine checks available balance and reserves the required funds.
4. If a matching order exists on the opposite side, the trade executes immediately.
5. Remaining unfilled quantity rests on the order book until matched or cancelled.

---

## Order Types

| Type | Behavior |
|---|---|
| **LIMIT** | Rests on the book at the specified price |
| **MARKET** | Fills immediately at the best available price on the opposite side |
| **Post-Only** | Cancelled if it would match immediately (ensures maker rebate) |
| **IOC** (Immediate-Or-Cancel) | Fills whatever is available instantly, cancels the rest |
| **GTC** (Good-Till-Cancelled) | Stays on the book until fully filled or manually cancelled |

---

## Balance & Settlement

- When a buy order is placed, the quote asset (e.g., USDT) is reserved.
- When a sell order is placed, the base asset (e.g., BTC) is reserved.
- On fill, the reserved amount is deducted and the received asset is credited instantly.
- Partial fills update the reserved balance incrementally.

---

## Market Data

The spot engine generates the same market data feeds as the futures engine:

- **Order book (L2)** — real-time depth at all price levels, updated on every change
- **Trades** — each fill is published as a trade event with price, size, and side
- **Candles (OHLCV)** — candle data aggregated across all standard intervals (1m, 5m, 1h, 1d, etc.)
- **Ticker** — 24-hour rolling statistics: last price, open, high, low, volume

All market data is streamed to clients via WebSocket.

---

## Differences from Futures

| Aspect | Spot | Perpetual Futures |
|---|---|---|
| Leverage | No | Yes (up to 125×) |
| Liquidation | No | Yes |
| Funding rate | No | Yes (every 8h) |
| Insurance Fund | No | Yes |
| Position tracking | No | Yes |
| Asset held | Base + quote asset | USDT margin |
| Settlement | Immediate on fill | Unrealized until closed |

---

## Supported Trading Pairs

Any trading pair can be configured by an admin (e.g., BTC/USDT, ETH/USDT, SOL/USDT). Each pair has its own independent order book, candle history, and ticker.
