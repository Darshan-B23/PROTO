# Personal OS - Finance Module Architecture V1

## 1. Global Design System Reminder (Architectural Brutalism)
- **Finance Accent Color:** `Concrete Yellow (#c8a97e)`.
- **Layout:** Strict, high-density financial ledgers. No pie charts, no 3D graphs, no spending categorizations with colorful bubbles. Use minimalist, monospace-aligned tables for currency to mimic a physical accounting book.
- **Rules:** Data before decoration. Structure before color. Function before customization.

---

## 2. Finance Module Requirements (Core Architecture)

### Functional Specs:
1. **Ledger & Transactions:** A simple income and expense tracker categorized by tags (e.g., hardware, software, food, rent, freelance).
2. **Account Balances:** Supports multiple accounts (e.g., "Main Checking", "Cash") but balances are calculated dynamically from transactions to prevent data drift.
3. **Subscription Manager:** A dedicated tracker for recurring costs (e.g., cloud hosting, ChatGPT Plus, gym) that automatically calculates your monthly burn rate.

---

## 3. SQLite Database Schema

```sql
CREATE TABLE accounts (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,           -- e.g., "Main Checking", "Cash"
    initial_balance REAL DEFAULT 0.0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE transactions (
    id TEXT PRIMARY KEY,
    account_id TEXT REFERENCES accounts(id) ON DELETE CASCADE,
    amount REAL NOT NULL,
    type TEXT CHECK(type IN ('income', 'expense')) NOT NULL,
    category TEXT NOT NULL,       -- e.g., 'hardware', 'food', 'subscriptions'
    description TEXT,
    transaction_date DATE NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE subscriptions (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,           -- e.g., "DigitalOcean Droplet"
    amount REAL NOT NULL,
    billing_cycle TEXT CHECK(billing_cycle IN ('monthly', 'yearly')) DEFAULT 'monthly',
    next_billing_date DATE,
    status TEXT CHECK(status IN ('active', 'cancelled')) DEFAULT 'active'
);
```

---

## 4. UI Layout Specifications

### A. Finance Ledger View
- **Max-width:** 900px.
- **Top Section:** Financial Summary (Total Balance across accounts + Subscriptions Burn Rate).
- **Middle Section:** The Brutalist Ledger (Transactions).
- **Bottom Section:** Active Subscriptions.

```text
  FINANCE LEDGER
  +-------------------------------------------------------------------------------+
  | CURRENT BALANCE: $1,250.00                   MONTHLY BURN RATE: $145.00/mo    |
  |                                                                               |
  | [PaperCard] Recent Transactions                            [+ Add Entry]      |
  | OCT 09   [expense]   Food          Groceries                    -$  45.00     |
  | OCT 05   [expense]   Hardware      Raspberry Pi 5               -$  80.00     |
  | OCT 01   [income]    Freelance     Web API Setup                +$ 300.00     |
  +-------------------------------------------------------------------------------+

  ACTIVE SUBSCRIPTIONS
  +-------------------------------------------------------------------------------+
  | [PaperCard]                                          [+ Add Subscription]     |
  | - GitHub Copilot          $10.00/mo       Next Bill: Oct 15                   |
  | - Spotify Premium         $10.99/mo       Next Bill: Oct 22                   |
  +-------------------------------------------------------------------------------+
```

---

## 5. React Component Guidance for the AI
- **Currency Formatting:** Use JavaScript's `Intl.NumberFormat` with a monospace font (`font-mono`) specifically for the amount columns to ensure the decimal points align perfectly vertically.
- **Dynamic Balance Calculation:** Do not store the current balance in the `accounts` table. Instead, calculate it on the backend by summing `initial_balance` + `income` - `expenses`.
- **Burn Rate Math:** For the monthly burn rate, sum all `monthly` subscriptions, add (`yearly` subscriptions / 12), and display this number prominently at the top of the dashboard.