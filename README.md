# gta6-twitch-ccu-forecast

Forecast GTA VI's launch-day Twitch viewers, hour by hour — an entry for make.fun's
bounty **“Forecast GTA VI's launch-day Twitch viewers hour by hour”**, $250 in USDC.

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/alinsyirahainun-rgb/gta6-twitch-ccu-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/alinsyirahainun-rgb/gta6-twitch-ccu-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)

## What it does

Given the first hour of a launch day — and nothing else about the day itself — it
writes a five-quantile forecast for every remaining hour, 2 through 24, as the kit's
`program/v1` contract asks: `{"hours": [{"hour": n, "q": [q05, q25, q50, q75, q95]}, …]}`.

The scored quantity is a pinball loss on log viewers, so the model works in log
space throughout: a 20% miss costs the same at 10,000 viewers as at 1,000,000, and a
band that is symmetric in ratios is the band that scores best.

## How

1. **Shape from comparable launches.** Every past big console launch shipped with the
   case becomes a curve relative to its own hour 1, `r_h = v_h / v_1`, in natural-log
   space. A comparable is only used when its own first hour is a real print and at
   least 20 of its 23 later hours were sampled.
2. **Size-matched neighbours.** Those curves are weighted by how close their own first
   hour is to ours — a tricube kernel in log viewers over the 20 nearest. The first
   hour of a mega-launch is inflated by the unlock spike, so a launch of our size
   teaches our shape better than one ten times bigger or smaller.
3. **Shape and honest spread.** The weighted median log ratio per hour is the shape; the
   weighted MAD around it (× 1.4826), blended with the spread over every comparable,
   sets the band's width — floored so it is never falsely narrow, capped so a needlessly
   wide band does not pay at every hour, and widened with the hour since the day drifts
   away from the one print we know.
4. **Pre-release tail.** The category's own history before the unlock calibrates the
   level: a category already busy before release peaks at the unlock and decays faster;
   a quiet one builds through the day. The correction fades in over the first eight hours.

The neighbourhood size and the width floors were chosen by leave-one-out
cross-validation over 34 comparable launches, scored with this contest's own pinball
loss: about 0.39 mean loss, against 0.58 for a pooled-median shape with a narrow band.

Standard library only, no network; the grader runs `python3 -P -s /bot/main.py`.

## Run it locally

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Every `<id>.in` in `input/` produces `<id>.out` with 23 rows of five non-decreasing
quantiles.

## Licence

MIT — see `LICENSE`.
