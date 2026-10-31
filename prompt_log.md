## Prompt 1: shuffle fix
- Prompt: TASK: In game.js, shuffle() (around line 65) uses
.sort(() => Math.random() - .5), which does not produce uniformly random
orderings. Replace only its implementation with a Fisher-Yates shuffle.
It must keep the same function name, signature and return type
(a new shuffled array; do not mutate the input if the original didn't).
Add one test in tests/game.test.cjs checking that shuffle returns a
permutation (same elements, same length) of its input.
- Result: agent changed only game.js and tests/game.test.cjs, 12 tests pass
- My check: read full git diff, ran tests myself
- Issue: none

## Prompt 2: instructions visible by default
- Prompt: TASK: SPEC acceptance criterion: "An instructions screen at the
beginning, before the player presses start, explains ingredient
selection, cooking methods, judge preferences, and the waste penalty."
Currently renderIntro() (game.js ~196) builds the instructions in a
drawer that is hidden by default and only opens via "How to Play".
Change only this: make the instructions visible by default on the intro
screen, keeping the existing instruction content and the existing Start
button behavior. Keep the "How to Play" toggle working (it may collapse
the section). Do not change the instruction wording. Add or adjust one
test that confirms the intro markup contains the instruction content
without needing a click.
- Result: agent removed `hidden` and set the toggle's aria-expanded to true in game.js, added one test, 13 tests pass
- My check: read git diff, ran tests, checked the intro screen in the browser
- Issue: None