# GTA VI launch-day Twitch CCU forecaster — gta6-twitch-ccu-forecast-01

Entry for the make.fun bounty `gta6-twitch-ccu-forecast-01` ($250, deadline
2026-11-19 01:00 UTC). Forecasts, for hours 2–24 of GTA VI's release day
(2026-11-19 UTC), five quantiles (5/25/50/75/95%) of Twitch concurrent
viewers in the "Grand Theft Auto VI" category, from the case's hour-1 print.

Built Muse-only (no Hermes draft: the VPS link was down). No submission made
from this folder — submit is handled in the main chat in the next turn.

## Files

- `main.py` — the entrypoint (Python 3.13, standard library only). Run:
  `MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py`
- `README.md` — this file.
- `LICENSE` — MIT.

## Model

Analog / conformal quantile forecast over the past big launches shipped in
each case file (`comparables`, 38 in practice cases, 42 in the final):

1. Each comparable launch contributes its release-day shape as log-ratios
   `ln(viewers_h / viewers_1)`, h = 2..24, anchored on its own hour-1 print.
   Usable donors only: a complete 24-hour release day, hour-1 viewers >= 15%
   of the day's max and >= 3,000. That drops simultaneous-global-unlock
   launches whose hour 1 is still pre-unlock (ratios of 100x–2000x) and tiny
   noisy games.
2. Donors are weighted by launch-size closeness:
   `w = exp(-((ln p1_target - ln p1_donor) / 1.2)^2)`. Launch size predicts
   shape: mega-launches (Elden Ring, Cyberpunk 2077, ~850k in hour 1) stay
   near flat (peak/hour-1 ~1.0–1.3); mid-size launches grow 2–3x into the
   evening as the US wakes. The kernel adapts to whatever hour-1 print GTA VI
   actually posts. Early-access-contaminated donors count half (their hour 1
   is inflated by days of early access).
3. Per hour, the five quantiles are the weighted quantiles of the donors'
   log-ratios, smoothed over ±1 hour. `q = p1 * exp(quantile)`. The
   cross-launch spread is the honest uncertainty: it widens toward the
   evening, and the US unlocks (05:00 UTC East, 08:00 UTC West) appear as the
   median curve climbing out of the 01:00–05:00 UTC dip toward the evening
   peak (~18:00–20:00 UTC).
4. No usable donors → fall back to the house baseline (persistence, 0.15
   log-spread per sqrt hour), so the output is always valid.

## Validation (leave-one-out, contest's own log-scale pinball loss)

Over the 27 usable launches in the kit's practice cases:

| model | mean pinball loss |
|---|---|
| this model | **0.224** |
| house baseline | 0.297 |

Beats the baseline on 23 of 27 launches. On the 4 console-only,
rolling-midnight launches (the closest GTA VI analogues: Ghost of Yōtei,
Zelda: Tears of the Kingdom, FFVII Rebirth, The Last of Us Part II) it scores
0.195 vs baseline 0.301. Restricting donors to console-only launches was
tested and is *worse* (0.257): the size kernel already picks the right
neighbours, and small console-only donors over-forecast big launches'
evenings. A pre-launch momentum term was tested and dropped (no gain).

Note: the house states the baseline scores ~0.387 on the actual practice
cases (harder targets than the average comparable), so the practice
leaderboard number will differ; the relative improvement is what was tuned.

## Checks run locally

- 4 kit practice cases (p00–p03): valid output, 23 rows, hours 2–24,
  5 non-decreasing quantiles in (0, 1e9], ~0.05 s per case (limit 8 s).
- Synthetic final-like case (minute reads, hour-1 print 850,000, streams +
  categories present): valid; median curve dips to 0.50x at hour 10, peaks
  0.98x at hour 18, band widens through the day. Deterministic across runs.
- Edge cases: no comparables → baseline fallback; no hour-1 print → last
  pre-checkpoint hourly mean; tiny hour-1 print (1,200) → sane curve.
  All outputs byte-identical across repeated runs.

## Review

Xingchi review via Codex CLI was attempted for both models and failed:
`gpt-6.1-sol` and `gpt-6-astra` both return 403 ACCESS_DENIED from
api.xingchiapi.com ("Your IP is ...") — the relay blocks this VM's egress IP.
No review was obtained; the build was validated by the LOO tests above
instead. Do not block submission on this.
