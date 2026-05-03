# trading212-portfolio

An agent skill that pulls live **Trading212** positions and totals via Playwright MCP browser automation, then reconciles against the user's own portfolio record system (a personal DB, an Obsidian note, a Google Sheet, a Notion DB — whatever they use).

Trading212 doesn't ship a public-friendly export API for individual investors, so the cleanest path is browser-driven extraction. This skill handles the bot-detection / 2FA / stale-cache edge cases that bite a naive implementation.

## What it does

1. **Asks for setup** — Trading212 region, account currency, target reconciliation system, discrepancy threshold. Uses `AskUserQuestion`-with-fallback if available; otherwise asks inline. Persists to KV `trading212-portfolio/setup` so subsequent runs are zero-prompt.
2. **Opens Trading212 via Playwright MCP** — handles bot detection that headless browsers trigger.
3. **Hands the login back to the user** — captcha + email/SMS 2FA mean automation isn't safe. The skill takes a screenshot, asks the user to log in, then resumes after the redirect to `/portfolio`.
4. **Extracts the snapshot** — per-position name, ticker, shares, value, return; account totals; cash balance; T212's own "Last updated" timestamp.
5. **Reads the user's records** — from whichever persistence layer they configured.
6. **Reconciles** — produces a comparison table flagging missing positions, value drift beyond threshold, stale records.
7. **Upserts back** — if the user's record system supports writes (KV, Notion API, Sheets API, etc.), upsert the new snapshot with a timestamp. Otherwise, output a markdown block to paste into notes manually.

## Constraints

- **Read-only.** Never places trades. Ever.
- **No password automation.** Login + 2FA hand off to the user every time. Trading212's session model isn't safe to script.
- **Currency-aware.** If T212 displays in EUR but the user's records are in CZK/USD, conversion is explicit and the rate gets recorded in the snapshot for traceability.
- **Pie-aware.** Pies show as one combined entry; the skill drills in only if per-instrument detail is needed.

## Out of scope

- Tax reporting (capital gains, dividend tax) — different concern; pair with a tax-return skill.
- Order execution — read-only by design.
- ISA vs. Invest account distinction — extraction is identical; treat as one portfolio unless the user explicitly separates them.

## Install

In an active agent session that supports the plugin format (e.g. Claude Code):

```
/plugin marketplace add MrSucik/trading212-portfolio
/plugin install trading212-portfolio@trading212-portfolio
```

Or from the terminal:

```bash
claude plugin marketplace add MrSucik/trading212-portfolio
claude plugin install trading212-portfolio@trading212-portfolio
```

Pin to a release tag:

```
/plugin marketplace add MrSucik/trading212-portfolio@v1.0.0
```

## Use

```
/portfolio
```

Or describe it: `"check my Trading212 portfolio"`, `"reconcile T212 against my records"`, `"pull my latest holdings"`.

## License

MIT — see [LICENSE](./LICENSE).
