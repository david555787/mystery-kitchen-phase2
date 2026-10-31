# Mystery Kitchen Handoff Report

**1. What did I read first, and did it help?**

I first read `SPEC.md` and `README.md` to understand the game the author wanted to build and how the implemented version worked. The spec helped me understand the main goal: use six random ingredients to prepare a dish within a time limit, present it to three judges with different preferences, and avoid wasting ingredients.

The README helped more directly with the gameplay details. For example, players must select two cooking methods, only plated ingredients count toward judging, and food can be taken off the plate before pressing Finish. The author also explained that the timer starts at a different point than the spec originally required. This helped me distinguish implementation problems from intentional design changes.

**2. Was the spec clear? Were the acceptance criteria easy to check?**

Overall, I could tell that the author wanted a game emphasizing experimentation, the cooking process, and judge feedback. Many acceptance criteria were specific: players must select exactly two cooking methods, each judge gives a score from 1–10, each unused ingredient costs two points, and players can name their dish after pressing Finish. These requirements can be checked by playing the game or inspecting the code.

However, some details required comparing the README with the code. For example, “unused ingredients” actually means ingredients that were not plated, even if they had already been chopped or cooked. The requirement for instructions before starting also leaves some room for interpretation: the original game included complete instructions, but they were collapsed by default, so players could start without reading them. Also, the spec says players must press Finish before serving, while the game automatically opens the naming and serving window when time runs out.

These details did not prevent me from understanding the game, but they made it harder to decide whether certain acceptance criteria were satisfied.

**3. What made the code easy to work with?**

Having the configuration grouped together was helpful. `INGREDIENTS` stores each ingredient’s preparation options, cooking time, and classic cooking methods, while `METHODS` defines the equipment and heating parameters for each method. This gave me a clear starting point when investigating how an ingredient or cooking method behaved.

`quality()` also made the code easier to understand. It provides a shared way to determine whether food is raw, undercooked, perfectly cooked, overcooked, or burnt, and both the interface and scoring use those states. When investigating a judge’s comment, I could first check the food’s quality and then trace how the scoring function used it.

Scoring is also concentrated in `scoreDish()`, which makes it easier to compare the three judges’ criteria and the waste penalty. The project’s 11 original tests provided a useful foundation, covering cooking states, vessel capacity, plating, penalties, and resetting the round.

**4. What made understanding and modifying the code take more time?**

The main difficulty was how densely some of the code was written. The first line of `style.css` contains a large amount of base styling and mobile layout rules. To understand why the judges disappeared on narrow screens, I had to locate `.game-judges{display:none}` within that long line and check whether any later rules overrode it.

Another detail that required tracing several functions was the ingredient’s `form`. Preparing an ingredient changes it to `'prepped'`, while blending changes it to `'puree'`. However, the original knife-work check treated every non-`'whole'` form as prepared. This meant an egg blended directly from whole could receive knife-work credit. Understanding the issue required following the preparation, blending, and scoring code together.

**5. What one thing would I ask the author to do differently next time?**

I would suggest keeping a centralized list of differences between the final implementation and the spec, including which changes were intentional. The author already explained the change to the timer’s starting point in the README, which was helpful. If decisions such as counting unplated ingredients as unused, automatically opening the naming window when time expires, and collapsing the instructions by default were documented together, the next developer could more easily decide what to preserve and what to fix while continuing within the original design.