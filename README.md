# Mystery Kitchen

**Original game design and foundation by [irishaoru](https://github.com/irishaoru).** They designed the game, wrote the spec, and built the code this repo starts from. Phase 2 work (finishing and fixing the game against their spec) by Jessie Lu ([@david555787](https://github.com/david555787)).

Original repo: https://github.com/irishaoru/masterchef-game
This repo's first commit is their code exactly as I received it, so `git diff 1dbeac2 HEAD` shows everything I changed.

The game is a cooking competition where players make a dish from 6 random ingredients within a 5 minute time limit and present it to a panel of 3 judges. The objective is to get the highest score out of 30.

## Run it

```
cd <this repo>
python3 -m http.server 8000
```

Then open http://localhost:8000/. Any static file server works. There is no build step, no `npm install`, no backend and no API key.

Run the tests (Node 18+; they load `game.js` into a VM with a fake DOM):

```
node --test tests/game.test.cjs
```

Assumptions: a modern browser (the CSS uses `:has()`, `svh` and `aspect-ratio`), and refreshing the page restarts the game (nothing is saved).

## Code map

Line numbers are approximate and refer to the code as I received it.

| File | Role |
|---|---|
| `index.html` | Page shell: `#app` render target and `#announcer` accessibility region |
| `style.css` | All layout, CSS illustrations, animations, responsive rules. Note: almost the whole base stylesheet is on line 1 |
| `game.js` | The entire game, no classes or modules (details below) |
| `tests/game.test.cjs` | Node tests with a fake DOM; they cover logic, not real clicks or layout |
| `SPEC.md` | The author's spec and acceptance criteria, kept exactly as received |

Inside `game.js`:

- **Data (top of file):** `CATEGORIES`, `INGREDIENTS` (what can be drawn), `METHODS`, `VESSELS` (cooking methods and appliances), `JUDGES`, `STATIONS`, `SEASONINGS`.
- **State:** one `state` object (~line 79) holding the current screen, basket, chosen methods, chef position, timer, appliance heat, seasonings, dish name. `reset()` and `newItem()` create a round and an ingredient record.
- **Screens:** `render()` dispatches to `renderIntro`, `renderSelection`, `renderMethods`, `renderKitchen`, `renderResults`.
- **Cooking simulation:** `simulate()`, `quality()`, `itemStatus()`, driven by `frame()` (animation) and `tick()` (countdown).
- **Actions:** `grab`, `startPrep`, `addTo`, `plateItem`, `togglePower`, `moveChef`, with click/drag/keyboard handlers at the bottom of the file.
- **Scoring:** `scoreDish()` produces each judge's score and comment and the waste penalty; `renderResults()` displays them.

Phase flow: intro (How to Play) -> pick 6 ingredient cards -> choose exactly 2 cooking methods -> cooking (timer runs) -> Finish or timer expiry -> name the dish and Serve -> judges and results on one screen -> Play Again.

## Acceptance criteria report

Status is against `SPEC.md`'s acceptance criteria, one by one.

| # | Criterion | Status | Notes |
|---|---|---|---|
| 1 | Arrow keys to move; touchpad to click everything | Met | Keyboard movement, click and drag all work |
| 2 | Select exactly two cooking methods | Met | Start stays disabled until exactly two are selected |
| 3 | All three judges' preferences visible before serving | Met | Shown on the method screen and in the kitchen. On screens <= 700px wide the original code hid the kitchen's judges row; I appended a CSS override so the judges appear as a horizontally scrollable strip below the kitchen controls |
| 4 | Two-point waste penalty per unused basket ingredient, shown in final score | Met | "Unused" means not plated. The code assumes a basket of exactly 6 |
| 5 | Serve only works after Finish | Partially met | Serve only exists in the modal opened by Finish. But when the timer runs out the game calls the same function automatically, so Serve appears without a Finish click. I think that is sensible behavior, so I left it |
| 6 | Each judge awards 1-10 | Met | Scores are clamped to 1-10 |
| 7 | Each judge's comment relates to the dish and their preferences | Met after fixes | Fixed two contradictions: the traditionalist could praise burnt food, and the technique judge called blended food "correct knife work" |
| 8 | Results show critiques, then scores, then total with waste subtracted | Partially met | All the numbers and the arithmetic are shown, but everything appears on one screen at once. There is no step-by-step reveal and the total starts counting before the last judge's score finishes |
| 9 | Play Again starts a fresh round | Met | `reset()` replaces the round state |
| 10 | Instructions before the player presses Start | Met after fix | Originally the instructions were in a drawer hidden until "How to Play" was clicked. They are now open by default |
| 11 | Dish name before judging | Met | Finish opens the naming modal |

Other items from the spec's priorities:

- **Timer starts when ingredient selection finishes** - Partially met, and the author's README says this was a deliberate change: the timer starts after the two cooking methods are chosen. I did not change it back.
- **Pantry for other miscellaneous ingredients** (nice to have) - Partially met. The pantry has seasonings only. Not extended.

## What I changed

Each change is its own commit.

1. `shuffle()` now uses a Fisher-Yates shuffle. The old `sort(() => Math.random() - .5)` does not give uniformly random card positions. Same function name and signature; one test added.
2. The How to Play instructions are open by default on the intro screen (criterion 10). The text and the Start button are unchanged; one test added.
3. The traditionalist judge no longer praises burnt food in `scoreDish()` (criterion 7). It prefers a well-cooked classic ingredient and otherwise names the actual problem. Scoring formulas untouched; one test added.
4. Knife-work credit now requires an ingredient to actually be prepped, so blending alone is no longer "correct knife work" (criterion 7). Blending still counts as a cooking method; one test added.
5. Narrow screens (<= 700px) show the judges' preferences below the kitchen controls (criterion 3). This is a commented CSS override appended to the end of `style.css`; line 1 was not touched.

**Restructuring:** none. No files were split, renamed or reorganized, and the original design (single `game.js`, one `state` object, existing function names) is kept.

Tests went from 11 to 15.

## Known issues and risks

- The countdown uses the wall clock (`Date.now()`) but the cooking simulation caps each frame at 0.1 s. If the tab is in the background, time can run out while food barely cooks.
- Waste scoring hard-codes a basket of 6 (in `scoreDish()` and in the kitchen feedback).
- The judging round is static (see criterion 8), and the kitchen is cramped on small screens even after the judges fix.
- The tests use a fake DOM, so they do not exercise real button gating, the Serve form, layout, or animation order. Always play a round in a browser after changing anything.
- `style.css` has the base stylesheet on one very long line, and some HTML template lines in `game.js` are very long. Both make precise edits hard.
- `state.basket[item.id]` relies on item ids equal to basket array indices. That is not documented in the code.

## Ideas for whoever continues

Sequential judge reveal, more ingredients in the pantry, sound effects, difficulty levels or different time limits, more browser/device testing, and real UI tests.

## AI use

I used an AI coding tool for the code changes, giving it one small scoped task at a time, reviewing every diff, and committing after each task. My prompts and what happened with them are in `prompt_log.md`. `HANDOFF_REPORT.md` is my own writing.

---

# Original author's notes (kept as received)

## How to Play

1. Read the opening instructions to clarify gameplay.
2. Then, press "start cooking" to pick your six random ingredients. Each ingredient has 3 options and you choose one of three cards. Each card flips over to show what you got.
3. Choose exactly two methods of cooking from these 4 options: sauté, bake, boil, or blend. Start cooking begins the five-minute timer.
4. Use arrow keys (or the on-screen arrows) to walk around the kitchen. Click food on the center table to pick it up, then click a nearby station to use it (a prompt above the station shows what will happen). You can also drag food from the table onto a nearby station, or use the controls below the kitchen.
5. Cook in any order you like. Prepping food is optional but prepped food cooks about a third faster. The pan and pot (on the stovetop), the oven tray, and the mixer each hold up to three ingredients, and ingredients cooked together are noted by the judges. Food can come out, go back on the heat, or move to another appliance.
6. Appliances heat up realistically. Pans take a moment to get hot, water has to come to a boil, and the oven preheats (you can turn them on ahead of time). Food browns as it cooks, moving from raw through perfect to overcooked, and dry heat eventually burns it. Boiling only makes food overcooked. The mixer turns food into a purée, which is plated as a sauce.
7. Season at the pantry and plate at the counter. Plated food can be taken back until you press Finish. Then name your dish and Serve. At five minutes, cooking ends automatically.
8. Three judges score from 1-10 each. Their total loses two points for each unplated basket ingredient, with a minimum final score of zero.
9. Play Again clears the entire round.

## Features built based on spec

**Main game:**

- ingredient selection (players are able to pick one of three hidden cards for each of the 6 ingredients without knowing what's behind them)
- Arrow-key movements
- Aerial view of the kitchen and all the different stations as well as the player itself
- Food can be cooked and burned on the stovetop and in the oven, and it changes colour as it browns
- Chopping, sautéing, boiling, baking and blending (mixer)
- Pantry with seasonings (salt, pepper, olive oil, garlic, chili, basil) that can be added to the dish
- The player names their dish before submitting it
- When the timer runs out, cooking stops and the player goes to naming and then judging
- 3 judges with different criteria
- waste penalty for unused ingredients

**Judging and results:**

- Each judge gives a score from 1-10 based on their preferences
- Each judge gives a short comment about the actual dish
- The results screen shows each judge's critique and score, the combined total, the waste penalty subtracted, and the final score out of 30
- A scoring explanation and a breakdown of how each plated ingredient was prepared help the player improve next time
- Play Again starts a new round and clears the previous results

**Gameplay:**

- The player moves around the kitchen with the arrow keys
- When the player gets close to a station, it lights up and a prompt appears showing what clicking it will do
- The touchpad/mouse is used to click buttons, pick up ingredients and move them into pots, pans, the oven, the mixer and onto the plate (ingredients can also be dragged from the table to a nearby station)
- The player must select exactly two cooking methods
- The judges' preferences are visible before serving (on the instructions, method selection and kitchen screens)
- The Serve button only appears after the player presses Finish
- A "How to Play" section explains ingredient selection, cooking methods, judge preferences and the waste penalty

**Partially built / changed from the spec:**

- The pantry has seasonings only. It doesn't offer extra ingredients like other condiments
- The kitchen is styled like a cooking-show studio but isn't an exact copy of the MasterChef layout. It's also rather squished on the screen

## What still needs to be done / idea suggestions (original author's list)

- Test on more browsers and devices (so far tested mainly in Chrome)
- On phones, the judges' row is hidden in the kitchen (addressed in Phase 2, see above)
- Judging round could be a lot more interactive because right now it feels very static
- More random ingredients (they're pretty limited right now)
- perhaps adding some sound effects (sizzling, boiling, timer countdown, "hands off!")
- difficulty levels or different time limits

## Original author's AI use

Used Codex originally to build out the first version. The UI and gameplay proved to be very challenging for it to fix so I switched to Claude Code. Claude rewrote the cooking system, the animations, replaced the food emojis with actually drawn illustrations, and all of this was based on the features in SPEC.md as well as some modifications given in the prompt, such as having only two cooking methods (which means players can't use all the stations and the timer doesn't start until after they've chosen their methods, which differs from what the spec says).
