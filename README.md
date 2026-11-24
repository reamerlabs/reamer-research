# Reamer Research

A backtesting and live trading engine for Python and C++, from
[Reamer Labs](https://reamerlabs.com). Install it with pip or include one
C++ header. Write a strategy as a function or a class, backtest it with
tick-level fills on a seeded path inside each bar, check every trade in
visual trade diagnostics, then send the same orders live through your own
broker.

**[PyPI](https://pypi.org/project/reamer-research/)** ·
**[Download](https://reamerlabs.com/download)** ·
**[Docs](https://reamerlabs.com/docs)** ·
**[Examples](https://github.com/reamerlabs/reamer-research-examples)** ·
**[MCP server](https://github.com/reamerlabs/reamer-research-mcp)** ·
**[Pricing](https://reamerlabs.com/pricing)**

Linux x86-64 · macOS on Apple silicon · Windows through WSL2 · Python 3.9+ · C++20

## Install

```bash
pip install reamer-research
```

The Python package is on [PyPI](https://pypi.org/project/reamer-research/),
with the compiled engine and the visual trade diagnostics tool inside it.
No key and no sign-up: without a licence it runs in Evaluation Mode.

## First run

```bash
pip install reamer-research
git clone https://github.com/reamerlabs/reamer-research-examples
cd reamer-research-examples/python
python3 examples/quickstart.py
```

To backtest from Claude Code, Claude Desktop or Cursor, the
[MCP server](https://github.com/reamerlabs/reamer-research-mcp) installs the
engine with itself: `claude mcp add reamer-research -- uvx reamer-research-mcp`.

## The full kit

For C++, and to activate a licence with `reamer-license`. The links below
always serve the latest release, straight from Reamer Labs.
No key and no sign-up.

| Platform | Kit |
|---|---|
| Linux x86-64, Windows through WSL2 | [Download the Linux kit](https://licenses.reamerlabs.com/evaluation/research/linux_x86_64) |
| macOS 11 or later, Apple silicon | [Download the macOS kit](https://licenses.reamerlabs.com/evaluation/research/macosx_11_0_arm64) |

Every file, with its SHA-256 checksum and signature, is listed at
[licenses.reamerlabs.com/evaluation/research](https://licenses.reamerlabs.com/evaluation/research).
Each kit holds the C++ header and library, the `reamer-license` tool, the
same Python wheel as PyPI, the visual trade diagnostics tool, example
strategies, sample data and the documentation.

## A strategy

ATR band reversion with a take profit and a stop loss on every entry, on
the sample bars from the examples repository:

```bash
curl -LO https://raw.githubusercontent.com/reamerlabs/reamer-research-examples/main/python/sample_data/AAPL.csv
```

```python
import numpy as np
import reamer_research as rr

def on_bar(d):
    c, h, l = d.close, d.high[1:], d.low[1:]
    if c[0] == 0: return                       # warm-up: the window is still filling
    atr = (np.maximum(h, c[:-1]) - np.minimum(l, c[:-1])).mean()
    mid, px = c.mean(), c[-1]
    if d.position() == 0 and px < mid - 2 * atr:
        return rr.buy_market(10, take_profit=mid, stop_loss=px - 2 * atr)
    if d.position() == 0 and px > mid + 2 * atr:
        return rr.sell_market(10, take_profit=mid, stop_loss=px + 2 * atr)

print(rr.backtest(on_bar, "AAPL.csv", lookback=20, open_viewer=True))
```

The same strategy in C++20:

```cpp
#include <reamer_research.hpp>
#include <algorithm>
#include <iostream>
namespace rr = reamer_research;

int main() {
  auto r = rr::backtest([](rr::Data& d) {
    auto c = d.close(), h = d.high(), l = d.low();
    if (c[0] == 0) return;                // warm-up: the window is still filling
    double atr = 0;
    for (size_t i = 1; i < c.size(); ++i)
      atr += std::max(h[i], c[i - 1]) - std::min(l[i], c[i - 1]);
    atr /= double(c.size() - 1);
    double mid = c.mean(), px = c.back();
    if (d.position() == 0 && px < mid - 2 * atr) d.buy_market(10).bracket(mid, px - 2 * atr);
    if (d.position() == 0 && px > mid + 2 * atr) d.sell_market(10).bracket(mid, px + 2 * atr);
  }, "AAPL.csv", {.lookback = 20, .open_viewer = true});
  std::cout << r << '\n';
}
```

```bash
c++ -std=c++20 -O2 atr.cpp -I$KIT/include -L$KIT/lib -lreamer_research -Wl,-rpath,$KIT/lib -o atr
```

CMake `find_package(ReamerResearch)` and pkg-config `reamer-research` also work.

## Features

| Area | Feature |
|---|---|
| Languages | A Python package on PyPI (`pip install reamer-research`); one C++20 header; a stable C ABI for Rust, Go, Julia, C# or any language that calls C. |
| Strategies | A function, a class or an object with `on_bar`. |
| Data | CSV, pandas DataFrames, NumPy arrays and memory-mapped `.bin` files. Bars of any type: time, tick, volume, dollar or your own rule. Multi-ticker on one clock. Alpaca and Databento fetchers. |
| Execution | Each bar replayed as a seeded tick path with bid, ask, spread and slippage. Market, limit and stop orders, brackets, time in force. Spread, slippage, commission, swap and margin per ticker, under a [public execution spec](https://reamerlabs.com/docs/execution-spec). |
| Results | Visual trade diagnostics: every closed trade on its own bars, beside every value the strategy printed. A JSON report with 31 metrics. Monte Carlo. Byte-identical runs for the same seed. |
| Live | The relay sends the orders `on_bar` returns to an [open-source receiver](https://reamerlabs.com/docs/research-live) that places them with your broker. |

## Speed

In [What a Backtest Costs](https://reamerlabs.com/papers/what-a-backtest-costs.pdf),
on one AMD EPYC 9575F with Reamer Research 4.6.0, one 500,000-bar backtest
of a 20-bar breakout took 3.38 s from Python and 0.36 s from C++. On the
same strategy and data, Backtrader took 15.04 s, QuantConnect LEAN
22.10 s and NautilusTrader 76.85 s as installed. Every runner and the raw
results are in the
[benchmark repository](https://github.com/reamerlabs/research-engine-benchmark).

## Evaluation Mode and pricing

Without a licence, each backtest runs on one ticker over at most 30 days or
50,000 bars, whichever comes first, and live trading is off. A run past a
limit stops with an error that names the limit. Monte Carlo and visual
trade diagnostics work within the limits.

A [Reamer Research licence](https://reamerlabs.com/pricing) is $20 per seat
per month, cancel any time, and removes every limit. The same code runs
unchanged.

## Porting from another engine

- [From Backtrader](https://reamerlabs.com/faq/port-backtrader-strategy)
- [From NautilusTrader](https://reamerlabs.com/faq/port-nautilustrader-strategy)
- [From QuantConnect LEAN](https://reamerlabs.com/faq/port-quantconnect-lean-algorithm)

## Licence

This repository holds documentation only. The engine is commercial software
under the Reamer Labs Software License Agreement, `LICENSE.txt` in each
kit and in the wheel, whose Section 2B grants Evaluation Mode without payment or a key.
Install it only from [PyPI](https://pypi.org/project/reamer-research/), the
links above or [reamerlabs.com/download](https://reamerlabs.com/download). Terms:
[reamerlabs.com/terms](https://reamerlabs.com/terms).

## Support

Bug reports, questions and feature requests: open an
[issue](https://github.com/reamerlabs/reamer-research/issues). Licence keys,
billing and anything private: support@reamerlabs.com, answered on a
best-effort basis. All Reamer Labs repositories:
[github.com/reamerlabs](https://github.com/reamerlabs).
