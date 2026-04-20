# Event Sourcing & State Recovery

## Overview

The matching engine uses an event-sourced architecture: instead of storing only the current state, every action (command) that changes state is logged permanently. The current state is always derivable by replaying that log from the beginning — or from a recent snapshot to skip ahead.

This guarantees that no trade, fill, or balance change is ever lost, even across server restarts or crashes.

---

## How a Trade Flows Through the System

1. **User places an order** via the REST API.
2. The API publishes a **command** (e.g., "place this order") to the Kafka command log.
3. The **matching engine** consumes the command, updates its in-memory order book and account state, and publishes **events** (e.g., "order was filled", "position opened", "balance changed").
4. Downstream workers consume those events in parallel:
   - **WebSocket feed** — pushes real-time updates to connected clients
   - **Market data workers** — build candles, update ticker, snapshot order book
   - **Persistence workers** — save order, trade, and position records to MongoDB

The matching engine itself never touches a database on the critical path — all persistence is asynchronous and downstream.

---

## Why This Matters

| Property | Benefit |
|---|---|
| **Auditability** | Every state change has a corresponding command in the log |
| **Crash recovery** | Restart from the latest snapshot and replay recent commands — no data loss |
| **Determinism** | The same sequence of commands always produces the same state |
| **Decoupling** | WebSocket, persistence, and analytics workers are independent consumers — adding a new consumer doesn't affect the engine |

---

## Snapshots

Replaying the entire command log from the beginning would be slow on restart. Instead, the engine periodically takes a **snapshot** — a full dump of the in-memory state (all orders, positions, accounts, products) saved to MongoDB, along with the Kafka offset at that point.

On restart:
1. Load the latest snapshot → restore state in seconds.
2. Resume consuming commands from the snapshot's offset → replay only recent history.

Snapshots are taken automatically in the background at a configurable interval. No operator action required.

---

## Command Log Retention

Kafka retains all commands for a configurable retention period (default: 7 days). This means:
- Full replay is possible within the retention window.
- Snapshots are the primary recovery mechanism for normal restarts.
- For disaster recovery, the combination of the latest snapshot + Kafka log covers all recent activity.

---

## Ordering Guarantee

Commands are partitioned by `product_id` in Kafka. This means:
- All commands for BTC-USDT are processed in strict order, one at a time.
- Commands for different products can be processed in parallel.
- No command for a given product is ever processed out of order.

---

## What Happens on a Restart

```
Startup
  │
  ├─ Load latest snapshot from MongoDB
  │    └─ Restores: order book, positions, accounts, products
  │
  ├─ Resume Kafka consumption from snapshot offset
  │    └─ Replays all commands issued since the snapshot
  │
  └─ Engine is live — accepting new commands
```

The window between the last snapshot and the restart is typically seconds to minutes. All commands in that window are in Kafka and will be replayed automatically.
