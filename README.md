# GPW Company Analysis — Claude Skill for Warsaw Stock Exchange

Analyze Warsaw Stock Exchange (GPW) companies with Claude, Cursor, or Codex — powered by [GpwAlert](https://gpwalert.com).

This **Agent Skill** turns your AI assistant into a GPW stock analyst: pull live prices, technical indicators (RSI, MACD, SMA), ESPI/EBI disclosures, quarterly financials, analyst recommendations, dividends, corporate events, and reconstruct order backlogs from primary-source reports — all via the [GpwAlert MCP server](https://gpwalert.com/mcp-dla-ai).

Works with **Claude Code**, **Cursor**, **Codex**, and any agent that supports the [Agent Skills](https://agentskills.io) standard.

---

## Analiza spółek GPW w AI / GPW stock analysis in AI

**Polski:** Skill do analizy spółek z GPW (Giełda Papierów Wartościowych) — WIG20, mWIG40, sWIG80. Pobiera notowania, wolumen, rekomendacje analityków, komunikaty ESPI, wyniki finansowe, dywidendy, wydarzenia korporacyjne i portfel zamówień. Wymaga [GpwAlert Premium](https://gpwalert.com).

**English:** A Claude skill / Cursor skill for Warsaw Stock Exchange (GPW) company analysis. Covers Polish stocks, ESPI filings, financial reports, technical analysis, brokerage recommendations, and order backlog reconstruction. Requires [GpwAlert Premium](https://gpwalert.com).

---

## Install

### Cursor, Codex, OpenCode, and 70+ agents (skills.sh)

```bash
npx skills add wlodekj/gpwalert-gpw-company-analysis
```

### Claude Code (plugin marketplace)

```text
/plugin marketplace add wlodekj/gpwalert-gpw-company-analysis
/plugin install gpwalert@gpwalert
/reload-plugins
```

Then invoke:

```text
/gpwalert:gpw-company-analysis
```

Or ask naturally — Claude loads the skill when you mention GPW company analysis.

### GpwAlert MCP (required for data)

The skill reads live GPW data from the GpwAlert MCP server. Add to `.cursor/mcp.json` (Cursor) or enable the bundled `.mcp.json` (Claude Code plugin):

```json
{
  "mcpServers": {
    "gpwalert": {
      "url": "https://gpwalert.com/mcp"
    }
  }
}
```

On first use, sign in with your **GpwAlert Premium** account. Full setup guide: [gpwalert.com/mcp-dla-ai](https://gpwalert.com/mcp-dla-ai)

---

## Example prompts

```
Zrób pełną analizę spółki CDR
```

```
Jak oceniasz PKO? Sprawdź notowania, rekomendacje i nadchodzące wydarzenia.
```

```
Jaki jest portfel zamówień Budimex? Przeanalizuj komunikaty ESPI.
```

```
Analyze Allegro (ALE) — price action, RSI/MACD, analyst targets, and latest ESPI news.
```

```
Compare recent volume and recommendations for KGHM and JSW.
```

---

## What this skill analyzes

| Data | GpwAlert MCP tools |
|------|-------------------|
| Company profile, sector, index membership | `get_company`, `list_companies` |
| Price history, OHLC, volume | `get_company_prices` |
| Returns, RSI, MACD, SMA50/200, drawdown | `get_company_price_metrics` |
| Analyst ratings and target prices | `get_company_recommendations` |
| Brokerage report content | `get_company_analyst_recommendation` |
| Press news | `get_company_news` |
| Dividends, earnings dates, WZA | `get_company_events` |
| Weekly AI analyses | `get_company_analyses` |
| Quarterly financials and KPIs | `get_company_financials`, `get_company_kpis` |
| ESPI filings and PDF attachments | `get_company_documents`, `get_company_document`, `get_company_document_attachment` |
| Order backlog reconstruction | ESPI contracts + management reports (step 6 in skill) |

---

## Report structure

A full analysis typically covers:

1. **Kurs i wolumen** — price move, horizons, RSI/MACD/SMA read, volume anomalies
2. **Rekomendacje** — latest analyst calls, target vs. current price
3. **Newsy / komunikaty ESPI** — material disclosures with context
4. **Nadchodzące wydarzenia** — next report date, dividends, catalysts
5. **Ocena całościowa** — synthesis, risks, what to watch
6. **Portfel zamówień** (on request) — reconstructed backlog with explicit caveats

---

## Keywords

Claude skill, Cursor skill, Agent Skills, MCP skill, GPW analysis, Warsaw Stock Exchange, Giełda Papierów Wartościowych, Polish stocks, WIG20, mWIG40, sWIG80, ESPI, EBI, stock analysis, analiza spółki, rekomendacje analityków, wyniki finansowe, notowania GPW, dywidendy, portfel zamówień, order backlog, RSI MACD, GpwAlert, gpwalert.com

---

## Requirements

- **[GpwAlert Premium](https://gpwalert.com)** subscription (MCP access is not included in the free trial)
- An AI coding agent with Agent Skills support (Claude Code, Cursor, Codex, etc.)
- Internet access for MCP and optional web search

---

## Get GpwAlert Premium

**[gpwalert.com](https://gpwalert.com)** — monitor 335+ GPW companies, ESPI alerts, financial analysis, weekly summaries, and AI-powered insights.

MCP documentation: **[gpwalert.com/mcp-dla-ai](https://gpwalert.com/mcp-dla-ai)**

---

## Disclaimer

This skill provides factual data and structured analysis. It is **not investment advice**. Always do your own research before making investment decisions.

---

## License

MIT — see [LICENSE](LICENSE).

Built by [GpwAlert.com](https://gpwalert.com) — GPW stock monitoring and AI analysis for Polish investors.
