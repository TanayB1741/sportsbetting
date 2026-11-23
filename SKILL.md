---
name: "nba-prop-probabilities"
description: "Estimate the probability of NBA player prop overs and unders (points, rebounds, assists, combos) for a given day using 4-season per-minute history, current role/minutes, shot-zone profiles, and opponent defense. Use whenever the user pastes PrizePicks/DFS line screenshots."
---

# NBA prop probabilities

Use this whenever the user pastes screenshots of NBA player prop lines (usually PrizePicks), or asks which NBA props are most likely to hit today. The user wants this done every day they paste new lines, without re-asking: score every line on the board they send, not just a sample, and consider both sides (More/over and Less/under). If they send more screenshots mid-run, add those lines to the same run. If no screenshots are sent, ask for them (props are usually posted by mid-morning, not overnight).

Always pull fresh data. Never reuse numbers from an earlier session: rosters, roles and minutes change constantly. There is no PrizePicks connector; the screenshots are the source of lines.

## 0. Grade the last run first (every day, before the new board)

The user wants the hit rate to improve over time. Before scoring a new board, grade every pick from the previous run's table (all three columns), then fix the model.

1. **Results:** `fetch_sports_data` `game_stats` for each game on the previous slate. Its player lists are partial, so for anyone missing, WebSearch "<player> <opponent> <date> box score" and read the ESPN or CBS box score. Never guess a stat line.
2. **Grade:** mark each pick hit / miss / push (whole-number lines) / void (player sat). Also grade the lines you excluded, to check whether the exclusions helped.
3. **Diagnose each miss** with one cause: minutes (blowout or foul trouble that didn't happen or did), role (a teammate in or out that you didn't account for), matchup multiplier wrong, line inflated by an absence, ordinary variance (within about 1 SD), or bad data. Count variance misses separately; don't tweak the model for them.
4. **Calibration:** group picks by predicted probability (55–65, 65–75, 75–85, 85+). If a bucket hits well below its prediction over several days, widen the SD or shrink toward the line for that pick type.
5. **Fix:** change only what a repeated, non-variance cause supports. Add each durable fix to the "Lessons from graded results" list below and propose the skill update, so the next session starts with it.
6. **Report:** open the new reply with one line, e.g. "Yesterday: 14–9 (61%), goblins 8–2, unders 3–5; fix: ...". Keep it to one or two lines.

## Lessons from graded results

Apply these on every run. Add to this list when grading shows a repeated cause.

- **Role-inflated lines:** when absences push a player's line more than about 20% above his season average (or the line jumped since the previous day), skip his under. The market knows the role changed (Utah role players Nov 11; Fears, Spencer and Coward Nov 13–14 — Spencer and Coward unders would have lost).
- **Blowout trims are unreliable:** 85–89% favorites lost or won close on Nov 14 (MIN lost to CHI; DEN won by 8). Cap the blowout minutes trim at 0.97, and don't rank a pick in the top 3 when its edge comes mainly from expected blowout minutes (bench overs, star unders).
- **Depleted opponents:** when a team rests its stars, its fill-in players get big usage. Don't take unders on them, and boost their overs (Cleveland bench Nov 14: Caleb Martin u9.5 PRA lost by 16 on the other side, and Cleveland's bench beat Dallas).
- **Low lines are noisier:** for a mean under about 10 PRA or 6 points, multiply the SD by 1.25. Small-role players swing more in relative terms than the standard coefficients assume.
- **Low goblins on assists and rebounds** (0.5–2.5) have been the most reliable picks. Rank them on their own merit, not as filler.
- **Whole-number lines:** always compute the push. A 59% under on a line of 5 can really be about 42% win, 17% push.
- **Unders keep losing (Nov 14: 3–5, Nov 15: 1–5):** for standard unders, blend the mean 0.5 model / 0.5 line (overs stay 0.65 / 0.35). Young players with growing roles (Harper, Castle, Bamba, Okogie) beat their lines; the market prices role growth faster than season averages do.
- **Small counts use Poisson:** for rebound or assist lines where the mean is under 4, compute P with Poisson, not the normal curve. The normal curve overstated Edey o0.5 ast at 84%; Poisson gives 62%.
- **Unders went 4–13 from Nov 14 to 16 (0–3 on Nov 16: Coward, Wade, Embiid):** show a standard under only if it is 60% or higher after the role-inflation filter. Never take an under on a team's main scorer, or on a player whose over on another stat is also in the table.
- **Lines set well below average:** when a standard line sits more than 20% below the player's current-season average, the market has priced in a smaller role (Kris Dunn on Nov 17, back to the bench behind Garland and Gradey Dick). Skip his overs unless the last-5 games show the role is back.
- **Promo lines (e.g. PrizePicks Taco Tuesday):** score a promo line as a discounted over (model mean, goblin SD), anchored to the player's normal line for that stat.
- **Regular overs are overconfident (6–4, 4–4, 2–6 from Nov 15 to 17: 12–14 at ~62% predicted):** blend standard overs 0.5 model / 0.5 line, the same as unders. Goblins went 18–6 over the same stretch, so leave them unchanged.
- **Growing roles break the minutes blend (Nov 20):** the 0.7 current / 0.3 last-season minutes blend drags a player's projection down when his current MPG is 25%+ above last season's (Battle, Proctor, Jenkins, Walter, Reed all showed 66-86% unders that were artifacts of this). Skip unders on those players, and skip unders on any team missing 3+ rotation players (Charlotte on Nov 20: Knueppel, Schroder, G. Williams). Nov 19 graded 8-5: regular overs 1-3, goblins 6-1, unders 1-1; the regular-over misses were bench players (Hyland, Wagler, Jones Jr.).
- **Regular overs 0-5 on Nov 20 (1-8 over the last two graded runs):** blend standard overs 0.4 model / 0.6 line, and skip a standard over when its line is more than 12% below the player's season average (the old 20% cutoff let Dort through; the market had already cut his role). Expect few regular overs to clear 58%; leave the column short rather than padding it. Misses were Dort (role), Shead (Quickley returned), and three near coin-flips (Suggs, Mitchell, Nembhard).
- **Shrinking roles break the minutes blend too (Nov 21):** when current MPG is under 75% of last season's, project minutes from current MPG only (blend weight 1.0). Otherwise Barnes, McBride and Kornet get minutes they no longer play (McBride's o0.5 ast looked like 72%, really ~67%).
- **Role-inflation is a player-level signal (Nov 21):** if any standard line for a player is more than 15% above his season average, skip all of his unders, not just that line. Green u11.5 P+R lost on Nov 20 with his line 19% above average.
- **Fast data route for big boards:** WebSearch "basketball-reference 2026-27 NBA Player Stats Per Game" to surface the league per-game URL, then WebFetch it (and the prior season's) with an explicit player list and the format "Name | Team | G | GS | MP | PTS | TRB | AST", in batches of about 30. The season page (NBA_<year>.html) has the Opponent Per Game table for points and rebounds allowed.

## 1. Inputs

From the screenshots, list every prop: player, team, opponent, game time, stat (pts, reb, ast, PRA, Pts+Rebs, Rebs+Asts, 3PM), line, and its tier:
- **S (standard):** has both Less and More, so both the over and the under are playable.
- **G (goblin, green devil icon, More only):** discounted line set below market, lower payout. Over only.
- **D (demon, red icon):** line set above market, higher payout. Over only.
The user calls both G and D "demon lines" and wants them scored too. Note which players changed teams or roles this season. If a name is hard to read, check the team's box score roster before modeling.

## 2. Data to gather (in this order)

1. **Today's slate and this season's games:** `fetch_sports_data` with `data_type: scores` for the league (gives win probabilities too; flag heavy favorites, ~85%+, for blowout minutes risk), then with `team` filters for each team on the slate to get their game IDs this season.
2. **Box scores:** `fetch_sports_data` with `data_type: game_stats` for each team's games this season (one call covers both teams). Use them for:
   - each player's points, rebounds, assists, FGA and points in the paint per game. Player lists are often partial and give no minutes, so a missing player has not necessarily sat out; confirm with season totals (step 3) before excluding anyone;
   - each opponent's defense by zone: points allowed, opponent points in the paint, opponent at-rim FGM/FGA, opponent mid-range FGM/FGA, opponent 3PM/3PA.
3. **4 seasons of history plus current-season totals:** WebFetch only works on URLs that came from a WebSearch result. Fastest route: search "<player> statscrew basketball stats" and fetch the statscrew.com `p-<bbrefid>` link; its Player Totals table has G, MIN, PTS, REB, AST for every season including the current one (with minutes), so compute per-game and current minutes from it. Fallbacks: basketball-reference.com player pages (search "<player> basketball-reference.com players <bbref id>" or "<player> Stats, Height, Weight, Position, Draft Status basketball-reference"; any of www., aws. or bare domain), landofbasketball.com season pages (one season each; the "2026" page is 2025-26), statmuse "<player> stats" pages (current season only). Statscrew sometimes has data errors (e.g. a season with 1 rebound); sanity-check totals. Current-season rows can lag by a game; trust box scores and RotoWire for the latest games.
4. **Shot profile (shot chart by distance):** from the basketball-reference player page, pull the **Shooting** table. The fetch summarizer often skips it unless asked explicitly; use this prompt: "Locate the table titled 'Shooting' (columns '% of FGA by Distance' 2P, 0-3, 3-10, 10-16, 16-3P, 3P and 'FG% by Distance' same). Copy the regular-season rows <seasons> exactly (season, team, G, then the 6 % of FGA values and the 6 FG% values). If absent, reply 'ABSENT'." Some pages (e.g. Kawhi Leonard) return ABSENT, and some players never surface a basketball-reference URL in search (statscrew has no shooting table); for those, set the zone fit to neutral (1.0) and say so. Never fill a shot profile from memory. On a big slate (30+ players), prioritise shot tables for points/PRA props near the top of the ranking on either side.
5. **Current role, minutes, preseason and injuries:** search "rotowire <player>" and fetch the RotoWire player page (or headline links) for recent stat lines with minutes, starting role and injury notes. Search injury news for any star or regular missing from both box scores (e.g. "<player> injury <month> <year>"). Exclude a player only when he is confirmed out, or has zero games this season and an open injury; say so in the output. Minutes limits and blowout risk matter most for unders.
6. **Defense vs. position (optional, noisy early season):** RotoGrinders NBA Defense vs. Position cheat sheet (rank 1 = allows the most). Treat as a weak signal with small samples.

Opponent zone defense by season (team opponent-shooting pages) was not reachable through search on basketball-reference; build it from this season's box scores (step 2) and shrink it hard toward league average (below). If a search does surface a team opponent shooting-by-distance page, use last season's numbers blended with this season's.

## 3. Model

Work in per-minute rates, then scale to projected minutes. Run it in Python (one data dict per player, loop over every prop), not in your head.

- **Season weights (per-minute PTS/REB/AST):** last season 24, two seasons ago 12, three ago 6, four ago 3, each multiplied by min(G, 60)/60.
- **Role change** (new team, returning from a long injury, or clearly new role such as a new starter or new lead scorer): halve all past-season weights.
- **Current regular-season games:** weight 6 each (these show the current role). When a game's minutes are unknown, use the season-total minutes split evenly, or last season's MPG as a stand-in, and flag it.
- **Preseason games:** weight 1.5 each.
- **No history (rookies, players back from overseas):** use only current and preseason games and mark the projection THIN.
- **Projected minutes:** 0.7 x current-season MPG + 0.3 x last-season MPG (0.85 / 0.15 for role changes). Flag big minute drops (for example a star suddenly at 24-28 minutes).
- **Last-5 role check:** for every player whose pick could make the table, compare his last 5 games (minutes and FGA, from box scores or RotoWire) with his season averages. If last-5 minutes differ by 15% or more, or last-5 FGA by 20% or more, and there's a clear reason (a new starter, a teammate in or out, a returning player, a minutes limit lifted or added), treat it as a role shift: weight each of those 5 games 12 instead of 6, and project minutes as 0.6 x last-5 MPG + 0.4 x the normal blend. With no clear reason, treat it as a hot or cold streak and change nothing; FG% or 3P% swings alone never count. Mention any applied shift in the notes, and track graded results for shifted vs unshifted picks so the rule can be kept or dropped.
- **Head-to-head history:** not used (2–4 games against one team is too small and stale); opponent strength comes from team defense this season plus the individual matchup tweaks.
- **Opponent factor on points:** 1 + 0.5 x (opponent points allowed per game / 113 - 1). Use 0.25 instead of 0.5 for assists, none for rebounds.
- **Zone-fit multiplier on points:**
  - Player profile: weight the Shooting-table seasons 24/12/6/3 x min(G,60)/60 (newest first) for both FGA share s_z and FG% p_z across zones 0-3, 3-10, 10-16, 16-3P, 3P. Blend current-season shares in at 25% (role signal); do not use current-season FG% (too few shots).
  - Opponent ratio r_z = allowed vs. league average by zone: rim (0-3) from opponent at-rim FG% (league ~.66), paint (3-10) from opponent paint points per game (league ~48), mid (10-16, 16-3P) from opponent mid-range FG% (league ~.42), 3P from opponent 3P% (league ~.36). Shrink toward 1: paint/rim weight 0.4 with 2+ games (0.25 with 1 game), 3P weight 0.2 (0.12), mid 0.1. Raise these weights as the sample grows (roughly paint 0.7, 3P 0.5, mid 0.4 after 15+ games).
  - r_avg = league points share by zone (rim .27, paint .27, 10-16 .05, 16-3P .04, 3P .37) dotted with r_z. Dividing by it stops double counting the points-allowed factor, so this term only measures fit.
  - zone_mult = sum(s_z x v_z x p_z x r_z) / (sum(s_z x v_z x p_z) x r_avg), with v = 2,2,2,2,3. Apply it to points (and the points part of combos) only. Typical range 0.97-1.04.
- **Blend with the line (standard lines only):** final mean = 0.4 x model + 0.6 x line for overs and 0.5 x model + 0.5 x line for unders (overs tightened Nov 21 after regular overs went 1-8 over two graded runs).
- **Goblin and demon lines:** do not anchor to the line (it is set off-market). Use the model mean, or anchor to the player's normal market line if the board implies it (e.g. a goblin points line next to a standard PRA line; LaMelo's 11.5 goblin points was anchored to about 17.5). Widen SD by 1.15 instead of 1.1.
- **Spread:** SD = coefficient x mean x 1.1 (1.25 for THIN standard lines, 1.3 for THIN goblins), with coefficients pts 0.30, reb 0.42, ast 0.42, PRA 0.21, Pts+Rebs 0.24, Rebs+Asts 0.30.
- **P(over)** = 1 - NormalCDF((line - mean) / SD); **P(under)** = NormalCDF((line - mean) / SD), standard lines only.
- **Whole-number lines** (e.g. 5, 7, 11): an exact hit is a push, not a win for either side. Compute P(over) at line + 0.5 and P(under) at line - 0.5, and say the remainder is push chance. (Mistake made once: an under on LaMelo 5 reb looked like 69% when computed at 5.5; it was really about 50% to win.)

Reference implementation:

```python
from math import erf, sqrt
def cdf(x, mean, sd): return 0.5 * (1 + erf((x - mean) / (sd * sqrt(2))))
def p_over(mean, sd, line):
    x = line + 0.5 if float(line).is_integer() else line
    return 1 - cdf(x, mean, sd)
def p_under(mean, sd, line):
    x = line - 0.5 if float(line).is_integer() else line
    return cdf(x, mean, sd)

# shot: list of (G, shares[5], fg[5]) oldest -> newest, zones 0-3,3-10,10-16,16-3P,3P
# cur_shares: this season's shares or None; opp_r: shrunk opponent ratios [rim, paint, mid, mid, 3P]
V = [2, 2, 2, 2, 3]; LGW = [.27, .27, .05, .04, .37]
def shrink(raw, w): return 1 + w * (raw - 1)
def zone_mult(shot, cur_shares, opp_r):
    if not shot: return 1.0
    dec = [0.125, 0.25, 0.5, 1.0][-len(shot):]
    s = [0]*5; p = [0]*5; W = 0
    for (g, ss, pp), d in zip(shot, dec):
        w = 24 * d * min(g, 60) / 60; W += w
        for i in range(5): s[i] += w * ss[i]; p[i] += w * pp[i]
    s = [x / W for x in s]; p = [x / W for x in p]
    if cur_shares: s = [0.75*a + 0.25*b for a, b in zip(s, cur_shares)]
    r_avg = sum(a * b for a, b in zip(LGW, opp_r))
    num = sum(s[i] * V[i] * p[i] * opp_r[i] for i in range(5))
    den = sum(s[i] * V[i] * p[i] for i in range(5))
    return num / den / r_avg

def T(g, mins, pts, reb, ast):  # statscrew season totals -> per-game tuple
    return (g, mins / g, pts / g, reb / g, ast / g)

# hist: list of (G, MP, PTS, REB, AST), oldest -> newest (up to 4 seasons; may be empty)
# cur: list of (MIN, PTS, REB, AST) for this season's games; pre: same for preseason
def project(hist, cur, pre, opp_allowed, new_role=False, zone=1.0, lg=113.0):
    decay = [0.125, 0.25, 0.5, 1.0][-len(hist):] if hist else []
    hw = 0.5 if new_role else 1.0
    num = [0, 0, 0]; den = 0
    for (g, mp, pt, rb, a), d in zip(hist, decay):
        w = 24 * d * min(g, 60) / 60 * hw
        for i, v in enumerate((pt, rb, a)): num[i] += w * v / mp
        den += w
    for games, w in ((cur, 6), (pre, 1.5)):
        for mn, pt, rb, a in games:
            for i, v in enumerate((pt, rb, a)): num[i] += w * v / mn
            den += w
    rate = [x / den for x in num]
    cur_mp = sum(c[0] for c in cur) / len(cur) if cur else hist[-1][1]
    last_mp = hist[-1][1] if hist else cur_mp
    k = 0.85 if new_role else 0.7
    mp = k * cur_mp + (1 - k) * last_mp
    f = opp_allowed / lg - 1
    pts = rate[0] * mp * (1 + 0.5 * f) * zone
    reb = rate[1] * mp
    ast = rate[2] * mp * (1 + 0.25 * f)
    return pts, reb, ast, mp
```

## 4. Output

The user wants the three kinds of picks kept separate but side by side in one table, so they don't have to scroll between lists. Format:

0. One line with the previous run's graded record and the fix applied (section 0).
1. One sentence naming the best regular over, the best discounted over and the best under.
2. One combined table, top 10 rows, one line per player per column (that player's highest-probability line in that column):

| # | Regular over | P | 😈 Discounted over | P | Regular under | P |

   Use short pick text like "Ziaire Williams o13.5 PRA" / "Kawhi Leonard u34.5 PRA". Bold probabilities of 70%+ for unders and regular overs, and 80%+ for discounted overs. Stop a column early (leave cells blank) once its picks drop below about 58%, and say so. Include THIN projections (rookies, small samples) in the table when they rank, using their wider SD, and mark them with "(R)"; the user wants rookies kept in. Still skip a rookie's under when the line is role-inflated. Include demon (D) lines in the discounted column only if they rank; otherwise mention a demon only when it looks bad.
3. One line naming players who appear in more than one column (pick only one line each, since props on the same player are correlated).
4. Short notes only where they change a decision: excluded players and why, estimated minutes, whole-number lines with real push chance, neutral zone fits on top picks, blowout risk, lines that moved since an earlier screenshot.
5. If the user asks how picks combine on a slip, multiply the probabilities (assume independence across different players and games) and show the 2- to 6-pick odds.

Close with one line of limits (early-season defensive samples are small so zone effects stay modest; even an 80%+ pick misses about 1 in 5; the game-to-game spread is a standard estimate, not each player's own), then a Sources list with every stats page fetched plus the box score source, as markdown links. Keep the tone neutral about gambling: no urging to bet, and do not repeat legality lectures the user has already heard, but mention once if a line looks like it comes from a sportsbook that is not legal where the user is.