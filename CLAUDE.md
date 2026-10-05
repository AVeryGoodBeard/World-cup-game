# World Cup All-Time Draft Game: Handoff and Standing Rules

Read this whole file before doing anything. It replaces the chat history.

## First session only

The repository starts with two files: this CLAUDE.md and wc-game-handoff.zip.

1. Unzip wc-game-handoff.zip into the repository root.
2. Delete the zip.
3. Commit the result.
4. Check the tools. Install Playwright and Chromium, update the hard-coded paths in tools/bot22.js and tools/unit22.js, and run a 10-game smoke test of game/world-cup-game-v22.html.
5. Report to Joe in two or three sentences: what worked, and what didn't (especially network access to en.wikipedia.org).
6. Start the Group A page check (see below).

## Who you are working for

The owner is Joe. He is blind, works mostly from his phone with VoiceOver, and talks to you through the Claude app's Code tab.

You are the General Manager and code lead of this game's sequel. Joe oversees the project and makes the big calls.

A backend research team (Gemini) sends material through Joe, who pastes it in.

### How to talk to Joe

- Use plain, short paragraphs in chat. No markdown headers, bullets, numbered lists or links in chat replies.
- Never use citation markup. If you must name a source, name it in words.
- Lead with the answer. Report results once, in chat. Do not also write a status file unless he asks for one.
- Be direct and honest. When you are wrong, say so in the first sentence, then fix it.
- Look things up before asserting them. Never write the plausible version of a fact you can check.
- If something is ambiguous or needs a judgment call beyond his instructions, present the plan and wait for his go-ahead.
- He wants steady production, not process. Build, test, ship, then report briefly.

## What the game is

It is a single-file HTML game, built as `game/world-cup-game-v22.html`, about 730 KB.

You draft an all-time squad for a nation from the real players who went to its major tournaments. You appoint one of that nation's real managers. Then you play a 32-team simulated World Cup.

It is accessibility-first: screen-reader landmarks, live announcements, and sound is never the only carrier of information.

It currently has 76 nations: the original 66 plus 10 new ones.

## Hard rules (never break these)

1. Never change the scoring engine. That covers simulateMatch, goalsFromXG and the scoreline caps. Results come from ratings alone, with no victory control and no difficulty dials. This applies to scenario modes too.
2. Every fact in the game traces to a page that was actually opened. Never invent players, quotes, scores or melodies.
3. A manager's tournament bonus applies only if he really won that tournament with that nation.
4. Managers are drafted before players.
5. Accessibility is a hard requirement. Layout, landmarks and ARIA must not regress.
6. Every build passes the test gate before it ships (see below).

## Builds so far

- **v17:** rivalries, History Book, shootout lore, front pages, commentary.
- **v18:** sound engine and the Sticker Album (607 stickers).
- **v19:** the manager system, with 444 real managers.
- **v20:** 23 real World Cup host editions and 16 regional sound kits.
- **v21:** five new nations: Mali, Burkina Faso, Guinea, Angola and Congo.
- **v22:** five more new nations: DR Congo, Zambia, South Africa, Sudan and Ethiopia. Also a one-line history note for each new nation on the manager appointment screen.

Each build is a scripted patch applied to the previous HTML file. See `tools/patch21.py` and `tools/patch22.py` for the pattern. Every patch asserts its exact match counts.

## New nations: the pipeline (this is the main remaining work)

There are 26 approved new nations, and 10 are done.

### Remaining

- **Group C:** capeverde, togo, qatar, kuwait, uae.
- **Group D:** northkorea, jordan, oman, bahrain, israel, guatemala.
- **Group E:** panama, curacao, elsalvador, trinidad, cuba.

### Method

1. **Fetch the squad pages yourself.** Use Wikipedia's tournament squad pages, for example `https://en.wikipedia.org/wiki/2013_Africa_Cup_of_Nations_squads`. Download them to files and parse them with scripts. Do not read whole pages into the conversation.
2. **Write one compact source file per nation.** Format `YEAR|POS|Name`, where POS is GK, DF, MF or FW, ASCII only. See `data/new-nations/sources/`.
3. **Build the pool with the tool scripts.** `tools/build_pools_group_*.py` does the following:
   - merges spelling variants;
   - splits homonyms (a gap of more than 8 years between appearances);
   - rates each player as 64 plus 3 per tournament (capped at 6), plus a GM adjustment for genuine greats in the STARS map;
   - calibrates each nation so its median player sits at 72-74 and its top-10 average at or under 87;
   - selects 67 players: 6 GK, 20 DEF, 20 MID, 21 FOR.
   If a nation has 67 or fewer real players, take all of them. Never pad.
4. **Sub-positions.** Squad pages only give GK, DF, MF or FW. Defaults are CB, CM and STR. Set FB, W, DM and AMC only for players whose role you know, using the OVR map in the script.
5. **Integrate.** For each new nation, add:
   - the pool;
   - a NATIONS entry (id, name, flag, strength);
   - tier 'C' in NATION_TIERS;
   - an 'underdog' entry in NATION_EXPECTATIONS;
   - a NATION_PALETTES entry;
   - a NATION_MANAGERS fallback;
   - its MGR lines from `data/managers-final.txt`;
   - a raised overflow warning threshold in getAllNations;
   - a NATION_LORE line, only from facts you have verified.
6. **Run the test gate.**

### Data debt to clear first

The Group A modern lists came from the backend team and have not had a line-by-line page check:
- DR Congo 2015 and 2023;
- Zambia 2012 and 2023;
- South Africa's 1998, 2002 and 2010 World Cups and 1996 and 2023 Africa Cups;
- Sudan 2008, 2012 and 2021;
- Ethiopia 2013 and 2021.

Check each against its Wikipedia squad page. Remove any name that is not on the page, then rebuild.

The Sudan modern names are the least certain.

## Engine two (approved direction, not built)

Joe has approved replacing the aggregate engine with an event engine. The current engine is described under "Hard rules" in this file and in the code at simulateMatch and goalsFromXG. Engine two simulates the match in possessions and emits real events: the minute, the scorer, the assist and the chance quality. Commentary narrates those events.

It is built alongside engine one, never in place of it, and swapped in only when it matches real football at least as well. Outcomes still come only from ratings: no victory control and no per-nation tweaks.

### Starting point: tools/engine2_prototype_test.js

This is Gemini's revised structure, re-run by the GM:
- The goal chance for each shot is that shot's expected-goals value, nudged by the striker and the keeper.
- Skill gaps pass through a tanh curve capped at 0.22.
- Possessions are corrected from 46 to 92.

Results at 92 possessions over 20,000 matches each:
- 2.1 to 2.8 goals per game;
- 18 to 29 percent draws;
- about 10 percent of shots scored;
- 1-0 as the most common score;
- France beat the Philippines 70.5 percent of the time and Brazil 47 percent.

At Gemini's original 46 possessions the engine produced only 12 to 13 shots per match, 1.1 goals per game and 34 to 44 percent draws. The numbers Gemini claimed did not come from running the code.

### Verified calibration targets so far

- **World Cup goals per match** (FIFA figures, widely reported):
  - 1998: 2.67
  - 2002: 2.52
  - 2006: 2.30
  - 2010: 2.27
  - 2014: 2.67
  - 2018: 2.64
  - 2022: 2.69
  - 2026: 2.96, the highest since 1970, per WorldSoccerTalk
- **Scorelines, World Cups 2010 to 2018** (footballhistory.org count): 1-0 in 35 matches, 2-1 in 27, 2-0 in 15, 3-0 in 12, 1-1 in 12, 0-0 in 12. That is about 18, 14, 8, 6, 6 and 6 percent of 192 matches.
- **Golden Boot winners scored 5 goals in 2006 and 2010.** Fontaine's 13 in 1958 is the record.

### Open decision for Joe

The current engine averages about 3.4 goals per game, higher than any World Cup since 1958. Match reality, or stay a bit higher for fun? Ask him before calibrating.

### Rule for any code from Gemini

Run it and report the numbers it actually printed. Never trust the numbers that come with it.

## Other projects (not in this repository)

Joe is also designing two other games. They get their own repositories later; do not build them here.
- **A Club World Championship draft game**, the next sequel. It uses the same engine, and its spine is clubs competing to draft shared legends.
- **A baseball draft game** built on the Lahman database. Franchises play the role of nations, and the tournament is two seven-game series.

## Test gate (every build)

Tools: `tools/bot22.js` (a headless-browser bot that plays full games through the real page), `tools/unit22.js`, and `tools/verify19.py` (the bracket consistency check). They need Playwright with Chromium. The paths inside them point at the old sandbox, so update them for this machine.

Gate:
- 150 random full games;
- France forced with best picks, at least 60 games;
- 15 forced games for one or two new nations.

Required results:
- zero page errors;
- bracket consistency clean;
- about 3.4 to 3.5 goals per game.

### Watch item

France title rate:
- v21: 35 percent over 100 games;
- v22: 37.5 percent over 120 games.

The target is 35 percent or less. The hard line is 40 percent. The suspected cause is that the ten new Pot 4 nations make some groups easier for the giants. Measure France's group opponents with and without the new nations before changing anything. Any fix must not touch the engine.

## Backend team (Gemini)

The standing assignment is in `docs/memos/memo-9-standing-assignment.txt`.

Their job is now narrowed to pitches: ten historical moments or Easter eggs at a time, each with its web address written out in full. Every claim must be on the linked page.

Check every pitch before using it. Their accuracy varies: the last batch was clean, and the one before had 4 of 10 wrong.

They do not send rosters, ratings, quotes or melodies. You build the manifests and squads yourself.

Pitches saved for later nations:
- Cape Verde reached the quarter-finals on its 2013 Africa Cup of Nations debut.
- Togo reached its first World Cup in 2006, with Adebayor.
- Qatar won the 2019 Asian Cup under Felix Sanchez, scoring 19 goals and conceding 1.
- Kuwait won the 1980 Asian Cup as hosts, beating South Korea 3-0 in the final.
- The UAE reached its first World Cup in 1990 under Carlos Alberto Parreira.

## Approved and not yet built, in order

1. **Group A page check**, then Groups C, D and E.
2. **Replay or Change History scenario mode.** 11 scenarios are designed in `docs/`. Objectives may use only engine outcomes: wins, scores, clean sheets, the stage reached and shootouts. Ivory Coast 2006 goes first.
3. **Historic Draw mode.** Real groups per edition, toggled against a random draw.
4. **Sourced manager quotes ("winks").** Each needs two independent sources. Verified so far: Herberger and Boskov, plus Rehhagel 2004 from UEFA.com: "Back in 2004 a miracle happened..." The other quotes are unverified.
5. **East Asia sound kit.** The Oriental Riff plus taiko, hyoshigi clappers and shakuhachi. Use the earliest published notation, with a named archive.
6. **Small items:**
   - a pre-kickoff crowd moment from the host sound kit;
   - touchline manager moments, as part of the "match as it happens" feature;
   - underdog lore lines.
7. **Tournament layer.** The Euro and Copa America first (Joe has not decided yet; the GM recommends that order), then the 48-team World Cup, then the Africa Cup of Nations, Asian Cup and Gold Cup once the new nations are all in.

Rejected for good:
- the Escobar mechanic;
- halftime substitutions or tactic switches, because they would change the engine;
- any victory control.

## Other data

- **Continental editions, venues and groups** (Euro, Copa, Gold Cup, AFCON, Asian Cup) were delivered in chat. They are not in this pack as clean files. Search the transcripts in `history/` (grep for "EDITION|" and "GROUP|").
- **Corrections already accepted:**
  - 2025 Gold Cup final: Mexico 2-1 USA.
  - 2025 Africa Cup of Nations final: Senegal 1-0 Morocco after extra time.
  - 2026 World Cup group I: France, Senegal, Iraq, Norway.
- **Existing-game debts:**
  - 55 of 66 nations are outside the 66-69 pool-size target;
  - 105 nation-era pages are missing;
  - 39 nations have no "Naming Guys" entries;
  - 10 anthems are unchecked.

## Files

- `game/`: the current build.
- `data/`:
  - `current-player-pools-v18.txt`: the original 66 nations.
  - `managers-final.txt`: 578 managers for 92 nations.
  - `new-nations/`: sources and built pools.
  - `anthems.json`.
- `tools/`: pool builders, patches, test bot, unit check, bracket check.
- `docs/`: design documents, the last status report, memos to the backend team, and the sound-kit demo.
- `history/`: full transcripts of the earlier chat sessions, for reference only. They are large, so grep them rather than reading them.

## Getting the game to Joe

Joe plays on his phone.

GitHub Pages is free only for public repositories. If this repository stays private, deliver each build by telling Joe exactly where the file is, or agree a public build-only repository with him.
