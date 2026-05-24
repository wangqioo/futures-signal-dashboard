# 期货信号看板 — Agent Guide

## Repository Intent

This repository contains a formula-driven futures market dashboard. It turns AkShare/Sina futures bars and TDX/Huiying-style formulas into chart panels, signal alerts, watchlist scans, and trend status.

Use the project language in [CONTEXT.md](CONTEXT.md). The core is the formula-to-panel pipeline, not trade execution.

Do not move live order placement, account login, broker APIs, payment/subscription flows, user systems, or execution risk controls into this repository.

## Domain Language

Prefer these terms in documentation and code-level explanations:

- **期货信号看板**, not trading bot, when describing the product.
- **公式指标**, not random calculation, when describing TDX/Huiying-derived logic.
- **主图信号**, not only marker, when describing QRG/破浪/空仓 output on the K-line chart.
- **波段王副图**, not second chart, when describing K/D momentum panel.
- **信号**, not order instruction, when describing 做多/做空 alert state.
- **信号队列**, not message bus, for `_pending_signals`.
- **信号去重库**, not generic database, for `data/signals.db`.
- **触发周期**, not polling interval, for the K-line period used by scanners.
- **主力连续合约**, not stock symbol, for `P0`, `CU0`, etc.
- **具体月份合约**, not custom symbol, for inputs such as `RB2510`.

## Runtime Entry Points

| Entry point | Purpose |
| --- | --- |
| `python server.py` | Main local dashboard runtime on port 8877 |
| `python scheduler.py` | Optional standalone scanner that pushes to `server.py` |

Useful commands:

```bash
python -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/python server.py
python -m compileall -q .
docker compose up -d --build
```

## Core Modules

| File | Responsibility |
| --- | --- |
| `server.py` | Flask API, data fetch/cache, indicator response assembly, signal queue, built-in scanner |
| `dashboard/index.html` | Single-file frontend for K-line chart, Wave King panel, watchlist, alerts, trend panel |
| `indicators.py` | Built-in formula reproductions: QRG, 破浪, 空仓, 波段王 K/D |
| `indicators_pkg/` | Hot-loadable formula indicator plugins |
| `tdx_parser/` | TDX/Huiying formula parser and plugin source generator |
| `data_fetcher.py` | Pure support/resistance and capital-flow calculations |
| `scheduler.py` | Optional standalone scanner posting signals to the main Flask runtime |
| `data/` | Runtime cache and `signals.db`; do not commit generated data |

## Development Rules

- Keep the formula-to-panel pipeline readable: bars in, formula indicators out, chart-ready JSON returned.
- Do not change formula semantics casually. If an indicator formula changes, document the before/after rule in README or a dedicated note.
- Keep `server.py` and `dashboard/index.html` as the current all-in-one runtime shape unless doing an explicit refactor pass.
- Prefer small, named extraction only when it improves locality for data fetching, signal scanning, indicator calculation, or frontend rendering.
- Do not commit secrets. Deployment credentials must come from environment variables or a local `.env` file.

## Verification

There is no test suite yet. Use focused static/runtime checks:

```bash
python -m compileall -q .
.venv/bin/python server.py
curl http://localhost:8877/api/symbols
curl "http://localhost:8877/api/data?symbol=P0&period=3&mode=update"
```

AkShare calls depend on live network/data-provider availability, so separate syntax errors from provider outages when reporting verification.
