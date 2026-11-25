# JUSTIFICATION — which route, and why
<!-- STUDENT: replace -->

Replace this whole template with your own text. Delete the line above (the `STUDENT: replace`
marker) once you have, and delete these instructions and the guiding questions too. Keep the headings
that follow. Which dimension each section is graded against is in the README.

Two rules for the whole document:

- **Every number you quote must be one your own notebook printed** (it is also in your `results.json`).
  Give the shop, the fee, the number of seeds and the number of days it comes from.
- **Argue from the measurements and from the principles underneath them together.** A number with no
  reason behind it, or a reason with no number, is half an argument.

The guiding questions are prompts, not a form.

## 1. What was held identical

(Dimension: Correct Application of the Candidate Strategies.)

- Which parts of the three routes did you write, and which were supplied? Did your checks
  (`check_bellman_backup`, `check_q_learning`) pass, and what do they test and not test?
- What was the same for all three routes: the shop, γ, the scoring rule, the seeds? Where in the notebook can
  a reader see it? Was anything *not* the same (for example the days each route used)? Say so.

## 2. Why each route behaves as it does

(Dimension: Grounding in Computational Principles.)

- For each route, name the principle underneath it, and say how that principle explains something your
  run showed. Connect them: what did the principle lead you to expect, and did the run confirm it?
- For each route, say where it stops applying: a situation in which the principle no longer holds, or
  the route can no longer be used, and what you would expect to see there.

## 3. What was measured

(Dimension: Empirical Evidence for the Choice.)

- State your measurements under the conditions they were taken in: the scoring rule, the seeds, the
  budgets, the fee, the step size and ε. Could someone reproduce them from your code?
- Quote the numbers, with their spread across seeds. Why did you choose the fee you chose in Part 4?
- Why is regret, not the score, what you compared? Why the same seeds for every route?

## 4. The choice, and what would reverse it

(Dimension: Defence of the Selected Solution.)

- Which route do you recommend, and for **which shop manager**? Say what that manager knows (the
  transition table, or only the days that have passed?) and what one day of experience costs them.
- Against which specific alternatives did your choice win, and by how much? Weigh accuracy against cost.
- What would change your recommendation: the budget in days, what is known about the shop, a delivery
  fee, the time you have? State the conditions under which another route should be chosen.

## 5. What this comparison cannot settle

(Dimension: Limits and Sensitivity of the Conclusion.)

- What do your numbers *not* show: how many seeds, how many shops, which settings of the learner you did
  not vary? Which of your differences are smaller than the spread between seeds?
- How uncertain is the ranking, and what would you run next to make it more reliable? Say how this
  tempers the claim you made in section 4.

## 6. AI-use declaration

Half a page: which AI tools you used in this assignment and for what. If you used none, say so. The rules
are on the course's AI use policy page in Canvas.
