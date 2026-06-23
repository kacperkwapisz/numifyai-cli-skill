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
  version: "1.0"
---

# Numify AI CLI

Numify AI is a bookkeeping tool for Polish sp. z o.o. companies. The `numify` CLI manages companies, transactions, journal entries, VAT, reports, and more from the terminal. Every command supports `--json` for agent and CI consumption.

## Install

```bash
npm i -g numifyai
```

Requires Node.js 22+.

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
    "cliVersion": "0.1.0"
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

| Command | Key flags | Description |
|---|---|---|
| `numify companies list` | | List companies you have access to. |
| `numify companies get <id>` | | Show company details. |
| `numify companies edit <id>` | `--immediate` | Edit company settings. |

### Transactions

| Command | Key flags | Description |
|---|---|---|
| `numify transactions list` | `--company <id>`, `--status`, `--from`, `--to` | List transactions. |
| `numify transactions get <id>` | `--company <id>` | Show transaction details. |
| `numify transactions create` | `--company <id>`, `--immediate` | Create a transaction. |
| `numify transactions edit <id>` | `--company <id>`, `--immediate` | Edit a transaction. |
| `numify transactions correct <id>` | `--company <id>`, `--immediate` | Create correction entry. |
| `numify transactions archive <id>` | `--company <id>`, `--immediate` | Archive a transaction. |
| `numify transactions verify <id>` | `--company <id>` | Verify/approve a transaction. |
| `numify transactions reject <id>` | `--company <id>` | Reject a transaction. |

### Journal entries

| Command | Key flags | Description |
|---|---|---|
| `numify journal list` | `--company <id>` | List journal entries. |
| `numify journal get <id>` | `--company <id>` | Show journal entry details. |
| `numify journal create` | `--company <id>`, `--immediate` | Create a journal entry. |
| `numify journal post <id>` | `--company <id>` | Post a draft entry. |
| `numify journal storno <id>` | `--company <id>`, `--immediate` | Storno (reverse) an entry. |

### Chart of accounts

| Command | Description |
|---|---|
| `numify accounts list --company <id>` | List accounts in chart of accounts. |
| `numify accounts get <id> --company <id>` | Show account details. |

### VAT

| Command | Key flags | Description |
|---|---|---|
| `numify vat list` | `--company <id>`, `--period` | List VAT register entries. |
| `numify vat create` | `--company <id>`, `--immediate` | Create VAT entry. |
| `numify vat edit <id>` | `--company <id>`, `--immediate` | Edit VAT entry. |
| `numify vat summary` | `--company <id>`, `--period` | VAT summary for period. |

### Contractors

| Command | Key flags | Description |
|---|---|---|
| `numify contractors list` | `--company <id>` | List contractors. |
| `numify contractors get <id>` | `--company <id>` | Show contractor details. |
| `numify contractors create` | `--company <id>`, `--immediate` | Create a contractor. |
| `numify contractors edit <id>` | `--company <id>`, `--immediate` | Edit a contractor. |

### Documents

| Command | Key flags | Description |
|---|---|---|
| `numify documents list` | `--company <id>` | List documents/invoices. |
| `numify documents get <id>` | `--company <id>` | Show document details. |
| `numify documents upload` | `--company <id>`, `--file <path>` | Upload a document. |

### Ledger

| Command | Description |
|---|---|
| `numify ledger list --company <id>` | List general ledger entries. |
| `numify ledger get <id> --company <id>` | Show ledger entry details. |

### Reports

| Command | Key flags | Description |
|---|---|---|
| `numify reports trial-balance` | `--company <id>`, `--period` | Trial balance report. |
| `numify reports balance-sheet` | `--company <id>`, `--period` | Balance sheet (Bilans). |
| `numify reports profit-loss` | `--company <id>`, `--period` | Profit & Loss (RZiS). |
| `numify reports cit` | `--company <id>`, `--year` | CIT tax report. |

### Bank accounts

| Command | Key flags | Description |
|---|---|---|
| `numify bank list` | `--company <id>` | List bank accounts. |
| `numify bank create` | `--company <id>`, `--immediate` | Create bank account. |
| `numify bank edit <id>` | `--company <id>`, `--immediate` | Edit bank account. |
| `numify bank lines` | `--company <id>`, `--account <id>` | List bank statement lines. |
| `numify bank import` | `--company <id>`, `--file <path>` | Import bank statement. |
| `numify bank auto-match` | `--company <id>` | Auto-match bank lines to transactions. |

### Fixed assets

| Command | Key flags | Description |
|---|---|---|
| `numify assets list` | `--company <id>` | List fixed assets. |
| `numify assets get <id>` | `--company <id>` | Show asset details. |
| `numify assets create` | `--company <id>`, `--immediate` | Register a fixed asset. |
| `numify assets edit <id>` | `--company <id>`, `--immediate` | Edit an asset. |
| `numify assets dispose <id>` | `--company <id>`, `--immediate` | Dispose/retire an asset. |
| `numify assets depreciation <id>` | `--company <id>` | Show depreciation schedule. |
| `numify assets generate-depreciation` | `--company <id>`, `--immediate` | Generate depreciation entries. |
| `numify assets export` | `--company <id>` | Export asset register. |

### Fiscal periods

| Command | Key flags | Description |
|---|---|---|
| `numify periods list` | `--company <id>` | List fiscal periods. |
| `numify periods lock <id>` | `--company <id>`, `--immediate` | Lock a period. |
| `numify periods close <id>` | `--company <id>`, `--immediate` | Close a period. |
| `numify periods reopen <id>` | `--company <id>`, `--immediate` | Reopen a period. |
| `numify periods year-end` | `--company <id>`, `--immediate` | Year-end closing. |

### Opening balance

| Command | Key flags | Description |
|---|---|---|
| `numify opening-balance get` | `--company <id>` | Show opening balance. |
| `numify opening-balance set` | `--company <id>`, `--immediate` | Set opening balance. |

### Compliance

| Command | Description |
|---|---|
| `numify compliance` | Compliance checks and status. |

### Pending actions

| Command | Description |
|---|---|
| `numify pending list` | List pending (deferred) actions. |
| `numify pending get <id>` | Show pending action details. |
| `numify pending cancel <id>` | Cancel a pending action. |
| `numify pending execute <id>` | Force-execute a pending action now. |
| `numify pending retry <id>` | Retry a failed action. |

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

### List companies

```bash
numify companies list --json | jq '.data'
```

### Create a transaction (deferred)

```bash
numify transactions create --company <id> --json < payload.json
```

### Create a transaction (immediate, non-interactive)

```bash
numify transactions create --company <id> --immediate --json < payload.json
```

### Get trial balance

```bash
numify reports trial-balance --company <id> --period 2025-01 --json | jq '.data'
```

### List pending actions and cancel one

```bash
numify pending list --json | jq '.data'
numify pending cancel <action-id> --json
```

### VAT summary for a period

```bash
numify vat summary --company <id> --period 2025-01 --json | jq '.data'
```

### Upload a document

```bash
numify documents upload --company <id> --file invoice.pdf --json
```

### Dashboard overview

```bash
numify dashboard --company <id> --json | jq '.data'
```
