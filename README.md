# USAU + International Player Glicko-2

Static player and club rankings built from the same complete USAU, EUF, UFA, and official WFDF world-championship corpus as [USAU + International Player Elo](https://github.com/sethlnx/usau-player-elo), with Glicko-2 replacing the Elo update.

**[Live rankings](https://sethlnx.github.io/usau-player-glicko/)**

## Current results

The site records each game's prediction from the opening rating of its
tournament day, then applies all of that day's results as one Glicko-2 batch.
The 2024–2025 slice is held out from model design; every earlier game remains
training history.

| Evaluation slice | Games | Accuracy | Log loss | Brier score |
|---|---:|---:|---:|---:|
| All seasons | 164,197 | 74.14% | 0.527502 | 0.175520 |
| Held-out 2024–2025 | 28,607 | 75.27% | 0.513509 | 0.169718 |
| Current 2026 | 10,074 | 76.34% | 0.508217 | 0.167352 |

On the identical held-out corpus, the published Elo model scores 78.08%
accuracy and 0.449024 log loss. Glicko-2 therefore provides native per-player
uncertainty, but it is not the stronger predictor in this implementation.

The period-boundary benchmark scored per-game, per-day, and per-tournament
updates at 76.86%/0.491153, 75.27%/0.513509, and 74.11%/0.530077 respectively
on 2024–2025 (accuracy/log loss). This publication deliberately uses per-day.

Accuracy uses a 0.5 decision threshold. Log loss evaluates the full probability and penalizes confident errors. Ties have outcome 0.5.

## Model contract

- One tournament day is one Glicko-2 rating period. Every game that day is predicted from its opening ratings; all of the day's results are then applied in one batch.
- A team's rating is the softmax-weighted mean of its event roster's player ratings, matching the Elo site's roster contract.
- Team rating deviation is the root-sum-square uncertainty of that weighted mean.
- Inactivity expands rating deviation in 30-day periods. It does not decay the rating itself.
- Published bands are $rating \pm 1.645 \times RD$, a 90% Glicko-2 interval.
- Default player RD is 350, volatility is 0.06, and volatility constraint $\tau$ is 0.5.
- Division debut priors, roster identity, cross-source player bridges, and stat-derived involvement weights match the Elo publication.
- All 19 modeled divisions share one player scale. Cross-source names are merged only through the audited identity rules in the source project.

The browser exposes 130,397 rated players, 62,055 players with at least 30 games, 1,700 current club rows, 164,197 scored games, and 5,464 rating events. The Tournaments tab remains USAU-only because only USAU supplies the bracket structure it renders.

## Rebuild the site

Run from a current checkout of [`usau-player-elo`](https://github.com/sethlnx/usau-player-elo) with its databases populated:

```bash
.venv/bin/python -m analysis.glicko_rankings
RANKINGS_DATA_DIR=data/glicko \
RANKINGS_SITE_OUT=../usau-player-glicko/docs/index.html \
RATING_NAME=Glicko-2 \
.venv/bin/python -m analysis.site
```

`analysis.glicko_rankings` writes the ranking CSVs, trajectories, audits, and `metrics.json` under `data/glicko/`. `analysis.site` turns those files into the static `docs/` application. Push this repository to deploy through its Pages workflow.

The generated `docs/` files are intentionally committed: GitHub Pages serves them without a database or application server.
