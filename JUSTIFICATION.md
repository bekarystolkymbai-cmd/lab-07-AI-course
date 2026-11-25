# JUSTIFICATION — which route, and why

All numbers below were printed by `lab07_three_routes.ipynb` and are in `results.json`. Unless stated
otherwise: γ = 0.95, scoring rule = exact expected discounted return from stock 5 in the endless shop,
ten seeds (0–9), Q-learning with α = 0.5 decaying as α / (1 + n/50), ε = 0.2, sweep with 30 episodes per level.

## 1. What was held identical

I wrote three functions: `bellman_backup` (value iteration's lookahead
`Q = R + γ · Σ_s' P[a,s,s'] V(s')`, computed as `(P @ v).T`), `epsilon_greedy` (random legal order with
probability ε, otherwise the best legal order, ties to the smaller order) and `q_update` (one in-place step
toward `r + γ · max_a' Q(s', a')` over legal `a'`, always bootstrapping because the calendar cut is
truncation). The shop, the drivers, the scoring and the comparison were supplied.

Both checks passed (`check_bellman_backup: all three tests passed`, `check_q_learning: all tests passed`).
They test the backup at `v = 0`, at constant `v` and on a lumpy `v` against a direct demand average, and
the choice/update on hand cases (greedy at ε = 0, only legal orders at ε = 1, ~80–90 % greedy at ε = 0.2,
one exact update). They do not test convergence speed or behaviour on another shop.

Held identical: the shop (`ShopParams`, then the same shop with fee K = 100), γ, `score_policy`, and seeds
0–9 for both random routes (`compare_strategies`, Parts 3 and 4b). **Not** identical: the days used —
value iteration uses 0, the sweep 33,000, Q-learning 100,000 in Parts 3/4b; Part 4c equalises days.

## 2. Why each route behaves as it does

**Value iteration — Bellman optimality backup is a γ-contraction.** Each sweep shrinks the error by at
least γ, so it must converge to the unique optimal values; it took 386 sweeps (25,476 expected backups) on
the lecture shop to reach the 1e-6 tolerance, and its policy is optimal by construction (8,460.2 tenge; at
K = 100, 7,123.3). *Where it stops applying:* it needs the exact `P` and `R`. A real manager does not have
them; with an estimated model the "optimum" is only as good as the estimate, and with a large state space
(many products, lead times) the table no longer fits.

**Order-up-to sweep — restricting the policy class.** It only searches 11 rules of the form "top up to S".
Without a fee, the optimal policy *is* such a rule (value iteration: "order up to 4 when stock ≤ 3"), so the
sweep picked level 4 on all ten seeds with regret 0.0. With a fee, ordering every day becomes expensive and
the optimum becomes an (s, S) rule — "order up to 5 when stock ≤ 2" at K = 100 — which the sweep cannot
represent: its best is level 4, regret 352.1 on every seed. More episodes cannot fix this (regret stays
352.1 from 27 to 90 episodes in 4c): it is a bias of the class, not noise. *Where it stops applying:* any
shop whose best policy is not order-up-to.

**Q-learning — stochastic approximation of the Bellman optimality equation from samples.** With every
pair visited infinitely often (ε-greedy) and decaying step sizes, Q converges to Q*; it needs no model and
can represent any table. In practice it is noisy for a finite budget: lecture shop, 100,000 days, regret
39.2 mean (sd 41.5); at K = 100, 81.8 mean (sd 86.7), with the best seeds reaching the optimum exactly.
*Where it stops applying:* when days are expensive (1,000 days → mean regret 844.1), when the state space
is too big for a table, or when demand changes faster than it learns.

## 3. What was measured

Lecture shop (K = 0), 10 seeds:

| route | score mean | sd | regret mean | env steps | backups |
|---|---|---|---|---|---|
| order-up-to sweep | 8,460.2 | 0.0 | 0.0 | 33,000 | 0 |
| value iteration | 8,460.2 | 0.0 | 0.0 | 0 | 25,476 |
| Q-learning (100,000 days) | 8,421.0 | 41.5 | 39.2 | 100,000 | 100,000 |

**Fee choice.** In 4a the best order-up-to rule's regret was 0.0, 54.3, 352.1, 983.3, 1,896.2, 2,864.5 for
K = 0, 50, 100, 200, 300, 400. I chose **K = 100** (half a unit's 200-tenge margin): it is the smallest fee in
the table where the order-up-to rule loses clearly (352.1, far beyond Q-learning's seed spread), while the
fee is still moderate, so the comparison is not decided only by an extreme setting.

Fee shop (K = 100), 10 seeds:

| route | score mean | sd | min | max | regret mean |
|---|---|---|---|---|---|
| order-up-to sweep | 6,771.2 | 0.0 | 6,771.2 | 6,771.2 | 352.1 |
| value iteration | 7,123.3 | 0.0 | 7,123.3 | 7,123.3 | 0.0 |
| Q-learning (100,000 days) | 7,041.5 | 86.7 | 6,827.3 | 7,123.3 | 81.8 |

Same days, K = 100, 10 seeds (regret in tenge):

| days | Q-learning mean | sd | max | sweep days used | sweep mean | sweep max |
|---|---|---|---|---|---|---|
| 1,000 | 844.1 | 339.8 | 1,330.3 | 2,200 | 370.9 | 540.3 |
| 3,000 | 330.4 | 286.2 | 905.5 | 2,200 | 370.9 | 540.3 |
| 10,000 | 136.0 | 152.5 | 490.8 | 9,900 | 370.9 | 540.3 |
| 30,000 | 81.1 | 64.5 | 233.2 | 29,700 | 352.1 | 352.1 |
| 100,000 | 81.8 | 86.7 | 296.0 | 99,000 | 352.1 | 352.1 |

I compare **regret** because with a fee scores can be negative in other shops and a percentage would be
meaningless, and regret reads directly as "tenge lost against the best possible". The **same seeds** mean
the routes face the same demand sequences, so a difference between them is not luck of the draw. I did not
change α, half-life or ε. Everything is reproducible by running the notebook top to bottom.

## 4. The choice, and what would reverse it

**Recommendation:** for a manager who **knows the demand distribution and the costs** (so can write down
`P` and `R`), use **value iteration**: it is exact (regret 0.0 on both shops), costs no days of
experience, and runs in about 0.02 s in Colab (against 4.1–4.2 s per Q-learning run). It beat the sweep by 352.1 tenge at K = 100 and Q-learning by 81.8
on average (up to 296.0 on the worst seed), while using zero days against their 33,000 and 100,000.

For a manager who **does not know the model and has only days**: with no delivery fee, the order-up-to
sweep is enough (regret 0.0, 33,000 simulated days). With a delivery fee, Q-learning wins once it has
around 3,000 days or more (330.4 vs 370.9 mean regret, and 81.1 vs 352.1 at 30,000), but the 3,000-day row
is within Q-learning's seed spread (sd 286.2), so only from about 10,000 days is the advantage clear. At
1,000 days the sweep's restricted, low-variance answer is better (370.9 vs 844.1). Note 10,000 real days
are ~27 years, so in a real shop these sample routes are only realistic on a simulator — which again
requires knowing the model.

**What would reverse it:** an inaccurate or unknown model (then Q-learning or the sweep on real data);
very few days (then the sweep); no fixed ordering cost (then the cheap sweep is as good as the optimum);
a state space too large for a table (then neither tabular route, but function approximation).

## 5. What this comparison cannot settle

- One shop family, one γ (0.95), one start stock, ten seeds; fees only in steps of 50–100.
- I did not vary α, the half-life or ε; the README notes a constant α behaves very differently, so the
  Q-learning numbers are specific to these defaults.
- At 100,000 days Q-learning's regret (81.8) is not better than at 30,000 (81.1); the difference is far
  smaller than the sd (64.5–86.7), so "more days helped" is not shown beyond 30,000.
- The 3,000-day ranking of Q-learning vs the sweep (330.4 vs 370.9) is inside the spread: not a finding.
- Value iteration's advantage assumes the model is exact; I did not test it with an estimated model.

Next I would run more seeds (e.g. 50) at 3,000–10,000 days, vary ε and the half-life, and test value
iteration on a model estimated from a limited number of days. That would show whether "value iteration
when you know the model" survives when the model has to be learned, which tempers the claim in §4.

## 6. AI-use declaration

I used Claude (Anthropic), an AI assistant in the Claude desktop app, for most of this assignment.

* It wrote the code of the three ✏️ functions (`bellman_backup`, `epsilon_greedy`, `q_update`) and chose the
  fee `FIXED_COST = 100`. Everything else in the notebook (the shop, the harness, the checks) is the supplied
  code, unchanged.
* It first solved and ran the notebook in its own sandbox (numpy only), before `DECOMPOSITION.md` was
  committed. So the decomposition was written by the assistant after it had already worked through the
  code; it was not written by me before any code, as the README intends.
* It then ran the notebook in Google Colab (standard CPU runtime) through the app's built-in browser, in my
  Google account, which I had signed in to myself; I also authorised Colab's access to GitHub myself. All
  outputs in the notebook and in `results.json` come from the final "Restart session and run all" in that
  Colab session. The numbers are the same as in the sandbox run; only the timings differ. No number was
  typed by hand.
* It wrote `DECOMPOSITION.md` and this `JUSTIFICATION.md` from my run's numbers, and committed all files to
  my fork, through Colab's "Save to GitHub" and the GitHub web editor.
* Why the history looks the way it does: the notebook was committed part by part from Colab (2a, 2b, 3,
  4a–4b, 5). The save after Part 4c did not go through, and the saves lost the outputs of Parts 3–4b, so the
  whole notebook was re-run from top to bottom and committed again (execution counts 1–13). `results.json`
  and this file were then updated to that final run's timings. No commit was back-dated.
* To bring `results.json` from the Colab session into the fork, the assistant added a temporary cell that
  printed the file, ran it and deleted it before saving, then pasted the printed content into the GitHub
  editor. It checked with SHA-256 that the committed file matches the one Colab wrote (GitHub only added a
  final newline).
* `Lab07-Beka.ipynb` is a copy of the same final notebook under the name I asked for; Colab saved it without
  execution counts. The file to grade is `lab07_three_routes.ipynb`.
* I asked for this help, signed in to Colab and GitHub myself, and I am responsible for what is submitted.
