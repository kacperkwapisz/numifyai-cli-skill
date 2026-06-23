---
name: numifyai-cli
description: >
  Operate the Numify AI CLI to manage Polish sp. z o.o. bookkeeping from the
  terminal. Use when the user asks to create transactions, manage companies,
  view reports, handle VAT, manage journal entries, or interact with Numify AI
  infrastructure. Triggers on: numify, numifyai, bookkeeping CLI, transactions,
  journal entries, VAT register, trial balance, Polish accounting.
metadata:
  author: numifyai
  version: "1.2"
---

# Numify AI CLI

Numify AI is a bookkeeping tool for Polish sp. z o.o. companies. The `numify` CLI manages companies, transactions, journal entries, VAT, reports, and more from the terminal. Every command supports `--json` for agent and CI consumption.

## Install

```bash
npm i -g numifyai
```

Requires Node.js 22+. Current CLI version: **0.1.3**.

## Authentication

Three methods, in precedence order:

1. **`--token <nfy_live_…>` flag** — overrides everything, never persisted.
2. **`NUMIFY_TOKEN` env var** — for agents and CI. Stateless: nothing written to disk.
3. **`numify login`** — browser device flow. Human-only — does not work in non-interactive mode.

Create API keys at **Dashboard → CLI → API keys**. The key is shown once.

**For agents: always use `NUMIFY_TOKEN` or `--token`.** Do not attempt `numify login` in non-interactive environments.

## Agent mode — always use `--json`

Pass `--json` to every command. Output is a stable versioned envelope on stdout:

```json
{
  "schemaVersion": 1,
  "ok": true,
  "command": "transactions.list",
  "requestId": "abc-123",
  "data": { },
  "meta": {
    "apiUrl": "https://numify.ai",
    "authSource": "env",
    "durationMs": 142,
    "cliVersion": "0.1.3"
  }
}
```

On failure:

```json
{
  "schemaVersion": 1,
  "ok": false,
  "command": "transactions.create",
  "requestId": "abc-123",
  "error": {
    "code": "NOT_AUTHENTICATED",
    "message": "Run `numify login` first.",
    "hint": { "action": "login" }
  },
  "meta": { }
}
```

Always check `ok` first. On failure, read `error.code` (closed enum) and `error.hint` for recovery actions.

## Deferred execution

Write commands (create, edit, archive) are **deferred by default** — the action is queued and executes after a delay. This provides an audit-safe undo window.

- Default: action queued, executes after delay. Cancel via `numify pending cancel <id>`.
- `--immediate`: typeback confirmation in TTY, then executes immediately.
- Deferred responses include `actionId`, `executesAt`, and `cancelUrl`.

## Global flags

Every subcommand inherits these:

| Flag | Description |
|---|---|
| `--json` | Output as JSON envelope. |
| `--output <human\|json\|ndjson>` | Choose output format. |
| `--verbose` | Verbose output. |
| `--api-url <url>` | Override API base URL. |
| `--token <nfy_live_…>` | API key for auth. |
| `--debug` | Dump HTTP traffic to stderr (secrets redacted). |
| `--immediate` | Execute write commands immediately (skip deferred queue). |

## Commands

### Auth

| Command | Description |
|---|---|
| `numify login` | Browser device flow (human-only). `--token <nfy_live_…>` to paste a key instead. |
| `numify logout` | Sign out. |
| `numify whoami` | Show current user. |
| `numify status` | Auth, API reachability, version. |
| `numify health` | API health check. |

### Companies

| Command | Description |
|---|---|
| `numify companies list` | List companies you have access to. |
| `numify companies get <id>` | Show full company details. |
| `numify companies edit <id>` | Edit company settings (deferred). |

#### `companies edit` flags

All optional. Pass at least one:

| Flag | API field | Description |
|---|---|---|
| `--name` | `name` | Company name |
| `--short-name` | `shortName` | Short name (without legal form suffix) |
| `--tax-id` | `taxId` | NIP (10 digits, validated) |
| `--krs` | `krs` | KRS (10 digits, validated) |
| `--regon` | `regon` | REGON number |
| `--legal-form` | `legalForm` | `sp_z_oo` \| `spolka_jawna` \| `jdg` \| `spolka_akcyjna` \| `other` |
| `--address-street` | `addressStreet` | Street address |
| `--address-city` | `addressCity` | City |
| `--address-zip` | `addressZip` | Postal code |
| `--address-country` | `addressCountry` | Country code (e.g. `PL`) |
| `--vat-status` | `vatStatus` | `czynny_vat` \| `zwolniony` \| `niezarejestrowany` |
| `--cit-rate` | `citRate` | `standard_19` \| `small_9` \| `estonian` |
| `--vat-filing-frequency` | `vatFilingFrequency` | `monthly` \| `quarterly` |
| `--fiscal-year-start` | `fiscalYearStartMonth` | Month number 1-12 |
| `--currency` | `currency` | `PLN` \| `EUR` \| `USD` |
| `--bank-account` | `bankAccountNumber` | Bank account number |
| `--bank-name` | `bankName` | Bank name |
| `--tax-office-code` | `taxOfficeCode` | 4-digit tax office code (TKodUS for JPK) |
| `--tax-office-name` | `taxOfficeName` | Tax office name |
| `--registration-date` | `registrationDate` | KRS registration date (`YYYY-MM-DD`) |

#### `companies get` output fields

Returns: Name, Short Name, NIP, KRS, REGON, Legal Form, Currency, VAT Status, VAT Filing, CIT Rate, Fiscal Year Start, Address (street, zip, city, country), Tax Office (name + code), Bank (name + account), Registration Date, ID.

### Transactions

| Command | Key flags | Description |
|---|---|---|
| `numify transactions list` | `--company` (required), `--type`, `--start-date`, `--end-date`, `--search` | List transactions. |
| `numify transactions get <id>` | `--company` | Show transaction details. |
| `numify transactions create` | `--company`, `--immediate` | Create a transaction (deferred). |
| `numify transactions edit <id>` | `--immediate` | Edit a transaction (deferred). |
| `numify transactions correct <id>` | `--company`, `--immediate` | Create correction entry. |
| `numify transactions archive <id>` | `--company`, `--immediate` | Archive a transaction. |
| `numify transactions verify <id>` | `--company` | Verify/approve a transaction. |
| `numify transactions reject <id>` | `--company` | Reject a transaction. |

#### `transactions create` flags

| Flag | Required | Description |
|---|---|---|
| `--company` | ✅ | Company ID |
| `--type` | ✅ | `income` \| `expense` \| `transfer` |
| `--description` | ✅ | Transaction description |
| `--amount-net` | ✅ | Net amount in grosze (integer) |
| `--amount-vat` | | VAT amount in grosze |
| `--amount-gross` | | Gross amount in grosze |
| `--vat-rate` | | VAT rate (e.g. `23`) |
| `--currency` | | Currency (default: `PLN`) |
| `--counterparty` | | Counterparty name |
| `--counterparty-tax-id` | | Counterparty NIP |
| `--invoice-number` | | Invoice number |
| `--invoice-date` | | Invoice date (`YYYY-MM-DD`) |
| `--due-date` | | Due date (`YYYY-MM-DD`) |
| `--category` | | Category |
| `--payment-method` | | Payment method |
| `--is-paid` | | Mark as paid (boolean) |

#### `transactions edit` flags

Optional: `--description`, `--amount-net`, `--amount-vat`, `--amount-gross`, `--category`, `--invoice-number`, `--invoice-date`, `--is-paid`.

### Journal entries

| Command | Key flags | Description |
|---|---|---|
| `numify journal list` | `--company` | List journal entries. |
| `numify journal get <id>` | `--company` | Show journal entry details. |
| `numify journal create` | `--company`, `--immediate` | Create a journal entry (deferred). |
| `numify journal post <id>` | `--company` | Post a draft entry. |
| `numify journal storno <id>` | `--company`, `--immediate` | Storno (reverse) an entry. |

#### `journal create` flags

| Flag | Required | Description |
|---|---|---|
| `--company` | ✅ | Company ID |
| `--description` | ✅ | Entry description |
| `--entries` | ✅ | JSON array: `[{"accountId": "...", "debit": 10000}, {"accountId": "...", "credit": 10000}]` |
| `--date` | | Entry date (`YYYY-MM-DD`) |

**Double-entry rule:** total debits must equal total credits. Amounts in grosze.

### Chart of accounts

| Command | Description |
|---|---|
| `numify accounts list --company <id>` | List accounts in chart of accounts. |
| `numify accounts get <id> --company <id>` | Show account details. |

### VAT

| Command | Key flags | Description |
|---|---|---|
| `numify vat list` | `--company`, `--period` | List VAT register entries. |
| `numify vat create` | `--company`, `--transaction-id` (required), `--immediate` | Create VAT entry from transaction. |
| `numify vat edit <id>` | `--company`, `--immediate` | Edit VAT entry. |
| `numify vat summary` | `--company`, `--period` | VAT summary for period. |

#### `vat edit` flags

Optional: `--net-amount` (grosze), `--vat-amount` (grosze), `--document-number`.

### Contractors

| Command | Key flags | Description |
|---|---|---|
| `numify contractors list` | `--company` | List contractors. |
| `numify contractors get <id>` | `--company` | Show contractor details. |
| `numify contractors create` | `--company`, `--immediate` | Create a contractor (deferred). |
| `numify contractors edit <id>` | `--company`, `--immediate` | Edit a contractor (deferred). |

#### `contractors create` flags

| Flag | Required | Description |
|---|---|---|
| `--company` | ✅ | Company ID |
| `--name` | ✅ | Contractor name |
| `--tax-id` | | NIP |
| `--type` | | `supplier` \| `customer` \| `both` |
| `--country` | | Country code |
| `--city` | | City |
| `--street` | | Street |
| `--zip` | | Zip code |
| `--bank-account` | | Bank account |
| `--email` | | Email |
| `--phone` | | Phone |

#### `contractors edit` flags

Optional: `--name`, `--tax-id`, `--type`, `--city`, `--street`, `--email`, `--is-active` (boolean).

### Documents

| Command | Key flags | Description |
|---|---|---|
| `numify documents list` | `--company` | List documents/invoices. |
| `numify documents get <id>` | `--company` | Show document details. |
| `numify documents upload` | `--company`, `--file <path>` | Upload a document. |

### Ledger

| Command | Description |
|---|---|
| `numify ledger list --company <id>` | List general ledger entries. |
| `numify ledger get <id> --company <id>` | Show ledger entry details. |

### Reports

| Command | Key flags | Description |
|---|---|---|
| `numify reports trial-balance` | `--company` (required), `--year`, `--month` | Trial balance report. |
| `numify reports balance-sheet` | `--company`, `--period` | Balance sheet (Bilans). |
| `numify reports profit-loss` | `--company`, `--period` | Profit & Loss (RZiS). |
| `numify reports cit` | `--company`, `--year` | CIT tax report. |

### Bank accounts

| Command | Key flags | Description |
|---|---|---|
| `numify bank list` | `--company` | List bank accounts. |
| `numify bank create` | `--company`, `--immediate` | Create bank account (deferred). |
| `numify bank edit <id>` | `--immediate` | Edit bank account (deferred). |
| `numify bank lines` | `--company`, `--account` | List bank statement lines. |
| `numify bank import <bank-account-id> <file>` | | Import bank statement (CSV/MT940). |
| `numify bank auto-match` | `--company` | Auto-match bank lines to transactions. |

#### `bank create` flags

| Flag | Required | Description |
|---|---|---|
| `--company` | ✅ | Company ID |
| `--account-number` | ✅ | Account number |
| `--bank-name` | ✅ | Bank name |
| `--currency` | | Currency (default: `PLN`) |

#### `bank edit` flags

Optional: `--account-number`, `--bank-name`.

### Fixed assets

| Command | Key flags | Description |
|---|---|---|
| `numify assets list` | `--company` | List fixed assets. |
| `numify assets get <id>` | `--company` | Show asset details. |
| `numify assets create` | `--company`, `--immediate` | Register a fixed asset (deferred). |
| `numify assets edit <id>` | `--immediate` | Edit an asset (deferred). |
| `numify assets dispose <id>` | `--company`, `--immediate` | Dispose/retire an asset. |
| `numify assets depreciation <id>` | `--company` | Show depreciation schedule. |
| `numify assets generate-depreciation` | `--company`, `--immediate` | Generate depreciation entries. |
| `numify assets export` | `--company` | Export asset register. |

#### `assets create` flags

| Flag | Required | Description |
|---|---|---|
| `--company` | ✅ | Company ID |
| `--name` | ✅ | Asset name |
| `--initial-value` | ✅ | Initial value in grosze |
| `--depreciation-method` | | `linear` \| `degressive` \| `one_time` |
| `--depreciation-rate` | | Annual depreciation rate (%) |
| `--acquisition-date` | | Acquisition date (`YYYY-MM-DD`) |
| `--kst-code` | | KŚT classification code |

#### `assets edit` flags

Optional: `--name`, `--depreciation-rate`.

### Fiscal periods

| Command | Key flags | Description |
|---|---|---|
| `numify periods list` | `--company` | List fiscal periods. |
| `numify periods lock <id>` | `--company`, `--immediate` | Lock a period. |
| `numify periods close <id>` | `--company`, `--immediate` | Close a period. |
| `numify periods reopen <id>` | `--company`, `--immediate` | Reopen a period. |
| `numify periods year-end` | `--company`, `--immediate` | Year-end closing. |

### Opening balance

| Command | Key flags | Description |
|---|---|---|
| `numify opening-balance get` | `--company` | Show opening balance. |
| `numify opening-balance set` | `--company`, `--immediate` | Set opening balance. |

#### `opening-balance set` flags

| Flag | Required | Description |
|---|---|---|
| `--company` | ✅ | Company ID |
| `--entries` | ✅ | JSON array: `[{"accountId": "...", "debit": 10000}, {"accountId": "...", "credit": 10000}]` |

### Pending actions

| Command | Description |
|---|---|
| `numify pending list` | List pending (deferred) actions. |
| `numify pending get <id>` | Show pending action details. |
| `numify pending cancel <id>` | Cancel a pending action. |
| `numify pending execute <id>` | Force-execute a pending action now. |
| `numify pending retry <id>` | Retry a failed action. |

### Compliance

| Command | Description |
|---|---|
| `numify compliance` | Compliance checks and status. |

### Export

| Command | Description |
|---|---|
| `numify export` | Export data (CSV, JSON). |

### Dashboard

| Command | Description |
|---|---|
| `numify dashboard --company <id>` | Quick company dashboard overview. |

### Misc

| Command | Description |
|---|---|
| `numify update` | Check for CLI updates. |
| `numify completion [zsh\|bash\|fish]` | Shell completion script. |

## Domain reference

### Money

All monetary amounts are **integers in grosze** (1 PLN = 100 groszy). Example: 1234 = 12.34 PLN.

### Double-entry

Every journal entry must balance: total debits = total credits. The API rejects unbalanced entries.

### Fiscal periods

- Periods can be locked (no edits) or closed (finalized).
- Writing to a closed period returns exit code `10` (`PERIOD_CLOSED`).
- Use `periods reopen` to unlock if needed.

### VAT rates (Poland)

| Rate | Usage |
|---|---|
| `23` | Standard |
| `8` | Reduced (food, some services) |
| `5` | Reduced (books, some food) |
| `0` | Zero-rated (EU exports) |
| `zw` | Exempt |
| `np` | Not applicable |

## Error codes and exit codes

| Exit | Code | Meaning |
|------|------|---------|
| 0 | — | Success |
| 1 | `UNKNOWN` / `CONFIG_ERROR` | Generic or config error |
| 2 | `USER_INPUT_ERROR` / `VALIDATION_ERROR` | Bad input |
| 3 | `NOT_AUTHENTICATED` | No token or invalid token |
| 4 | `FORBIDDEN` | Not authorized |
| 5 | `NOT_FOUND` | Resource not found |
| 6 | `CONFLICT` | Conflicting state |
| 7 | `RATE_LIMITED` | Too many requests |
| 8 | `NETWORK_ERROR` | Cannot reach the API |
| 9 | `SERVER_ERROR` | Server-side failure |
| 10 | `PERIOD_CLOSED` | Fiscal period is locked/closed |

`error.hint` may contain `action`, `flag`, `env`, or `url` fields suggesting recovery steps.

## Environment variables

| Var | Purpose |
|---|---|
| `NUMIFY_TOKEN` | API key. Overrides saved config. |
| `NUMIFY_API_URL` | Override API base URL (default: `https://numify.ai`). |
| `NUMIFY_DEBUG=1` | Dump HTTP traffic to stderr (secrets redacted). |
| `NUMIFY_NO_UPDATE_CHECK=1` | Disable update check. |

## Agent recipes

### List companies and pick one

```bash
COMPANY=$(numify companies list --json | jq -r '.data.companies[0].id')
```

### Show full company details

```bash
numify companies get $COMPANY --json | jq '.data'
```

### Update company tax office

```bash
numify companies edit $COMPANY --tax-office-code 1432 --tax-office-name "Urząd Skarbowy Kraków-Śródmieście" --immediate --json
```

### Create a transaction (deferred)

```bash
numify transactions create --company $COMPANY --type expense \
  --description "Office supplies" --amount-net 10000 --vat-rate 23 \
  --amount-vat 2300 --amount-gross 12300 --json
```

### Create a transaction (immediate, non-interactive)

```bash
numify transactions create --company $COMPANY --type income \
  --description "Consulting invoice" --amount-net 500000 \
  --invoice-number "FV/2025/001" --invoice-date 2025-01-15 \
  --counterparty "Acme Sp. z o.o." --counterparty-tax-id 1234567890 \
  --is-paid --immediate --json
```

### Create a journal entry (double-entry)

```bash
numify journal create --company $COMPANY \
  --description "Office rent January" \
  --entries '[{"accountId":"ACC_ID_1","debit":500000},{"accountId":"ACC_ID_2","credit":500000}]' \
  --date 2025-01-31 --immediate --json
```

### Get trial balance

```bash
numify reports trial-balance --company $COMPANY --year 2025 --month 1 --json | jq '.data'
```

### VAT summary for a period

```bash
numify vat summary --company $COMPANY --period 2025-01 --json | jq '.data'
```

### Register a fixed asset

```bash
numify assets create --company $COMPANY --name "MacBook Pro" \
  --initial-value 1299900 --depreciation-method linear \
  --depreciation-rate 30 --acquisition-date 2025-01-10 \
  --kst-code 491 --immediate --json
```

### Upload a document

```bash
numify documents upload --company $COMPANY --file invoice.pdf --json
```

### List pending actions and cancel one

```bash
numify pending list --json | jq '.data'
numify pending cancel <action-id> --json
```

### Set opening balance

```bash
numify opening-balance set --company $COMPANY \
  --entries '[{"accountId":"ACC1","debit":1000000},{"accountId":"ACC2","credit":1000000}]' \
  --immediate --json
```

### Dashboard overview

```bash
numify dashboard --company $COMPANY --json | jq '.data'
```

### Create a contractor

```bash
numify contractors create --company $COMPANY --name "Acme Sp. z o.o." \
  --tax-id 1234567890 --type supplier --city Kraków \
  --street "ul. Główna 1" --zip 30-001 --immediate --json
```
