# Changelog

All notable changes to the `portfolio` skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-05-03

### Added

- Initial release of the `portfolio` skill (Trading212).
- Step 0: Configure with **`AskUserQuestion`-with-fallback** — region, currency, reconciliation system, drift threshold. Persists to KV `trading212-portfolio/setup`.
- Step 1-2: Playwright MCP browser navigation + accessibility-snapshot extraction (per-position name/ticker/shares/value/return + account totals + cash + last-updated timestamp).
- Step 3-4: Reconciliation table against the user's record system, flagging missing positions, value drift, stale records.
- Step 5: Upsert back into KV / Notion / Sheets (or output paste-able markdown if no API write).
- Step 6: Optional dashboard render verification.
- Common-issues guide: bot detection (Playwright MCP required), login + 2FA hand-off, stale T212 cache, currency mismatch, Pies handling.
- Read-only by design — explicitly does NOT place trades. Login and 2FA always hand off to the user.
