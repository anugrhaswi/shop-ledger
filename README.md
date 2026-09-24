<div align="center">

# 🧾 Shop Ledger

**Simple, self-hosted bookkeeping for small shops and freelancers.**

Track money across accounts, debts, receivables, and daily profit — all on your own machine, no cloud, no spreadsheets.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-%E2%89%A53.0-black.svg)](https://flask.palletsprojects.com/)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-%E2%89%A52.0-red.svg)](https://www.sqlalchemy.org/)
[![Made for small shops](https://img.shields.io/badge/Made%20for-small%20shops-yellow.svg)](README.md)

</div>

---

## Why this exists

A shop owner juggles a cash drawer, a bank account, a wallet, money lent to regulars, and money owed to suppliers. Spreadsheets get messy, and online accounting tools mean your financial data lives in someone else's cloud.

**Shop Ledger is the middle ground:** a single-page dashboard on your own computer that answers the questions you actually ask every day.

## What it solves

| Problem | Solution |
|---|---|
| Money spread across multiple accounts | Per-account ledgers with a **Total Balance** rollup, plus **transfers** that move money without touching P&L |
| "Am I actually making money?" | Clean **profit = income − expenses** math. Loans and repayments are financing activity, so they never distort your P&L |
| Money you owe / money owed to you | Full **Debt** and **Receivable** lifecycles, from recording to receive/settle — balances update automatically |
| Manual profit tracking per day | One-tap **daily profit log** on the dashboard, with a running total at `/reports/profits` |
| Messing up | Every transaction can be **edited or undone**, and the balance changes are reversed for you |
| Losing data | Automatic **SQLite backup** on every start + corruption recovery from the last good copy |
| Accounts, debts, categories | **Edit anything** after the fact, with safeguards (e.g. can't change a debt amount after the loan was deposited) |

## What it gives you at a glance

- 💰 **Dashboard KPIs** — Total Balance, Fixed Capital, Profit, Outstanding Debt, Outstanding Receivable, Net Worth
- 🏦 **Multiple accounts** — bank, wallet, cash, UPI, anything — with individual balances
- 🔀 **Transfers** — move money between accounts as a paired debit/credit that P&L ignores
- ➕ **Four transaction types** — `credit` (income), `debit` (expense), `loan`, `repayment`
- 🗂️ **Categories** — income/expense labels with 12 sensible defaults seeded for you
- 🧾 **P&L reports** — daily breakdown and monthly overview, filterable by month/year
- 📒 **Daily profit log** — record profit manually each day, view history with running total
- 💳 **Debts** — borrow money, receive the loan into an account, settle it when paid
- 💵 **Receivables** — lend money, receive payment, watch your net worth update
- 📜 **Full log viewer** — every transaction, filterable by account and type
- 💾 **One-click backup** — download the whole database at `/backup`

---

## Tech stack

- **Backend** — [Flask](https://flask.palletsprojects.com/) (WSGI) with SQLAlchemy ORM 2.0
- **Database** — SQLite, single file at `~/accounting.db`
- **Templates** — Jinja2 + [Bootstrap 5](https://getbootstrap.com/)
- **Money** — stored as `Float`, displayed in ₹ (Indian Rupees)

---

## Getting started

### Requirements

- Python 3.8+
- `pip` (or Conda — the author uses the `accounts` env)

### Install & run

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Run it
python main.py

# 3. Open it
#    http://127.0.0.1:5000/
```

> Your data lives in `~/accounting.db` — created on first run, next to dated backups in `~/backups/`.

### First steps

1. **Add your accounts** (`Add Account`) — e.g. *Savings Bank*, *Cash Drawer*, *Wallet*.
2. **Set your fixed capital** — click *Edit* on the Fixed Capital card. This is your starting money, and profit is shown as `Total Balance − Fixed Capital`.
3. **Record transactions** (`Add Transaction`) — pick an account, choose `credit`/`debit`, and describe it.
4. **Log today's profit** — the *Log Profit* button on the dashboard Profit card.

---

## Usage tour

| Page | What it does |
|---|---|
| **Dashboard** `/` | KPIs, account balances with quick "+ Add", the 10 most recent transactions with Edit/Undo, inline capital & profit forms |
| **Add Account** `/accounts/new` | Create an account (name, type, optional description) |
| **Add Transaction** `/transactions/new` | Record a credit/debit/loan/repayment against any account |
| **Transfer** `/transactions/transfer` | Move money between two accounts as a paired transfer |
| **Categories** `/categories/` | Manage income/expense labels; delete is blocked while in use |
| **Debts** `/debts/` | Record what you owe. *Receive* = loan deposited into an account. *Pay* = settles it |
| **Receivables** `/receivables/` | Record who owes you. *Receive* = payment received |
| **Reports** `/reports/` | Daily and monthly P&L with a month/year picker, plus overall summary |
| **Daily Profit** `/reports/profits` | Your manually logged daily profit, with running totals |
| **Logs** `/logs` | Every transaction, filterable by account and type |

---

## How it works

### The four transaction types

| Type | Direction | P&L? | Example |
|---|---|---|---|
| `credit` | **Increases** account balance | ✅ Income | Sold goods, commission received |
| `debit` | **Decreases** account balance | ✅ Expense | Rent, electricity, supplies |
| `loan` | **Increases** balance | ❌ | Borrowed money deposited into your account |
| `repayment` | **Decreases** balance | ❌ | Paying back borrowed money |

Loans and repayments are financing activities — they move cash between your balance sheet and a lender's, but they aren't income or expenses. That's why only credits and debits show up in your P&L.

### Account balances are automatic

Every `create`, `edit`, `undo`, transfer, debt settlement, and receivable receipt increments/decrements the account balance for you. You never edit a balance by hand — P&L and net worth always reconcile with the transactions.

### Linked transactions

Some transactions carry a `reference` that links them to a debt or receivable, or to their transfer pair:

- `debt:<id>` — loan received from / repayment made for a debt
- `receivable:<id>` — payment received for a receivable
- `xfer:<id>` — the paired debit + credit of a transfer

Undoing a linked transaction **reverts the debt/receivable state too** — e.g. deleting a repayment marks a debt unpaid again. Deleting one side of a transfer removes both. Linked fields can't be edited in ways that would break the chain.

### Startup lifecycle

On every launch, before the web server starts:

1. **`restore_db()`** — runs an integrity check; if the DB is corrupted it's quarantined and the last good `.bak` is restored
2. **`create_tables()`** — creates tables and applies legacy migrations (safe to re-run)
3. **`backup_db()`** — copies the DB to a dated file in `~/backups/` (keeps the 3 newest) and refreshes `.bak`
4. **`seed_data()`** — seeds the 12 default categories if none exist

### Data model

```
accounts ──< transactions >── categories
debts           (loan / repayment / credit / debit)
receivables     (linked by reference)
daily_profit_log
settings        (key-value: fixed_capital, …)
```

- **Debts**: `unpaid` → loan received (`received_at`) → `paid` (`settled_at`)
- **Receivables**: `unreceived` → `received` (`received_at`)
- Profit on the dashboard: `Total Balance − Fixed Capital`
- Net worth: `Total Balance + Outstanding Receivables − Outstanding Debts`

---

## Backup & restore

- **Automatic** — every startup copies your DB to `~/backups/accounting_YYYY-MM-DD.db` (3 kept) and a `~/.bak`.
- **Manual** — visit `/backup` to download the database file as `accounting_backup.db`.
- **Restore** — stop the app, replace `~/accounting.db` with a backup, and start it again. If the file is ever corrupted, the app restores the `.bak` itself.

---

## Deployment

The app is standard WSGI Flask — run it anywhere Python runs:

```bash
# Dev
python main.py

# Production
gunicorn -w 2 -b 0.0.0.0:8000 main:app
```

The database lives on the machine running the app (SQLite). For a single shop on one computer, that's all you need — and the machine has no internet dependency.

---

## Project structure

```
main.py                        # Entry point + startup lifecycle + core routes
crud.py                        # All database operations (response: single source of truth)
models.py                      # SQLAlchemy ORM models
schemas.py                     # Form-validation dataclasses
database.py                    # Engine, session factory, per-request session
routers/
  accounts.py                  # Account CRUD
  categories.py                # Category management
  transactions.py              # Transactions + transfers + edit/undo
  debts.py                     # Debt lifecycles (record, receive, settle)
  receivables.py               # Receivable lifecycles (record, receive)
  reports.py                   # P&L + daily profit log
templates/                     # Jinja2 + Bootstrap 5 views
requirements.txt
LICENSE
```

---

## Roadmap & known limitations

- **Single-user** — no authentication; keep it on a trusted machine or behind a VPN
- **No multi-currency** — amounts are plain ₹ floats
- **Floats for money** — intentional trade-off for simplicity; sufficient for a shop ledger, but not ledger-grade precision for large sums
- **Profit is manual** — the daily profit log is a manual entry, not auto-calculated (P&L reports are the automatic view)

---

## License

Released under the [MIT License](LICENSE) — Copyright © 2026 [Anugrha Bhujel](https://github.com/anugrhaswi). Use it, change it, sell it — just keep the license notice and please don't hold the author liable. 🙏