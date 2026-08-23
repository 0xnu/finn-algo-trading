## finn-algo-trading

A framework for calibrating take-profits to structure-anchored stop-loss distance via VRMR, SLF, R²CTP, OTCC, and PSE, with ten-year walk-forward FX results, a 25 000 USD production-parameter simulation, an FTMO 100K Challenge backtest, and a January to June 2026 session-level study.

> [!WARNING]
> Disclaimer: I do not have any affiliation with FTMO. This project is purely academic and non-commercial. Trade sessions are simulated for educational purposes only.


### Session-Level Performance (Jan–Jun 2026)

Six-month, 126-trading-day daily-profit decomposition on the 25 000 USD production-parameter account simulation (1 Jan 2026 to 30 Jun 2026).

| Trading Session     | Opening (UTC) | Closing (UTC) | Net Profit (USD) | Share of Total | Contribution to Book |
|---------------------|---------------|---------------|-----------------:|---------------:|---------------------:|
| European            | 07:00         | 16:00         |          2 822   | 46.7 percent   | 11.29 percent        |
| North American      | 12:00         | 21:00         |          1 742   | 28.8 percent   |  6.97 percent        |
| Asian               | 23:00         | 08:00 (+1)    |          1 463   | 24.2 percent   |  5.85 percent        |
| **All sessions**    | —             | —             |      **6 048**   | **100.0 %**    | **24.19 percent**    |

### Automated Deployment on AWS

The live signal pipeline runs on AWS EC2, hosts the FastAPI trading API on port 8000, and exposes endpoints for health checks, session state, pair lists, and per-pair signals.

| Layer | Path in `finn-algo-trading/` | Purpose |
|-------|---------------------------|---------|
| Reference pipeline (paper) | `signal_v2/` | VRMR regime classification, SLF pivot stops, R²CTP take-profit, OTCC order routing, PSE position sizing, session-aware orchestrator |
| — regime & structure | `signal_v2/market_structure.py` / `market_session.py` | Range Class VRMR partition, swing-pivot SLF buffers, Asian/European/North American session windows |
| — routing & sizing | `signal_v2/position.py` / `signal_generator.py` | R²CTP TP = entry ± λ × realised SL distance, OTCC MARKET/LIMIT/SKIP, PSE lot sizing, dual orchestrator entrypoints |
| — strategy mix | `signal_v2/strategies.py` | MA Crossover, Momentum Breakout, Mean-Reversion RSI, Trend-Following ATR, Volume-Profile Breakout |
| — HTTP API (live) | `signal/trading_signals_api.py` + `signal/run_api.py` | FastAPI endpoints `GET /health`, `/session`, `/account`, `/pairs`, `/signals?symbol=EURUSD` |
| FTMO live harness | `live/finn_ftmo_live_trading.ipynb` | Standalone FTMO execution notebook. Runs classic strategies against FTMO 100K. Separate from the paper's `signal_v2/` reference pipeline. |
| Deployment guide | `signal/WINDOWS.md` | Security Group rules, Windows Firewall opening for port 8000, EC2 public-IP verification, curl smoke tests. |

Host architecture on AWS EC2 (Windows):
- **API process**: `python signal/run_api.py` (uvicorn `0.0.0.0:8000`)
- **Security Group**: inbound TCP 8000 opened to operator CIDR
- **Windows Firewall**: `netsh advfirewall firewall add rule name="FastAPI Port 8000" dir=in action=allow protocol=TCP localport=8000`
- **Smoke checks from outside**: `curl http://<EC2_PUBLIC_IP>:8000/health` and `curl http://<EC2_PUBLIC_IP>:8000/signals?symbol=EURUSD`
- **Docs/Swagger**: `http://<EC2_PUBLIC_IP>:8000/docs`

### License

This project is licensed under the [MIT License](./LICENSE).

### Citation

```tex
@misc{dtpctrsld2026,
  author       = {Oketunji, A.F.},
  title        = {Dynamic Take-Profit Calibrated to Real Stop-Loss Distance},
  year         = 2026,
  version      = {1.0.0},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.22066503},
  url          = {https://doi.org/10.5281/zenodo.22066503}
}
```

### Copyright

(c) 2026 [Finbarrs Oketunji](https://finbarrs.eu).