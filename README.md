# nba-prop-probabilities

A Claude skill that estimates the probability of NBA player prop overs and unders (points, rebounds, assists and combos) from PrizePicks-style line screenshots.

It uses per-minute history, current role and minutes, opponent defense, and the line itself, then ranks regular overs, discounted ("goblin"/"demon") overs and regular unders in one table. Each run starts by grading the previous run's picks.

## Install
Copy this folder to your Claude skills directory (or upload `SKILL.md` as a custom skill), then run `/nba-prop-probabilities` and paste your line screenshots.

## Notes
- Probabilities are model estimates, not guarantees.
- Check that sports betting and DFS are legal where you live.
