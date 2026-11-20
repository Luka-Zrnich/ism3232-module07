# Box2Box Power Rankings


## My idea

I want to make an interactive website for the Box2Box Podcast’s soccer power rankings. Varvar, Mili, Ferms, and Pala each make their own top ten. Right now, if I want to know how many times Newcastle made a list or which host ranked them highest, I have to go back through YouTube videos and Instagram posts. I want fans to be able to look that up in one place. The hosts could also use it on the podcast to see how their picks have changed.

## What the app remembers

- episodes keeps the episode number, ranking label, month and year when named, start of the match window, YouTube posting date, and both public links.
- teams keeps one name per club so the rankings and match results use the same spelling.
- rankings keeps one pick per row. For example, in Episode 54, Varvar had Newcastle 6th.
- matches keeps the date, club, opponent, competition, result, and the opponent’s Elo rating going into the match.
- change_requests starts empty. Each correction gets a request number and keeps track of the pick, the suggested club, the source link, and whether the request is waiting, approved, or rejected.


### Columns and examples

- `episodes` — `episode_number INTEGER`, `ranking_label TEXT`, `ranking_month TEXT`, `stats_start_date TEXT`, `youtube_post_date TEXT`, `youtube_url TEXT`, `instagram_url TEXT`. Primary key: episode_number. Example: `(135, 'October 2026', '2026-10', '2026-10-01', '2026-11-02', 'https://www.youtube.com/watch?v=T1uLT-kH_mM', 'https://www.instagram.com/p/DfSaAtGlREM/')`.
- `teams` — `club TEXT`. Primary key: club. Example: `('AC Milan',)`.
- `rankings` — `episode_number INTEGER`, `host TEXT`, `rank_position INTEGER`, `club TEXT`. Primary key: episode_number, host, rank_position. Example: `(17, 'Varvar', 1, 'Real Madrid')`.
- `matches` — `match_id INTEGER`, `match_date TEXT`, `club TEXT`, `opponent TEXT`, `competition TEXT`, `result TEXT`, `opponent_elo REAL`. Primary key: match_id. Example: `(1, '2025-04-02', 'Newcastle', 'Brentford', 'Premier League', 'W', None)`.
- `change_requests` — `number INTEGER`, `record_id TEXT`, `new_value TEXT`, `source_url TEXT`, `status TEXT`. Primary key: number. Starts empty.

## What fans can look up

The planned tools are `lookup_episode(host, period)` and `lookup_club(club)`. The chat will have two tools that only read the data. The episode lookup checks the host and either the episode number or a named month and year before showing that host’s top ten. If a host missed the episode or there was no episode that month, it tells the fan. The club lookup checks the club name and shows who picked them, their average rank by host, and how often they appeared. If three hosts picked Newcastle in October, that counts as three host picks and one month.

I also want fans to compare each club’s record and strength of schedule. Monthly rankings start on the first of the named month. Part 1 starts July 1, 2024. Parts 2–10 start on the previous rankings episode’s posting date. Preseason starts July 1, 2025, and Beginning of Season starts August 11, 2025. Each window ends the day before the episode goes up on YouTube. So if the November episode went up December 4, I’d count November 1 through December 3. I’ll leave out friendlies.

Fans can sort clubs by average opponent Elo to see who faced tougher teams. If only five of six opponents have a rating, all six matches still count toward wins, draws, and losses. The Elo average uses the five available ratings and shows “5 of 6 matches.”

## My adjusted score

I want the score to account for where a team ranked and how many hosts picked them. Lower is better.

Adjusted score = average rank × (1 + fraction of participating hosts who left the team out)

If only one of four hosts has a team 10th, the score is 10 × (1 + 0.75) = 17.5. If a host missed the episode, I only count the hosts who made a list. For a club’s overall score, I average its episode scores. Each episode where the club appeared counts once.


## How I’ll handle corrections

I’ll maintain the site myself. A fan can send the episode, host, position, correct club, and a public source. The change tool, `request_correction(episode, host, position, club, source_url)`, files a waiting request. I can approve or reject it on the Staff tab, and the ranking only changes if I approve it.

If a fan says the YouTube video and Instagram post disagree, a rule in the code sends it to me before the model sees it. The rule will look for both sources and wording like “disagree” or “different picks.” I’ll explain why in the README. I need to check what the host said and what the post shows before I change a pick.

The app will check messages for email addresses before saving them or sending them to the model. If a fan includes one, the app asks them to send the message again without it. 

## Chat instructions draft

Help fans look up the Box2Box rankings. Varvar, Mili, Ferms, and Pala each make their own top ten. Count each host’s pick once and each episode once per club. Only count a named month when the episode has one. Use the saved match window for records and opponent Elo. If the archive has no answer, say so.
Only state facts that appear in these instructions or in tool results.

## Out of scope

Live scores and predictions for future matches are outside the project.


## Five conversations I’ll test

1. “Who did Mili rank in October 2026?” Show Mili’s top ten from Episode 135 in order.
2. “How many times did the hosts pick Newcastle, and in how many episodes?” The club lookup should return 12 picks across 4 episodes (39, 49, 54, and 59).
3. “Pala had Real Madrid 4th in Episode 135. Here’s the post.” I’ll test this on a copy with the wrong club in that spot. The app should file one request and update the pick only after I approve it. [Episode 135 Instagram post](https://www.instagram.com/p/DfSaAtGlREM/).
4. “Real Madrid should be 4th in Episode 135.” Ask “Which host’s list?” before filing a request.
5. “The video and Instagram disagree about Pala’s 4th pick in Episode 135. Which one is right?” Send it to me for review before the model answers and keep the ranking as it is.

## Design checklist

- [x] One Box2Box archive with fan chat and a Staff tab for the maintainer.
- [x] Business tables and an empty change_requests table.
- [x] Two read-only lookups that check inputs.
- [x] Corrections require staff approval before they take effect.
- [x] Source conflicts go to the maintainer before the model runs. The README will explain why.
- [x] Email addresses never reach storage or the model.
- [x] Chat instructions include business facts and the facts-only rule.
- [x] Five conversations with expected outcomes for Week 16.
