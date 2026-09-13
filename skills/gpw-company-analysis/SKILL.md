---
name: gpw-company-analysis
description: >-
  This skill should be used when the user asks to analyze, evaluate, or review a company listed on the Warsaw Stock Exchange (GPW). It produces a sourced GPW stock analysis via the GpwAlert MCP at https://gpwalert.com — prices, volume, RSI/MACD/SMA, ESPI/EBI disclosures, financials, KPIs, dividends, analyst recommendations, news, events, and order-backlog reconstruction. Use for Polish tickers (WIG20, mWIG40, sWIG80) and requests about stock analysis, company review, price action, trading volume, brokerage recommendations, ESPI filings, quarterly results, dividends, order backlog, or planned tenders. Requires GpwAlert Premium (https://gpwalert.com).
license: MIT
compatibility: Requires GpwAlert MCP (https://gpwalert.com/mcp) and GpwAlert Premium subscription.
metadata:
  author: GpwAlert
  version: "1.0.0"
  homepage: https://gpwalert.com
---

# GPW Company Analysis

A repeatable workflow for analyzing a single company on the Warsaw Stock Exchange (GPW), built around the **GpwAlert** MCP connector as the primary data source, with **web search** used to fill in anything GpwAlert doesn't cover (live tender platforms, competitor moves, sector-level news).

This produces the kind of multi-angle report a serious retail investor or analyst would want: not just "the stock is up," but price + volume + technicals + what analysts think + what the company itself has disclosed + what's coming next.

## When to use this

Trigger on requests like:
- "zrób pełną analizę spółki X" / "jak oceniasz to co się dzieje ze spółką X"
- "sprawdź wolumeny/rekomendacje/wiadomości dla X"
- "jaki jest obecny portfel zamówień X"
- "jakie przetargi są zaplanowane dla X" (when the answer feeds back into that company's pipeline)
- Any question about a specific ticker's fundamentals, technicals, or news that would benefit from combining several data types rather than answering from general knowledge

Don't use this for questions about GPW/the market in general, or for companies not listed on GPW (no GpwAlert coverage) — for those, fall back to plain web search.

## Before anything else: connect GpwAlert MCP

Data comes from the **GpwAlert MCP server** at https://gpwalert.com/mcp. Requires an active **GpwAlert Premium** subscription (https://gpwalert.com).

**Setup (Cursor):** add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "gpwalert": {
      "url": "https://gpwalert.com/mcp"
    }
  }
}
```

**Setup (Claude Code):** enable the bundled `.mcp.json` in this plugin, or add the same config manually. On first use, authenticate with your GpwAlert Premium account.

Discover tool schemas before calling tools — do not guess parameter names. If a short company name is given (not a ticker/UUID/slug), resolve it with `list_companies(query=...)` first — never guess tickers.

## Core workflow

Run these in roughly this order. Not every report needs every step — scale to what the user actually asked — but a request for a "full/pełna analiza" should touch all of them.

### 1. Identify the company
Call `get_company` with the ticker, name, or friendlyUrl the user gave you. This confirms you have the right entity and gives you sector, indices, and basic profile info to anchor the rest of the report.

### 2. Price action, volume, technicals
- `get_company_prices` for OHLC history (pick a window — 3–6 months is usually enough context; go longer only if the user asks about long-term trend).
- `get_company_price_metrics` for returns over multiple horizons (5D/1M/3M/6M/12M), RSI, MACD, SMA50/SMA200, drawdown, liquidity/turnover flags.
- Read these together, not in isolation: a price jump only means something in light of whether volume was unusual, whether RSI signals overbought/oversold, and where price sits relative to SMA50/SMA200. Flag when a stock is extended (RSI overbought, price above short-term MA but still below the long-term one) rather than just reporting the numbers.

### 3. Analyst view
- `get_company_recommendations` for a list of recent analyst calls and brokerage reports (summaries).
- `get_company_analyst_recommendation` with `includeReportContent: true` on the most recent/relevant one if you need the underlying detail — but check `hasReportContent`; many sources only expose the summary, not full text (respect this, don't fabricate report content that isn't there).
- Note the rating, target price, and how the current price compares to that target — that comparison is often the single most useful line in the whole report.

### 4. Company-disclosed news and events
- `get_company_news` for external press coverage (try a wider `days` window if a narrow one comes back empty — companies don't always get daily press coverage).
- `get_company_events` for corporate actions: dividends, earnings dates, WZA (shareholder meetings). The **next scheduled report date** is usually worth calling out explicitly — markets often position ahead of it.
- `get_company_analyses` for GpwAlert's own weekly AI-generated analyses — these are a fast way to get a running narrative of sentiment/consolidation/breakout without reading every ESPI filing yourself.

### 5. Financials and KPIs
`get_company_financials` and `get_company_kpis` for quarterly figures. If these come back empty for the requested period, don't just report "no data" — try widening the request or pulling the underlying annual/quarterly report via step 6 instead.

### 6. Primary-source documents (for anything the structured endpoints don't capture)
This is the step that separates a surface-level summary from a real analysis, and it's essential for things like **order backlog**, which companies almost never disclose as one clean number.

- `get_company_documents` lists ESPI filings (reports, contracts, other disclosures). This can be long — if the tool result gets truncated/stored to a file, search it with `grep` rather than trying to read the whole thing.
- `get_company_document` with `includeContent: true` gets you the cover/summary of a specific filing (e.g. an annual report). This is usually just the headline financial table, not the full management discussion — check the `attachments` list on the response.
- `get_company_document_attachment` with `includeContent: true` on the "Sprawozdanie Zarządu" / management-report attachment is where the real narrative lives: signed contracts, backlog commentary, risk factors, strategy. These attachments can be very large (500k+ characters) — save the returned text to a file and `grep` for the terms you need (e.g. `portfel zamówień`, `wartość umowy`, `mld zł`, `backlog`) rather than reading it end to end.

**Key lesson on backlog specifically**: Polish issuers typically report *new contracts signed in the reporting year* (e.g. "w 2025 roku spółka zawarła umowy o łącznej wartości X mld zł"), not a single cumulative backlog figure. To estimate backlog:
1. Pull every ESPI contract announcement (from `get_company_documents`/news) that hasn't been fully delivered yet.
2. Note whether each figure is the base value or includes an exercised/potential option — state both where relevant, and use the *current* total (after any option already exercised).
3. Sum only genuinely distinct, still-open contracts — watch for the same framework agreement appearing under multiple "executive agreement" announcements (these are additive, not duplicates) versus the same contract being restated with an updated total after an option (these replace the earlier figure, don't add to it).
4. Present the sum as **your reconstruction**, explicitly labeled as such, not as a figure the company itself published. Say plainly that the company doesn't disclose one number, and point to the next scheduled report as the place to check for a more direct management comment.

### 7. External context via web search
When the question extends beyond the company itself — planned tenders from its customers, a competitor's moves, sector-wide regulatory changes — switch to web search. Useful patterns from practice:
- Check the actual procurement platform of major customers (e.g. a national rail operator's e-procurement site) for open/planned tenders, not just news aggregators.
- Cross-reference company self-reported "expected 2026 tenders" commentary (from step 6) against what's actually been announced — note where the company appears to be excluded from a process it was previously pursuing (e.g. a partnership that was dissolved), since that's a materially different signal than "still in the running."
- Government/ministry press releases are usually more reliable than trade press for hard dates and figures.

## Report structure

Default to plain, conversational prose in the reply (not a file) unless the user asks to save/export it. Use light structure — short section labels, not a formal report template — matching the register of the rest of the conversation. A reasonable shape:

1. **Kurs i wolumen** — recent move, how it compares across horizons, technical read (RSI/MACD/SMA), anything unusual about volume
2. **Rekomendacje** — latest analyst calls, target vs. current price
3. **Newsy / komunikaty ESPI** — material disclosures, in date order, each with a one-line read on why it matters
4. **Nadchodzące wydarzenia** — next report date and why it matters, other scheduled catalysts
5. **Ocena całościowa** — synthesis: what's driving the recent move, what the risks/catalysts are, what to watch next
6. For backlog/deep-dive requests: a dedicated section with the reconstructed figure, the individual contracts behind it, and the caveat that it's an estimate

## Non-negotiables

- **Always** end an analysis touching price/valuation with a plain disclaimer that this isn't investment advice — state facts and let the person draw their own conclusion.
- **Never** invent a number that isn't in the data (a target price, a backlog figure, a report date). If a field is empty or a report has no content available, say so rather than filling the gap.
- **Never** present your own summed/reconstructed figures as if the company published them — always attribute reconstructions to yourself and flag the uncertainty.
- Prefer **primary sources** (the company's own ESPI filings and reports) over secondhand summaries when the two could differ, especially for anything with a specific number attached.
