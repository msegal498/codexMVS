
# Project Requirements Document (PRD) v 1.0  
_Backtest.AI MVP – Web‑based Strategy Back‑testing Tool_

**Revision history**  
| Date | Version | Author | Notes |
|------|---------|--------|-------|
| 2025‑04‑17 | 1.0‑rc3 | ChatGPT (o3) | Locked scope for MVP |

---

## 1. Purpose & Goals
Build a public web app where registered users can back‑test simple trading strategies on selected tickers (BTC, QQQ, AAPL, MSFT) over the range **2020‑01‑01 → 2022‑04‑15 +**.  
MVP delivers:
1. Account system (Supabase Auth; Google, GitHub, email)  
2. “3 back‑tests free, then $1 each” pay‑as‑you‑go model (Stripe Checkout + webhook)  
3. Dashboard wizard or LLM‑chat flow that outputs:
   * TradingView price chart (Option A)  
   * Equity curve & trade markers overlay  
   * Stats table (# trades, total P/L, CAGR, MDD, win‑rate)  

---

## 2. User Stories
* **U1 (Onboard)** – As a first‑time visitor I can sign up with Google / GitHub / email.
* **U2 (Design strategy)** – I can describe a strategy in plain English _or_ pick indicators ➜ system shows DSL.
* **U3 (Run back‑test)** – I enter start/end dates and click **Run**; result appears within 60 s (dev) / 10 s (prod).
* **U4 (Pay as I go)** – After 3 free tests, I’m prompted to pay $1 and instantly receive new credits.
* **U5 (Ask questions)** – I can chat “What does RSI 14 mean?” and the LLM answers contextually.

---

## 3. Functional Requirements

### 3.1 Workflow Wizard
| Step | Prompt | Output |
|------|--------|--------|
| 1 | Choose **Ticker** | one of BTC, QQQ, AAPL, MSFT |
| 2 | Enter **Start / End** | yyyy‑mm‑dd |
| 3 | Select **Indicators / Conditions** | UI list or natural language |
| 4 | Confirm **DSL** | readonly string |
| 5 | Click **Run Back‑test** | triggers API |

### 3.2 Strategy DSL (examples)
```plain
CROSSOVER(MA(50),MA(200))
BREAKOUT(price, SR(zone_id="daily_high"))
RSI(14) < 30 AND PRICE > MA(200)
```

**Supported indicators (MVP)**  
MA, EMA, **Long‑term MA bands**, RSI, **VWAP**, **ATR**, **Stochastic**, Bollinger Bands, custom Support/Resistance IDs.

Start/End bound data window only; multiple trades can occur within.

### 3.3 LLM Assistant
* Cloudflare Worker → OpenAI (o3).  
* Converts NL → DSL; answers Q&A.  
* No usage cap (monetised by per‑test fee).

---

## 4. Non‑Functional Requirements
| NFR | Target |
|-----|--------|
| **Performance (dev/staging)** | p95 ≤ 60 s click→render |
| **Performance (prod)** | p95 ≤ 10 s |
| Uptime | 99 % monthly |
| Security | Supabase Auth JWT; Stripe secret keys server‑side |
| Compliance | GDPR + Stripe PCI‑DSS handled by hosted Checkout |

---

## 5. Tech Stack & Architecture

| Layer | Tech |
|-------|------|
| Front‑end | React/TypeScript on **Replit** (Nix); TradingView **AdvancedChartWidget** (iframe) |
| Back‑end | FastAPI functions → **Supabase Edge (Deno)** |
| Data ETL | nightly yfinance → Parquet in Supabase Storage; index table `prices_ohlcv` |
| DB | Supabase Postgres (`users`, `backtests`, `credits`) |
| Payments | Stripe Checkout (client) + **webhook** (Edge) for crediting |
| LLM | Cloudflare Worker proxy to OpenAI |
| DevOps | GitHub ▸ Supabase CI; Replit CI for FE |

**Data freshness** – single snapshot at 23:59 UTC.  (Tiingo upgrade + 15 min cache‑miss fetch = Milestone 2 option.)

---

## 6. Codebase Layout

```
repo-root/
├── frontend/        # Replit UI
├── backend/
│   ├── functions/   # Supabase edge functions (FastAPI compatible)
│   ├── etl/
│   └── tests/
└── infra/           # Supabase config, Stripe keys, CI
```

---

## 7. Road‑map / Milestones
| # | Deliverable | ETA |
|---|-------------|-----|
| 1 | Repo scaffold, Supabase tables, nightly ETL job | +1 wk |
| 2 | Wizard UI + LLM flow; runs back‑test locally | +2 wks |
| 3 | Stripe payments + credit logic | +3 wks |
| 4 | **Option A** TV widget overlay live on prod | +4 wks |
| 5 | **Option B** upgrade to licensed Charting Library | +8 wks |

---

## 8. Open Questions
1. Exact stat list for results table? (Currently: total $, # trades, win‑rate, MDD, Sharpe.)
2. Branding / domain?  
3. Licence cost & approval for TradingView Charting Library (Milestone 5).

---

*End of document*
