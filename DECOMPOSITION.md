# DECOMPOSITION — the task in named parts, before any code
<!-- STUDENT: replace -->

Replace this whole template with your own text. Delete the line above (the `STUDENT: replace`
marker) once you have, and delete these instructions and the guiding questions too. Keep the
numbered headings. Which dimension each section is graded against is in the README.

**Commit this file to your fork before you touch the notebook.** The commit order is part of the
evidence: this document is meant to be written *before* any code, from the task as the README states it.
(The GitHub web editor is enough: open this file in your fork, edit, commit.)

**The task you are decomposing:** *Given the shop, obtain an ordering policy by each of three routes
(order-up-to sweep, value iteration, Q-learning), score all three by one rule, see what changes when one
condition of the shop changes, and recommend one route in writing.*

The guiding questions are prompts, not a form. Write in your own words; a table is fine where it is clearer.

## 1. The subproblems

(Dimension: Subproblem Identification and Boundaries.)

- Break the task into named subproblems. For each: what does it do, in one or two sentences?
- Do any two overlap? Is anything the task needs not covered by any of them? How would a reader
  check that your set is complete, and that no part appears twice?
- Is there a smaller set that still covers everything? If you kept two subproblems apart, why?

## 2. Inputs, outputs and data flow

(Dimension: Specification of Inputs, Outputs and Data Flow.)

- For each subproblem: what goes in and what comes out, with names and types (and shapes, where
  there are arrays)?
- How does data flow between the subproblems? Say which output feeds which input, in an order
  someone could follow. A small diagram or numbered list is fine.
- Could someone implement one subproblem from your description alone, without seeing the others? What
  does each one do when its input is wrong or its result is not usable (for example, when it does not
  converge, or when a route returns a policy that is not allowed)?

## 3. Constraints, assumptions and the reason for the split

(Dimension: Constraints, Assumptions and Justification of the Split.)

- Which constraints bear on which subproblem (what it may use, what it may not, what it must hold the
  same as the others)?
- Which assumptions are you making, and for which subproblem does each matter?
- Why this split and not another? Name **one specific alternative split you considered and rejected**,
  and say why you rejected it.
- Which single assumption would hurt your approach most if it were wrong?
