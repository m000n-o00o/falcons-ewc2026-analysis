# Data spec - handoff to design (v1, Rocket League)

## About the data
- Source: ballchasing.com, Esports World Cup 2026, Rocket League.
- 23 matches, all involving Team Falcons (FLCN). Falcons won 18 and lost 5.
- Every metric is a team average (the mean of the team's 3 players) for one match.
- Opponents appear only in their matches against Falcons (4-6 matches each). Their numbers describe how they played against Falcons, not their general level.
- A second game (CS2) is planned. The dashboard should allow adding a game later (for example a game switcher), because each game has its own metrics.
- Terminology: a "match" in these files is one game (one replay). The 23 games come from 5 series, one per opponent. Wherever the spec says "match", read it as "game" (the column n_matches counts games).

## Team names
FLCN = Team Falcons, KC = Karmine Corp, M8 = Gentle Mates, R8 = R8 Esports, SSG = Spacestation Gaming, TM = Twisted Minds

## Files (outputs/data_spec_exports/)
|      File        |         One row per      |                    Columns               |
|------------------|--------------------------|------------------------------------------|
| effect_sizes.csv | metric (12 rows)         | metric, effect_size                      |
| team_match.csv   | team per match (46 rows) | replay_id, team_name, won, + 12 metrics  |
| team_summary.csv | team (6 rows)            | team_name, 12 metric averages, n_matches |
| series.csv       | series (5 rows)          | series, opponent, first_game, games_played, falcons_wins, falcons_losses, score |
| games.csv        | game (23 rows)       | replay_id, match_date, series, opponent, falcons_goals, opponent_goals, falcons_won |

- won (in team_match.csv): True if this team won the match, False if it lost. Every match has exactly one winner and one loser.

## Metrics (as defined by Ballchasing)
- boost_bpm: boost used per minute
- avg_speed: average speed
- saves, shots: counts per player, averaged over the team
- boost_stolen: boost collected from the opponent's half
- pct_zero_boost: share of time with no boost
- pct_supersonic: share of time at supersonic speed
- pct_defensive_third / pct_offensive_third: share of time in own / opponent third
- dist_to_mates: average distance to teammates
- demos_inflicted / demos_taken: demolitions caused / suffered (mirror images of each other)

effect_size = (winners mean - losers mean) / pooled std. Positive means winners are higher. About 0.6 would be needed to stand out from chance with 23 vs 23 matches. In plain words: how big the gap between winners and losers is, compared with how much that metric normally varies from match to match.

## View 1: What separates winners from losers?
- Data: effect_sizes.csv
- Suggested chart: horizontal dots centered on zero, with a shaded band for +/-0.6 labeled "within normal variation". Avoid a plain two-bar comparison of means: it would suggest a bigger gap than exists.
- Key message: no metric clearly separates winners from losers (largest is 0.50). Weak hint, a hypothesis and not a conclusion: winners use boost more economically and spend more time in their own half.
- Shots are nearly circular (shots produce goals). Demos are symmetric, so show them once or leave them out.

## View 2: Where do Falcons stand?
- Data: team_summary.csv
- Suggested chart: one row per metric, one dot per team, Falcons highlighted.
- Show n_matches next to each team (Falcons 23, opponents 4-6) and mark opponent values as low-confidence.
- Caveat: opponent numbers come from matches against Falcons only.
- Each metric has its own scale (boost_bpm is around 450, saves around 2), so every row needs its own axis. Do not color values as good or bad: higher is not always better (for example pct_zero_boost).

## View 3: Series and games
- Data: series.csv (5 rows, one per opponent) and games.csv (23 rows)
- Suggested layout: a list of the 5 series with the opponent and the score (for example 3-2), each one expandable to its games (date, score, result).
- Games inside a series are not independent of each other, and each opponent appears in one series only.
- Stage of each series: R8 and M8 = group stage, TM = quarter-final, KC = semi-final, SSG = grand final. Display the opponent and stage instead of the technical series id.

## Not included yet
CS2.