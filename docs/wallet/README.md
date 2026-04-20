# Wallet & Transfers

## Overview

The wallet system manages user funds across two account types: the **funding wallet** (for deposits and withdrawals) and the **trading accounts** (futures margin and spot balances). Users move funds between these accounts via internal transfers.

---

## Account Structure

```
User
├── Funding Wallet       ← external deposits/withdrawals land here
├── Futures Account      ← USDT margin for perpetual trading
└── Spot Account         ← base + quote asset for spot trading
```

Funds in the funding wallet are not at risk from trading — they must be explicitly transferred into a trading account before they can be used.

---

## Deposit Flow

1. An external system (blockchain node, custodian, or admin API) detects an incoming deposit.
2. The deposit is submitted via the Admin API (`POST /api/deposit`).
3. Funds are credited to the user's **funding wallet**.
4. User transfers funds to their futures or spot account to start trading.

The engine itself is chain-agnostic — it accepts deposits via a `DepositCommand` interface and does not integrate directly with any blockchain node.

---

## Withdrawal Flow

1. User requests a withdrawal from the REST API.
2. The system checks available balance in the funding wallet.
3. A withdrawal record is created with `PENDING` status.
4. The `TransferOutboxThread` picks up pending withdrawals and submits them to the external custodian or blockchain integration.
5. On confirmation, the record is marked `COMPLETED` and the balance is finalized.

Withdrawals are processed asynchronously — the user's balance is reserved immediately on request and released only if the withdrawal fails.

---

## Internal Transfers

Users can move funds between their funding wallet and trading accounts at any time:

| Transfer | Direction |
|---|---|
| Deposit to futures | Funding Wallet → Futures Account |
| Withdraw from futures | Futures Account → Funding Wallet |
| Deposit to spot | Funding Wallet → Spot Account |
| Withdraw from spot | Spot Account → Funding Wallet |

Transfers are instant and atomic — funds leave one account and arrive in the other in the same operation.

---

## Balance Safety

- Funds reserved for open orders or positions cannot be transferred or withdrawn.
- The funding wallet is separate from trading accounts, so a losing trade cannot affect funds not allocated to that account.
- All balance changes are recorded as `Bill` entries for a full audit trail.

---

## Admin Controls

Exchange administrators can:
- Credit deposits to any user account (`POST /api/admin/deposit`)
- Process withdrawals manually
- View all pending transfer records
- Freeze or adjust user balances (for compliance or dispute resolution)
