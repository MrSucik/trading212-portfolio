---
name: portfolio
description: Use when reviewing Trading212 positions, syncing portfolio data, checking holdings, or when user says /portfolio. Extracts positions, dividends, and account totals from Trading212 via Playwright MCP and reconciles them against the user's own portfolio record system.
---

# Trading212 Portfolio Update

## Overview

Pull live portfolio data from Trading212 (app.trading212.com) via browser automation, extract all positions and account totals, and reconcile against the user's own portfolio record system (a personal DB, a notes file, a tracker SaaS, whatever they use). Flags discrepancies — missing positions, value drift, stale sync.

Trading212 doesn't have a public-friendly export API for individual investors, so the cleanest path is browser-driven extraction via Playwright. This skill handles the bot-detection edge cases that bite a naive implementation.

## Step 0: Configure (one-time setup)

On first invocation, gather the data below and persist to a KV namespace `trading212-portfolio/setup`. Skip on subsequent runs.

**If a structured-question tool is available** (e.g. `AskUserQuestion` in Claude Code), batch the prompts into one structured call. Otherwise ask inline.

| Field | Notes |
|---|---|
| Trading212 account region | EU / US / UK — affects URL (`app.trading212.com` vs `app.trading212.com/?region=...`) |
| Account currency | EUR / USD / GBP / etc. — what T212 displays in |
| Target reconciliation system | Where the user keeps their own portfolio record (e.g. a Jarvis-like agent, an Obsidian note, a Google Sheet, a Notion DB, none) |
| Discrepancy threshold | How much value drift triggers a flag — default 1% |

The user is NOT asked for their T212 password — they enter it themselves in the browser when prompted.

## When to Use

- "Check my Trading212 portfolio"
- "Reconcile T212 against my records"
- "Update my portfolio"
- "Pull dividends from T212"
- `/portfolio`

## Workflow

### Step 1: Open Trading212 in Playwright MCP browser

```
mcp__playwright__browser_navigate → https://app.trading212.com/portfolio
```

- If a login page appears, take a screenshot and **ask the user to log in themselves**. Trading212 sessions involve a captcha + email/SMS 2FA — not safe to automate.
- Wait for the portfolio to load, then take an accessibility snapshot.

### Step 2: Extract positions from the snapshot

For each holding, extract:

- **Name** (e.g. "VanEck Semiconductor (Acc)")
- **Ticker** (e.g. `SMGB`)
- **Shares** (e.g. 3.911)
- **Value** in account currency
- **Return** (amount + percentage)

Also capture top-level totals:

- Total account value
- Cash balance (Main Pot)
- Total return + rate of return
- Last sync timestamp from T212 (visible in the corner)

### Step 3: Read the user's reconciliation system

Use whatever the user configured in Step 0:

- **Agent KV / Jarvis-like system** — read their persisted portfolio.
- **Markdown / Notion / Sheets** — fetch the latest version.
- **None** — skip reconciliation; jump to Step 5.

### Step 4: Compare and flag discrepancies

Show a table comparing T212 live vs the user's records:

```
| Holding | T212 Value | Recorded Value | Match? |
|---------|-----------|----------------|--------|
| ...     | ...       | ...            | ...    |
```

Flag any of:

- **Missing positions** — held on T212 but not in records (or vice versa)
- **Value drift** — beyond the threshold from Step 0
- **Stale records** — last update > 1 day old

### Step 5: Update the user's records

If the user's reconciliation system supports writes (KV, Notion API, Sheets API, etc.), upsert the latest data with a timestamp. Otherwise, output a markdown block the user can paste into their notes manually.

Always include:
- Snapshot date/time
- Per-position: ticker, shares, value, return
- Totals: account value, cash, total return, RoR
- Source: "Trading212 live snapshot via Playwright MCP"

### Step 6: (Optional) Render in the user's dashboard

If the user has a portfolio dashboard URL (their own site / Notion page / etc.) configured, navigate there to verify the new data renders correctly.

## Common Issues

- **Trading212 bot detection** — use Playwright MCP, NOT a generic browser-control wrapper. T212 blocks headless browsers without a realistic user agent and viewport. Playwright MCP handles this out of the box.
- **Login required** — never automate the password. Take a screenshot, hand the keyboard to the user, wait for the redirect to `/portfolio`.
- **2FA** — same as login. T212's 2FA is email or SMS; the user types the code.
- **Stale T212 cache** — T212's portfolio page sometimes shows yesterday's prices for the first ~5 seconds after load. Wait for the "Last updated" timestamp to refresh before extracting.
- **Currency mismatch** — if the user's records are in CZK but T212 shows EUR, convert at a documented rate (and record the rate in the snapshot for traceability).
- **Pies / autoinvest** — Trading212 Pies appear as a single combined entry. Drill into the Pie if you need per-instrument detail.

## Out of scope

- Tax reporting (capital gains, dividend tax) — different concern, different skill.
- Order execution — this skill is read-only by design. Don't place trades automatically.
- ISA / Invest account distinction — extraction is the same; treat as one portfolio unless the user explicitly tracks them separately.
