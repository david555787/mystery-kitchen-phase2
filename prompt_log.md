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

## Prompt 3: traditionalist judge praised burnt food
- Prompt: in scoreDish(), stop the traditionalist's classicPick from praising an ingredient whose quality is burnt or bad; prefer a good-quality classic ingredient, otherwise give a critical comment in the existing tone; don't touch scoring formulas or the other judges; add one burnt-chicken test (SPEC criterion 7: judge comments should relate to the dish)
- Result: agent changed scoreDish() so the traditionalist prefers classic ingredients rated perfect, then fresh or warm; if every classic ingredient is bad, the comment names the problem (e.g. "The chicken is burnt. A classic preparation deserves better."); added one regression test; 14 tests pass
- My check: read git diff to confirm scoring formulas and the other two judges were not touched, ran the tests myself
- Issue: none

## Prompt 4: blending counted as knife work
- Prompt: in scoreDish(), stop the technique judge from giving knife-work credit / the "correct knife work" comment to ingredients that were only blended (form 'puree') and never chopped; keep blending-as-method credit, scoring weights and the other judges unchanged; add tests for blended-egg vs prepped ingredient (SPEC criterion 7: judge comments should relate to the dish)
- Result: agent made knife-work credit require form==='prepped'; when nothing qualifies the technician says "No knife work on this plate"; added one test (blended egg gets no knife-work praise, prepped carrot still does); 15 tests pass
- My check: read git diff, ran tests myself, played a round with only blending and one with chopping in the browser
- Issue: - Issue: I asked for two separate tests (a blended egg gets no knife-work praise; a prepped ingredient still does). The agent combined both cases into one test. Both checks were included, but separate tests would make a failure easier to identify. What I did: reviewed the test and accepted it because both assertions are present and asked the agent to split it into two tests.

## Prompt 5: judges' preferences hidden on narrow screens
- Prompt: on screens <= 700px, style.css line 1 hides .game-judges so players can't see judge preferences while cooking (SPEC criterion 3). Make them reachable without editing line 1: append a commented override at the end of style.css, no game.js changes unless unavoidable, no logic/scoring/wording changes
- Result: agent appended a commented override at the end of style.css (line 1 untouched, game.js untouched); at <=700px the judges appear below the kitchen controls as a horizontally scrollable strip with portrait, name, preference and description; no CSS test added because the fake DOM can't check layout; 15 tests pass
- My check: read git diff, ran tests, resized the browser to phone width and checked the strip, the controls and the wide layout
- Issue: The agent only ran the Node tests, which use a fake DOM and cannot check CSS, so passing them did not confirm the mobile layout. The agent said this itself and said browser verification was still needed. What I did: I resized the browser to phone width and checked the judges strip, the kitchen controls and the wide-screen layout myself.