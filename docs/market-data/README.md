# Market Data

## Overview

The market data layer transforms raw trade events from the matching engine into the data products that clients consume: candles, tickers, order book snapshots, and trade history. All processing is asynchronous and downstream from the matching engine — it has no effect on trading latency.

---

## Candles (OHLCV)

Candles are built from trade events in real-time as orders fill.

- Supported intervals: 1m, 5m, 15m, 30m, 1h, 4h, 6h, 12h, 1d, 1w
- Each candle is updated on every fill within its interval window
- The current (incomplete) candle is merged with the latest trade to reflect live price
- Candles are stored in MongoDB and queryable via REST API
- New candles are published to WebSocket subscribers as they update

---

## Ticker

The ticker provides a rolling 24-hour summary for each product:

| Field | Description |
|---|---|
| Last price | Price of the most recent trade |
| Open | First trade price in the last 24 hours |
| High | Highest trade price in the last 24 hours |
| Low | Lowest trade price in the last 24 hours |
| Volume | Total traded volume in the last 24 hours |
| Change | Price change percentage vs. 24h open |

The ticker updates on every fill and is pushed to all `ticker` WebSocket subscribers.

---

## Order Book (L2)

The order book shows all resting bid and ask orders grouped by price level.

- **Snapshot**: a full depth snapshot is taken periodically and stored; new WebSocket subscribers receive this snapshot immediately on subscribe.
- **Incremental updates**: every change to a price level (new order, cancel, fill) is published as an L2 update message.
- Clients reconstruct the live book by starting from the snapshot and applying updates sequentially.

The order book does not expose individual order IDs — only aggregated size per price level (same as Binance and Coinbase).

---

## Trade History

Every fill between two orders creates a trade record with:

- Timestamp
- Price
- Size
- Aggressor side (taker = buyer or seller who placed the market/crossing order)

Trade history is stored in MongoDB and queryable via the REST API with pagination. Recent trades are also streamed via WebSocket on the `matches` channel.

---

## Data Persistence

Market data is persisted asynchronously by dedicated background threads, one per data type:

| Thread | What it saves |
|---|---|
| `CandleMakerThread` | Builds and updates candles in MongoDB |
| `TickerThread` | Updates the 24h ticker |
| `OrderBookSnapshotThread` | Saves periodic L2 snapshots |
| `OrderPersistenceThread` | Saves order state changes |
| `TradePersistenceThread` | Records completed trades |
| `AccountPersistenceThread` | Records balance changes (bills/fills) |

Separate threads exist for futures and spot — they operate on isolated Kafka topics and MongoDB collections.

---

## Querying via REST API

| Endpoint | Description |
|---|---|
| `GET /api/products/{id}/candles` | Historical candles (interval + time range) |
| `GET /api/products/{id}/trades` | Recent trade history |
| `GET /api/products/{id}/ticker` | Current 24h ticker |
| `GET /api/products/{id}/book` | Current order book snapshot |
