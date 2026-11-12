# Testing notes

All tests ran on 11–12 November 2026 against this `index.html`, using local MQTT relays (aedes) that delay every message by a random 0–300 ms so the relays deliver events in different orders. Phones are real Chromium pages running the game; extra players are protocol bots that behave like phones. Only Chromium was available, so Safari (iOS) and the WeChat in-app browser still need a real-device check.

## Concurrency (50 players: 20 browser phones + 30 bots)

| Scenario | Runs | Result |
| --- | --- | --- |
| Everyone joins within the same second | 5 games | Pass: 50 players, correct team sizes, exactly one agreed captain per team. For up to ~1.6 s after joining, several phones in a team may briefly show the captain star before it settles. |
| Captain race: 12–13 players per team claim at the same moment | 15 rounds | Pass: host, every phone and every bot agree on one captain |
| Last-second votes (50 votes in 300 ms, half change twice) | 3 × 5 reveal timings | Every phone always shows the same result as the big screen. Votes still in flight at the moment of reveal are not counted, so wait until every team shows "Captain locked". |
| Captain re-locks B→C→D in 120 ms | 3 | Pass: the last lock counts |
| Host resets a captain while the captain locks | 2 | Pass: the old lock is dropped and the new captain's lock counts |
| All relays drop for 5 s / each relay drops in turn | 1 / 1 (and 2 / 2 on the previous version) | Pass: no duplicate players, no lost votes, captains unchanged; all phones live again after 11.5 s / 3.6 s (8.5–17.4 s / 1–3.9 s on the previous version) |
| Same phone in two tabs, page reload, rejoin | 1 + 3 | Pass: counted once; captain kept after reload and re-entering the PIN |
| Someone opens "Host a game" from the link mid-game | 1 | Pass: they get a different game code, so the running game is untouched |

## Template features

| Check | Result |
| --- | --- |
| Editor: change title, background, pattern, 5 teams, option text, add a round, save and host | Host and all 5 phones show the custom game; 6 rounds and the final ranking work |
| Duplicate team PIN | Blocked with a message; nothing saved |
| Export to text and load it back | Same settings |
| Poisoned settings published into a running game (HTML in titles, invalid colours) | Ignored by phones; the host restores its own settings; no script runs |
| Two games at the same time | No cross-talk |
| Wrong game code / link without a code | Clear error / code field shown |
| QR codes with 4, 5 or 6 teams at 1024–1920 px wide | No overflow or overlap |
| 17 phone and tablet sizes × 6 game stages | No sideways scrolling, nothing cut off, smallest text 16 px |

## Capacity (earlier version with the same networking code)

- Public relays, one real phone + 99 simulated players: every vote arrived and every reveal was correct over 5 rounds.
- Local relays: 200 players at full speed; 400 correct but slower; 800 failed on the 2-core test machine.
- One network address held 100 connections per public relay; 200 simultaneous attempts from one browser tab partly timed out.
