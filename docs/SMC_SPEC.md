# SMC Pro — Structural Specification

This document defines the non-discretionary rules used by the state-driven engine. It exists to prevent individual conditions from silently changing the meaning of BOS, protected swings, and retests.

## 1. Confirmed swing

A swing high/low is created only after `pivotLen` bars have elapsed on the right side of the pivot. The pivot's timestamp/index refer to the actual pivot candle, not the confirmation candle.

## 2. BOS

A BOS occurs when the confirmed bar satisfies the configured break rule against the nearest eligible unbroken swing of the relevant type:

- Close mode: `close > level + ATR × breakBuffer` for bullish breaks.
- Close mode: `close < level - ATR × breakBuffer` for bearish breaks.
- Wick mode uses the corresponding high/low instead of close.

A broken or invalidated swing cannot be broken again.

If both sides qualify on the same confirmed candle, the engine resolves the conflict deterministically using the older structural candidate instead of allowing loop order to decide the resulting bias.

## 3. CHoCH

A bullish break is CHoCH when the prior structural bias is bearish. A bearish break is CHoCH when the prior structural bias is bullish. The first directional break from neutral is BOS, not CHoCH.

## 4. Protected swing

After a bullish structural break, the protected swing is the most recent confirmed, non-invalidated low that existed before the break. After a bearish break, it is the analogous high.

The protected level is a structural price. ATR is used only as tolerance/invalidation policy; it does not move the structural level.

## 5. Retest state machine

A retest setup is created only after a BOS/CHoCH event establishes a protected level.

States:

- `0`: no active retest setup.
- `1`: bullish protected-level retest pending.
- `-1`: bearish protected-level retest pending.

A BOS candle itself cannot count as a retest.

### Bullish retest

A bullish setup can register a touch when:

`low <= protectedLevel + tolerance`

and

`low >= protectedLevel - invalidation`

This intentionally accepts a price that is slightly **above** the protected price. The exact protected level remains unchanged.

A valid rejection requires the confirmed close to finish above the protected level when rejection is enabled.

### Bearish retest

Mirror the bullish rules:

`high >= protectedLevel - tolerance`

and

`high <= protectedLevel + invalidation`

with a rejection close below the protected level.

A setup expires after `retestMaxBars` bars or is invalidated when price closes beyond the invalidation boundary.

## 6. FVG

A bullish FVG exists when the current low is above the high from two bars earlier and the gap is at least `ATR × minimumFvgSize`. Bearish FVG is the inverse.

FVGs have bounded storage and explicit fill/expiry lifecycle.

## 7. Order block

An order block is created only from a structural displacement candle that exceeds the configured ATR threshold. The engine searches backward for the most recent opposite-colored candle within the configured lookback.

This is intentionally a conservative baseline definition. Future versions may add body-only, wick-inclusive, mitigation, and displacement-quality modes.

## 8. Liquidity

Equal highs/lows are identified using an ATR-normalized tolerance and an age limit. A sweep requires price to trade through the pool and close back on the originating side.

A swept pool is one-shot and is not repeatedly signaled.

## 9. Repainting policy

Chart-time structure and state transitions execute only on confirmed bars. Pivot offsets point to the actual historical pivot candle, while the event itself occurs on its confirmation bar.

Final acceptance still requires compilation and replay testing in TradingView because platform-specific Pine execution/object semantics cannot be fully reproduced by this repository tooling.

## 10. Required test matrix

Before commercial release, replay at minimum:

- FX: EURUSD, GBPUSD, USDJPY.
- Crypto: BTCUSDT and ETHUSDT.
- Index: a liquid major index future/CFD available to the user.
- Timeframes: 1m, 5m, 15m, 1h, 4h.
- Regimes: trend, range, high volatility, low volatility, session open, session overlap, gaps.
- Both close and wick break confirmation.
- Retest tolerance values around 0, 0.10, 0.20 and 0.30 ATR.

The acceptance criteria are deterministic state transitions, no duplicate BOS for the same swing, no pre-break retest, correct invalidation, bounded object counts, and no historical/realtime divergence caused by unconfirmed data.
