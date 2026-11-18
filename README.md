# gta6-twitch-forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Arifpratama19/gta6-twitch-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Arifpratama19/gta6-twitch-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecast how many people watch Grand Theft Auto VI live on Twitch in each
hour of its release day (2026-11-19 UTC), as five quantiles an hour, from
the first hour's viewer print and the launch-day shapes of past big game
launches.

Built for the make.fun bounty [Forecast GTA VI's launch-day Twitch viewers hour by hour](https://make.fun/bounty/gta6-twitch-ccu-forecast-01).

## How it works

`main.py` (Python 3.13, standard library only, no network) reads each
`makefun/twitch-ccu-case/v1` case in `$MAKEFUN_INPUT_DIR` (else `/input`)
and writes `/output/<id>.out` with exactly 23 rows (hours 2–24), five
quantiles each:

1. **Anchor on hour 1.** Every later hour is forecast as a log-ratio to
   hour 1's print, so the same code scales from a 20k launch to a 300k one.
2. **Comparable launch curves.** Each comparable's release day becomes 24
   hourly log-ratios `ln(viewers_h / viewers_1)`. A comparable joins the
   pool only if its hour 1 was already a real audience (≥ 15% of its own
   release-day peak) — launches whose hour 1 was near zero because they
   unlocked later in the day have meaningless ratios and are excluded,
   as is GTA VI's own pre-release category history (not a launch).
3. **Similarity weighting.** Each comparable is weighted by
   `exp(-3 * |ln(r_case / r_comp)|)`, where `r` is hour-1 viewers over
   the previous 24 h peak — launches that "woke up" like this one count
   most.
4. **Empirical quantiles.** The five quantiles per hour are the weighted
   quantiles of the pooled log-ratios, so the spread is whatever the
   comparables actually disagreed by — wide late in the day, narrower in
   the small hours.
5. **Smoothing.** Neighbouring hours are blended (¼, ½, ¼ in log space).

The pre-checkpoint trend is deliberately not extrapolated: the audience
falls into hour 1 (the New Zealand unlock was hours earlier) yet rises
again as Europe and the US unlock, so trend extrapolation made forecasts
worse in validation.

## Validation

Leave-one-out over the kit's 37 comparables, scored only on the 26
rolling-midnight launches whose hour 1 was ≥ 15% of their peak (the shape
this contest asks for), using the shipped `main.py` end to end:

| | pinball loss | typical median miss | 90% coverage |
|---|---:|---:|---:|
| this forecaster | **0.224** | 46.1% | 0.856 |
| flat persistence baseline | 0.284 | — | — |

## Run

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```
