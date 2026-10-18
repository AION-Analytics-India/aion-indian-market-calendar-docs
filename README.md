# aion-indian-market-calendar

> **Source code has moved.** The package source now lives at
> **[aion-indian-market-calendar](https://github.com/AION-Analytics-India/aion-indian-market-calendar)**.
> This repository holds documentation only.
>
> - Source: https://github.com/AION-Analytics-India/aion-indian-market-calendar
> - Package: https://pypi.org/project/aion-indian-market-calendar/
> - Full write-up: https://dashboard.aiondashboard.site/open-source/indian-market-calendar

Indian market calendar for NSE, BSE, and MCX — Python package.

[![PyPI version](https://img.shields.io/pypi/v/aion-indian-market-calendar)](https://pypi.org/project/aion-indian-market-calendar/)
[![Python](https://img.shields.io/pypi/pyversions/aion-indian-market-calendar)](https://pypi.org/project/aion-indian-market-calendar/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](https://opensource.org/licenses/MIT)
[![Downloads](https://img.shields.io/pypi/dm/aion-indian-market-calendar)](https://pypi.org/project/aion-indian-market-calendar/)

## What changed in 1.5.1 (18 October 2026)

SEBI revised the Closing Auction Session on 15 October 2026 (circular
HO/47/11/11(3)2025-MRD-POD2/I/23843/2026, forwarded by NSE as NSE/CMTR/76963). From
**7 December 2026**:

| Segment | Until 2026-12-06 | From 2026-12-07 |
|---|---|---|
| `NSE_EQUITY_FNO_UNDERLYING`, `BSE_EQUITY_FNO_UNDERLYING` | continuous to 15:15, auction 15:15 to 15:35 | continuous to **15:30**, auction **15:30 to 15:40** |
| `NSE_EQUITY_DERIVATIVES`, `BSE_EQUITY_DERIVATIVES` | to 15:40 | to **15:45** (expiring index contracts stop at 15:30 on expiry day) |
| `NSE_EQUITY`, `BSE_EQUITY` and the 15:50 to 16:00 post-close | unchanged | unchanged |

Dates before 7 December keep the August timings. NSE commodity non-agri contracts trade to 23:55
from 2 November 2026 to 12 March 2027. Installs of 1.4.x receive the timing rules through the
hash-verified data refresh. 1.5.1 also corrects the PyPI description and links that 1.5.0 published
out of date. Full history: [CHANGELOG.md](CHANGELOG.md).

## Install

```bash
pip install aion-indian-market-calendar
```

**PyPI is the only installation source.** This repository is documentation-only — see
[Documentation Boundary](#documentation-boundary).

## What It Does

`aion-indian-market-calendar` is a Python package for Indian market calendar workflows across NSE,
BSE, and MCX. It is designed for developers who need reliable Indian exchange session checks,
trading-day checks, holiday calendars, and market-open validation in Python.

It is built for these developer search intents:

- Indian market calendar Python
- NSE market calendar Python
- MCX trading calendar Python
- BSE trading calendar Python
- India trading holidays API
- Python NSE trading calendar
- market open India Python
- NSE holiday list Python
- Indian exchange session times Python
- MCX evening session calendar Python
- Muhurat trading calendar API
- Closing Auction Session Python
- Python library for Indian stock market holidays
- How to check if NSE market is open in Python
- aion-indian-market-calendar vs pandas_market_calendars
- Why pandas_market_calendars fails for Indian exchanges

## Closing Auction Session — from 2026-08-03

Per **SEBI circular HO/47/11/11(3)2025-MRD-POD2/I/2765/2026 dated 16 January 2026**, the cash segment
no longer has a single closing time. It splits by whether a security has derivative contracts, and it
applies to NSE and BSE alike.

| Segment | Continuous trading | Closing Auction | Effective |
|---|---|---|---|
| `NSE_EQUITY_FNO_UNDERLYING` | 09:15 – **15:15** | **15:15 – 15:35** | 2026-08-03 |
| `BSE_EQUITY_FNO_UNDERLYING` | 09:15 – **15:15** | **15:15 – 15:35** | 2026-08-03 |
| `NSE_EQUITY` / `BSE_EQUITY` (no F&O contracts) | 09:15 – 15:30 | — | unchanged |
| `NSE_EQUITY_DERIVATIVES` / `BSE_EQUITY_DERIVATIVES` | 09:15 – **15:40** | — | 2026-08-03 |

The post-close session moves to **15:50 – 16:00**. Index derivatives — NIFTY, BANKNIFTY, SENSEX,
BANKEX — have no auction and trade continuously to 15:40.

`is_market_open` stays **True** during the auction, because the market is still operating. Code that
places ordinary orders should gate on **`is_continuous_trading`**:

```python
from datetime import datetime
from aion_indian_market_calendar import IndiaMarketCalendar

cal = IndiaMarketCalendar.bundled(2026)
when = datetime(2026, 8, 3, 15, 20)

cal.is_market_open(when, market="NSE_EQUITY_FNO_UNDERLYING")         # True  — auction running
cal.is_continuous_trading(when, market="NSE_EQUITY_FNO_UNDERLYING")  # False — not continuous
cal.is_continuous_trading(when, market="NSE_EQUITY")                 # True  — non-F&O unaffected
cal.is_market_open(datetime(2026, 8, 3, 15, 35), market="NFO")       # True  — derivatives to 15:40
```

Dates before 2026-08-03 return the old timings, so backtests over historical dates stay correct.

## Quick Start

```python
from datetime import datetime
from aion_indian_market_calendar import is_market_open, is_continuous_trading, next_trading_day

# Top-level helpers — default to now() in IST
is_market_open("NSE")
is_continuous_trading("NSE")
next_trading_day("NSE")

# Explicit datetime
is_market_open("NSE", at="2026-08-03T10:00:00+05:30")
is_market_open("MCX", at=datetime(2026, 8, 3, 20, 0))
```

## Calendar Object

```python
from datetime import date, datetime
from aion_indian_market_calendar import IndiaMarketCalendar

cal = IndiaMarketCalendar.bundled(2026)
dt = datetime(2026, 8, 3, 10, 0)
day = date(2026, 8, 3)

cal.is_market_open(dt, market="NSE")            # bool
cal.is_continuous_trading(dt, market="NSE")     # bool — False during a closing auction
cal.is_trading_day(dt, market="NSE")            # bool — ignores time of day
cal.get_session(dt, market="NSE")               # list[SessionSegment] | None
cal.closing_auction_session(dt, market="NSE")   # SessionSegment | None
cal.active_session_rule(day, market="NSE")      # SessionRule | None
cal.holidays(market="NSE", year=2026)           # list[date]
cal.events_on("2026-11-08", exchange="NSE")     # list[MarketEvent]
cal.covers(date(2027, 1, 26))                   # bool — is this date inside the bundled range?
```

`SessionSegment` carries `market`, `open`, `close`, and `kind` (`"continuous"` or
`"closing_auction"`), plus an `is_continuous` convenience property.

## Supported Market Inputs

All inputs normalise to canonical segments internally. Unknown inputs raise `ValueError`.

| Input | Resolves to |
|---|---|
| `NSE`, `NSE_CASH` | `NSE_EQUITY` |
| `NSE_CASH_FNO`, `FNO_UNDERLYING` | `NSE_EQUITY_FNO_UNDERLYING` |
| `NFO`, `FNO`, `NIFTY`, `BANKNIFTY` | `NSE_EQUITY_DERIVATIVES` |
| `BSE`, `BSE_CASH` | `BSE_EQUITY` |
| `BSE_CASH_FNO`, `BSE_FNO_UNDERLYING` | `BSE_EQUITY_FNO_UNDERLYING` |
| `BFO`, `BSE_FNO`, `SENSEX`, `BANKEX` | `BSE_EQUITY_DERIVATIVES` |
| `CDS`, `USDINR`, `EURINR`, `GBPINR`, `JPYINR` | `NSE_CURRENCY_DERIVATIVES` |
| `MCX`, `MCX-COMMODITIES` | `MCX` |

Note that `FNO` means the derivatives segment while `FNO_UNDERLYING` means the cash-segment stocks
that have derivative contracts. From 2026-08-03 they close at different times.

The package does not ship the list of which symbols have derivative contracts — that changes on
exchange review and belongs in your instrument master. Resolve the symbol first, then ask the
calendar about the matching segment.

## MCX Evening Session

There is no separate `MCX_EVENING` market. MCX is a two-segment day on one calendar, so pass the
time you care about:

```python
from datetime import datetime
from aion_indian_market_calendar import IndiaMarketCalendar

cal = IndiaMarketCalendar.bundled(2026)

cal.is_market_open(datetime(2026, 6, 18, 20, 0), market="MCX")   # True — evening session
cal.is_market_open(datetime(2026, 3, 3, 10, 0), market="MCX")    # False — Holi morning is shut
cal.is_market_open(datetime(2026, 3, 3, 18, 0), market="MCX")    # True  — evening still runs
```

This is the MCX behaviour generic calendars miss: on 11 of the 16 equity holidays the MCX morning is
closed but the **evening session runs 17:00 – 23:30**, and `is_trading_day` is `True`.

## Muhurat Trading

Diwali Laxmi Pujan falls on **Sunday 8 November 2026** and the ceremonial Muhurat session runs that
day, so the Sunday is a trading day:

```python
from datetime import datetime
from aion_indian_market_calendar import IndiaMarketCalendar

cal = IndiaMarketCalendar.bundled(2026)

cal.is_trading_day(datetime(2026, 11, 8, 18, 30), market="NSE")   # True
cal.is_market_open(datetime(2026, 11, 8, 12, 0), market="NSE")    # False — only the window trades
[e.name for e in cal.events_on("2026-11-08", exchange="NSE")]
# ['NSE Muhurat Trading (Diwali Laxmi Pujan)']
```

The **date is confirmed**; the 18:15 – 19:15 window is the long-standing convention and is
**provisional** until the exchange circular is published, flagged as
`metadata.timings_pending_exchange_circular`.

## Holiday Calendars By Segment

Segments do not share one holiday list:

| Segment | 2026 holidays | Relationship |
|---|---|---|
| `NSE_EQUITY`, `NSE_EQUITY_DERIVATIVES`, all `BSE_*` | 16 | identical — NSE and BSE observe the same equity trading holidays |
| `NSE_CURRENCY_DERIVATIVES` (CDS), `NSE_INTEREST_RATE_DERIVATIVES`, `NSE_CORPORATE_BONDS` | 20 | the 16 equity holidays **plus 4 bank holidays** |
| `MCX`, `NSE_COMMODITY_DERIVATIVES` | 4 full | 4 full closures, plus 11 evening-only days |

```python
from aion_indian_market_calendar import IndiaMarketCalendar

cal = IndiaMarketCalendar.bundled(2026)
cal.holidays(market="NSE", year=2026)   # 16 dates
cal.holidays(market="CDS", year=2026)   # 20 dates
```

The four extra bank-linked holidays are Chatrapati Shivaji Maharaj Jayanti (19 Feb), Gudi Padwa
(19 Mar), **Annual Bank Closing (1 Apr)**, and Id-E-Milad (26 Aug). These segments settle through
banks, so they close while equity trades normally. Annual Bank Closing is the trap: equity is open,
CDS is not.

**1 January is the inverse of the MCX pattern** — the commodity morning session trades and the
*evening* is shut, tracking global commodity markets.

The 2026 dates were audited against the published NSE, BSE and MCX calendars on 2026-08-01; all 16
equity dates matched exactly.

## Data Coverage

The bundled data covers a fixed range. Outside it there are no holiday records, so answers degrade to
weekday-only logic and a warning is logged:

```python
from datetime import date
from aion_indian_market_calendar import IndiaMarketCalendar

cal = IndiaMarketCalendar.bundled(2026)
cal.coverage_start              # date(2026, 1, 1)
cal.coverage_end                # date(2026, 12, 25)
cal.covers(date(2027, 1, 26))   # False — do not trust an answer for this date
```

2027 data is not yet bundled; exchanges publish the following year's holiday list late in the
preceding year.

## Why Not pandas_market_calendars?

Generic market-calendar packages do not cover the full Indian exchange reality. Three gaps cause
silent failures:

1. **MCX** — no MCX calendar at all, so evening sessions and evening-only holiday days are invisible.
2. **Currency derivatives** — XNSE gives NSE equity hours (09:15–15:30). CDS closes at 17:00 and has
   four extra bank holidays, so both close times and holiday answers come out wrong for a USDINR
   workflow.
3. **Muhurat and the Closing Auction Session** — Diwali is marked closed, so the Muhurat session is
   silently skipped; and a single "closes at 15:30" constant is now wrong for two of the three
   cash-side segments, in different directions.

## Live Calendar Refresh

Since 1.4.0 the package refreshes its data at most once a day from
[aion-market-calendar-data](https://github.com/AION-Analytics-India/aion-market-calendar-data). Every
file must match the SHA-256 in the manifest before it is adopted. The data carries events and, since
the 2026-10-18 dataset, `session_rules`, so an effective-dated timing change reaches an installed
1.4.x package with no upgrade. Set `AION_IN_CALENDAR_OFFLINE=1` to use the bundled data only. The
older `refresh_url` path was removed in 1.4.1 (the argument is accepted and ignored).

## Changelog

Release history is in [CHANGELOG.md](CHANGELOG.md), a copy of the changelog in the source repository.

## Documentation Boundary

This repository is **documentation-only**. It exists so developers and LLM search tools can discover
the PyPI package and understand the supported Indian market-calendar workflows.

It does not contain package source code, exchange data files, automation scripts, live override
files, or release machinery. The package is **not** released from here — PyPI is the single source of
installation. See `.gitignore`, which blocks package artefacts from being committed to this repo.

## Issues and Feature Requests

Open an issue in this repository for bug reports and feature requests.

## License

MIT — free to use in commercial and open-source projects.

---

*Part of the AION Analytics open-source ecosystem — Built for Indian markets.*
