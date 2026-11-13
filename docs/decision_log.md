# Decision log

A running log of small decisions made during data collection and cleaning.
Add a new entry every time you make a choice that affects the data —
excluding a match, renaming a column, handling a missing value, etc.
Oldest entries at the top; add new entries at the bottom.

## Template (copy this for each new entry)

```
## [date] — [short title]

**Decision:** what you did exactly
**Reason:** why you did it this way
**Effect:** what changed in the data (if anything)
```

---

<!-- Add entries below this line, newest first -->

## 2026-09-12 — Discovering group structure & team names

Decision: Used get_groups(group=id) instead of get_group(id).get("children") to fetch sub-groups

Reason: get_group only returns aggregated stats, it doesn't actually contain a children field

Effect: Found 18 leaf groups; team filtering was done using "flcn" instead of "falcon" (the actual name is abbreviated in the data)

---------------
## 2026-09-19 — Converting match_date to datetime

Decision: Converted the match_date column from text (str) to datetime

Reason: If I later did analysis or built charts in Python, it wouldn't recognize these as dates and couldn't treat them accordingly — it would just see them as text, which could cause an error, or worse, a silent logic error I wouldn't even notice

Effect: The match_date column's data type is now datetime

---------------
## 2026-09-26 — Deriving match winner and Falcons win/loss flag

Decision: Built a "winner" column using groupby(["replay_id","team_name"])["goals"], stored the result in a new wide table (matches_wide), then merged the "winner" column back into the main long table using merge on "replay_id"

Reason: Needed the actual winning team per match to answer the research question.
For readability, also added a "won" column (True/False) showing whether Falcons specifically won each match, 
using a direct == comparison between "team_name" and "winner"

Effect: matches now has a "won" column ready for EDA (comparing Falcons' stats in wins vs losses)

---------------
## 2026-10-03 — Correction to the 2026-09-26 entry

Correction: The 09-26 entry described "won" as whether Falcons won the match. In reality "won" compares team_name with winner, so it is True on every row of the winning team and False on every row of the losing team (Falcons or the opponent).
Reason: Noticed while building team_match, which has 23 True and 23 False rows.
Effect: No code change, only the description. The 09-26 entry is left as written.

---------------
## 2026-10-03 — Team summary with match counts

Decision: Built team_summary (average of each metric per team) with an n_matches column.
Reason: Falcons have 23 matches, but each opponent has only 4-6, and only against Falcons. Without n_matches the comparison would look more reliable than it is.
Effect: Opponent averages describe how they played against Falcons, not their general level. This is noted in data_spec.md.

---------------
## 2026-10-03 — Excluding goals and using standardized differences

Decision: Left goals out of the compared metrics, and measured each metric as (winners mean - losers mean) / pooled std instead of comparing means only.
Reason: "won" is derived from goals, so comparing goals would be circular. Means alone do not show whether a gap is large compared with normal variation; about 0.6 is needed to stand out from chance with 23 vs 23 matches.
Effect: The largest difference is 0.50 (pct_defensive_third). No metric clearly separates winners from losers, so the findings are written as hypotheses.

---------------
## 2026-10-03 — Unit of analysis: team per match, all teams

Decision: Moved from player rows (138) to team-per-match rows (46) by averaging each team's 3 players, and compared winners vs losers across all teams instead of Falcons only.
Reason: The match is what is won or lost, and teammates in one match share the same conditions, so player rows are not independent. Falcons alone have only 5 losses, which is too few; all teams give 23 winners vs 23 losers.
Effect: team_match has 46 rows (23 True, 23 False). Every match still involves Falcons, because collection was filtered to Falcons matches.

---------------
## 2026-10-03 — Re-collected the data with 8 tactical columns

Decision: Added 8 tactical columns to the collection step in notebook 01 (boost_stolen, pct_zero_boost, pct_supersonic, pct_defensive_third, pct_offensive_third, dist_to_mates, demos_inflicted, demos_taken) and re-ran 01, 02 and 03.
Reason: The first metrics (boost_bpm, avg_speed, saves, shots) showed almost no difference between winners and losers, and the research question is tactical. The replay data had richer boost, positioning and demo fields.
Effect: The raw CSV was overwritten (same 138 rows, now 20 columns), and the cleaned file was regenerated with no missing values in the new columns.

---------------
## 2026-10-03 — Handed off v1 (Rocket League) to Rahmah

Decision: Sent Rahmah data_spec.md and the data_spec_exports folder (effect_sizes.csv, team_match.csv, team_summary.csv) as one zip.
Reason: The v1 analysis and the data spec were complete, so I handed off a week earlier than planned (the plan said 2026-10-10).
Effect: Rahmah can start the dashboard design. Not included yet: the match list (date, opponent, result) and CS2 (planned as the second part).

---------------
## 2026-10-03 — Games vs series, and match list tables

Decision: Added games.csv (one row per game) and series.csv (one row per opponent series), and defined "match" in the data spec as one game. Sent Rahmah an updated package.
Reason: Each replay is one game, and the 23 games come from 5 series (one per opponent), so calling them matches was inaccurate. A coach reads results per series, for example 3-2.
Effect: Falcons' record is 18 wins and 5 losses in games. Games inside a series are not independent of each other. data_spec.md was updated and re-sent.

---------------
## 2026-10-03 — Kept v1 scoped to Falcons games

Decision: Did not widen collection to all tournament games (about 86 in Group B and the playoffs); v1 stays as delivered (23 Falcons games, 5 series).
Reason: The project is about Falcons; widening would turn it into a study of the whole tournament.
Effect: The winners-vs-losers comparison has only 5 Falcons losses, so the result stays "no clear difference". Opponent figures come only from games against Falcons (selection bias). Both limits are written in data_spec.md and the README.

---------------
## 2026-11-13 — Confirmed boost_stolen is a per-game average

Decision: Confirmed that boost_stolen is comparable across teams.
Reason: Rahmah asked whether it is a per-game value or a total over the series. Ballchasing gives amount_stolen per player per replay; notebook 03 averages the team's 3 players per game, and team_summary averages those games.
Effect: No code change. Caveat: it is a raw per-game amount, not divided by game length, so overtime games inflate it slightly.

---------------
## 2026-11-13 — Received and reviewed the v1 dashboard

Decision: Reviewed Rahmah's v1 dashboard (Power BI) page by page and checked its data against the exports, then sent her the review notes.
Reason: To confirm the dashboard shows the data correctly before it goes into the portfolio.
Effect: All 72 team_summary values, the 12 effect sizes, the 23 games and the 46 team-game rows match the exports exactly. Notes sent: sort the series table by tournament order (series_order), show that Falcons figures cover all 23 games while each opponent's cover only its games against Falcons, label the effect-size chart as all teams (46 team-games), show absolute gaps next to Gap %, and add source links to About the data. Still open: verify the "excluding middle boosts" wording in the Boost stolen definition against Ballchasing.

---------------
