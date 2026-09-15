# myPine — SMC Pro

A production-oriented Smart Money Concepts (SMC) indicator for TradingView Pine Script v6.

## Current implementation

`SMC_Pro.pine` is the primary implementation. The original `zzsp.pine` is preserved as a research/prototype implementation.

### Structure engine
- Confirmed fractal swing detection with configurable fractal length.
- Bullish/bearish BOS.
- CHoCH detection from the prior structural bias.
- Close-confirmed breaks by default, with optional wick confirmation.
- ATR-normalized break buffer to reduce marginal level breaches.
- Explicit protected-swing state and invalidation.
- Bounded swing history to control runtime and object pressure.
- No future-bar lookahead for chart-time structure.

### Price-delivery concepts
- Fair Value Gaps (FVG) with ATR minimum-size filtering.
- FVG fill/expiry lifecycle management.
- Displacement-qualified order blocks.
- Configurable order-block search window and lifecycle.
- Equal-high / equal-low liquidity pools using ATR-normalized tolerance.
- Buy-side and sell-side liquidity sweep detection.
- Protected-level retest tolerance rather than requiring an exact tick touch.
- Basic premium/equilibrium reference.

### Multi-timeframe
- Optional higher-timeframe pivot overlay.
- `request.security(..., lookahead_off)` is used for HTF data.
- HTF events are deduplicated by source pivot timestamp.

### Alerts
Alert conditions are provided for:
- Bullish BOS
- Bearish BOS
- Bullish CHoCH
- Bearish CHoCH
- Buy-side liquidity sweep
- Sell-side liquidity sweep
- Long confluence
- Short confluence

## Accuracy principles

The implementation deliberately avoids treating every wick as a structural break. Structure is stateful: a pivot must first become a confirmed candidate, a break must satisfy the configured confirmation rule, and protected levels remain independent from drawing objects.

The ATR parameters are filters/tolerances, not replacements for market-structure levels. This distinction is important when dealing with slight overshoots around protected prices.

## Known limitations / next production gates

Pine cannot provide a universal definition of SMC. Terms such as order block, CHoCH, BOS, liquidity and inducement vary by methodology. The implementation therefore exposes material thresholds rather than silently encoding one rigid discretionary interpretation.

Before publishing a commercial version, validate the indicator across:
1. FX majors and crosses.
2. Crypto perpetuals and spot markets.
3. Index CFDs/futures where available.
4. Multiple sessions and low-liquidity periods.
5. Multiple chart/HTF combinations.
6. Historical replay and realtime bars.
7. Symbols with unusual tick sizes and gaps.

TradingView compilation and visual replay should be treated as the final acceptance test because Pine execution semantics and platform object limits cannot be fully reproduced outside TradingView.

## Roadmap

- [x] Establish standalone SMC Pro implementation.
- [x] Confirmed fractal structure engine.
- [x] BOS / CHoCH state.
- [x] Protected swing lifecycle.
- [x] FVG lifecycle.
- [x] Displacement-qualified OBs.
- [x] Liquidity pools and sweeps.
- [x] ATR-normalized tolerances.
- [x] HTF confirmed pivot overlay.
- [x] Alert conditions.
- [ ] Add inducement / liquidity-engine classification.
- [ ] Add premium/discount zones tied to active dealing range.
- [ ] Add mitigation/retest state machine for OB/FVG.
- [ ] Add configurable entry models and explicit invalidation rules.
- [ ] Add a separate strategy implementation for quantitative validation.
- [ ] Replay-test and tune defaults by asset class/timeframe.
- [ ] Final TradingView compilation and object/runtime audit.
- [ ] Commercial release hardening and documentation.

## Disclaimer

This is a technical-analysis tool, not financial advice. Signals are model outputs and should be independently validated before use in live trading.
