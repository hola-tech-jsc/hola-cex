# Futures Trading

## Overview

The futures engine handles USDT-margined perpetual contracts — the same contract type used by Binance Futures, Bybit, and OKX. Users trade with leverage against a margin balance, and positions remain open indefinitely (no expiry) with periodic funding payments exchanged between longs and shorts.

---

## Opening a Position

1. User deposits USDT into their futures account.
2. User places a LIMIT or MARKET order with a chosen leverage (e.g., 10×).
3. The engine reserves the required initial margin from their balance.
4. When the order matches, a position is opened — entry price, size, and liquidation price are recorded.
5. The user sees unrealized PnL updating in real-time as the mark price moves.

---

## Order Types

| Type | Behavior |
|---|---|
| **LIMIT** | Rests on the order book at the specified price; fills when the market reaches it |
| **MARKET** | Fills immediately at the best available price |
| **Stop-Loss** | Triggers a market close when price moves against the position past a set level |
| **Take-Profit** | Triggers a market close when price reaches a profit target |

Time-in-force options: **GTC** (rest until filled or cancelled), **IOC** (fill immediately, cancel remainder), **Post-Only** (maker-only, rejected if it would match immediately), **Reduce-Only** (can only reduce an existing position, never open a new one).

---

## Leverage & Margin

- Leverage is configurable per position from 1× up to 125× (set per product by admin).
- **Initial margin** is reserved when the order is placed — the higher the leverage, the less margin required upfront.
- **Maintenance margin** is the minimum margin needed to keep a position open. If margin falls below this level due to adverse price movement, liquidation is triggered.
- Users can adjust their leverage after opening a position; the liquidation price recalculates immediately.

---

## Funding Rate

Perpetual contracts are kept close to the spot price through a periodic funding mechanism:

1. Every 8 hours, a funding rate is calculated based on the gap between the perpetual price and the mark price index.
2. If the rate is positive: longs pay shorts. If negative: shorts pay longs.
3. Payments are applied automatically across all open positions — no user action required.
4. Users receive a real-time WebSocket notification each time funding settles on their position.

This mechanism incentivizes arbitrage that keeps the perpetual price anchored to spot.

---

## Liquidation

When a position's margin balance falls below the maintenance threshold:

1. The engine detects the breach using the **mark price** (sourced from Binance/OKX/Bybit), not the last traded price — this prevents manipulation-triggered liquidations.
2. The position is force-closed at the bankruptcy price.
3. Any remaining margin after covering the loss is transferred to the Insurance Fund.
4. The user receives a liquidation notification and their position is closed with the realized PnL recorded.

---

## Insurance Fund

The Insurance Fund is a reserve that protects traders from socialized losses:

- It is funded by the residual margin from liquidated positions (when liquidation price > bankruptcy price).
- When a liquidation results in a loss larger than the position's margin (e.g., gap in a fast-moving market), the Insurance Fund covers the shortfall.
- If the fund cannot cover the loss, Auto-Deleveraging (ADL) activates.
- The fund balance is visible in real-time via the Grafana monitoring dashboard.

---

## Auto-Deleveraging (ADL)

ADL is the last-resort mechanism, activated only when the Insurance Fund is exhausted:

1. Profitable positions on the opposite side of the liquidated position are ranked by their PnL percentage.
2. The most profitable positions are partially closed to cover the deficit, at the liquidation price.
3. Affected users receive an ADL notification immediately via WebSocket.
4. This mirrors the same ADL model used by Binance and Bybit.

ADL is rare and only occurs in extreme market conditions.

---

## Mark Price

To prevent last-price manipulation from triggering unfair liquidations, the engine uses a **mark price** sourced from external exchanges (Binance, OKX, Bybit) rather than the internal last traded price. All liquidation checks and unrealized PnL calculations use the mark price.

---

## Self-Trade Prevention

Users cannot fill their own orders. Configurable behavior when a user's order would match their own resting order:

| Mode | Behavior |
|---|---|
| **Cancel Newest** | The incoming order is cancelled |
| **Cancel Oldest** | The resting order is cancelled |
| **Cancel Both** | Both orders are removed |
| **Allow** | No prevention (disabled by default) |
