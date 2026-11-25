# DECOMPOSITION — the task in named parts, before any code

**The task:** given the shop, obtain an ordering policy by each of three routes (order-up-to sweep,
value iteration, Q-learning), score all three by one rule, see what changes when one condition of the
shop changes, and recommend one route in writing.

## 1. The subproblems

| # | Subproblem | What it does |
|---|---|---|
| S1 | **Shop definition** | Fixes one shop (`ShopParams`: capacity, prices, demand, fee `K`, γ = 0.95, start stock 5). Everything else reads it; nothing else changes it. |
| S2 | **Scoring** | Turns any legal policy into one number: its exact expected discounted return from stock 5 in the endless shop, and regret = optimum − score. |
| S3 | **Route A: order-up-to sweep** | Simulates every level `S = 0..10` for some episodes and keeps the level with the best mean simulated return. |
| S4 | **Route B: value iteration** | From the model (`P`, `R`) computes the one-step lookahead `Q(s,a)` (the *backup*, which I write), repeats until `V` settles, returns the greedy policy. |
| S5 | **Route C: Q-learning** | Lives in the simulated shop day by day; chooses orders ε-greedily and updates one `Q(s,a)` per day (both of which I write); returns the greedy policy of the final table. |
| S6 | **Comparison under one rule** | Runs S3–S5 on the same shop, γ and seeds, scores each with S2, reports mean / sd / min / max of regret and the cost (days, backups, seconds). |
| S7 | **Changed condition** | Repeats S6 on a shop with a delivery fee `K > 0`, and gives the sweep and Q-learning the same budget of days (1,000 to 100,000). |
| S8 | **Recommendation** | Reads the outputs of S6–S7 and argues for one route, for a stated kind of manager, with the conditions that would reverse it. |

**Completeness and overlap.** The task statement has five verbs: *obtain* (S3, S4, S5), *score* (S2),
*compare* (S6), *change a condition* (S7), *recommend* (S8); S1 is the shared object they all act on. A
reader can check completeness by mapping each verb to a row, and check overlap by noting that only S2
computes a score and only S3–S5 produce policies.

**Why not fewer.** S2 could be folded into each route, but then each route would grade itself and the
comparison would not be under one rule. S6 and S7 could be one step, but S7 adds a new axis (the day
budget) that S6 does not have, so I keep them apart.

## 2. Inputs, outputs and data flow

| # | Input | Output |
|---|---|---|
| S1 | numbers from the README | `params: ShopParams`, `gamma: float` |
| S2 | `params`, `policy: TablePolicy` (11×11, rows are distributions over *legal* orders), `gamma` | `score: float` (tenge); `regret = optimum − score` |
| S3 | `params`, `gamma`, `n_episodes: int`, `seed: int` | `policy` (order-up-to-S), chosen `S`, days used = 11 · n_episodes · 100 |
| S4 | `P: (11, 11, 11)` with `P[a, s, s']`, `R: (11, 11)` with `R[s, a]`, `mask: bool (11, 11)`, `v: (11,)`, `gamma` | backup returns `q: (11, 11)`; driver returns greedy `policy`, number of sweeps |
| S5 | environment (state = stock `int`, action = order `int`, reward `float`), `mask`, `alpha`, `epsilon`, `n_steps`, `seed` | `q: (11, 11)` (−inf on illegal orders), greedy `policy` |
| S6 | outputs of S3–S5 for seeds 0–9 | table of score/regret mean, sd, min, max; env steps, backups, seconds |
| S7 | `ShopParams(order_fixed_cost=K)`, budgets in days | same table at fee `K`; regret vs. days for S3 and S5 |
| S8 | tables of S6–S7 | written choice |

**Data flow.**
1. S1 → `params`, `gamma` to every other step.
2. S1 → model `P`, `R` → S4 → optimal policy → S2 → **optimum** (the reference for all regrets).
3. S1 → simulator → S3 and S5 → their policies → S2 → scores → regrets.
4. S2 outputs for all routes → S6 table; S7 repeats 1–4 with fee `K` and with budgets.
5. S6 + S7 → S8.

**When an input is wrong or a result unusable.**
- S4: if values have not settled after the sweep limit, stop with an error rather than return a policy;
  a backup of the wrong shape or with non-finite values on a legal order is an error.
- S5: an illegal order must never be chosen (choose only among `mask[s]`); illegal entries of `q` stay −inf
  and never enter the max. Updates change exactly one entry.
- S2: a policy that puts probability on an order that does not fit is rejected, not scored.
- S6/S7: fewer than two seeds is rejected (no spread to report).

## 3. Constraints, assumptions and the reason for the split

**Constraints.**
- S3 and S5 may use the simulator only; they must not read `P` or `R`. S4 may use `P` and `R` but no days.
- All routes share the shop, γ = 0.95, the scoring rule and the seeds; only the route differs.
- S5: the calendar cut at day 100 is truncation, so the last day's update still bootstraps from the next stock.
- Every reported number must come from the notebook run.

**Assumptions and where they matter.**
- Zero lead time and lost sales, so the stock alone is a Markov state (matters for S4 and S5: both rely on it).
- The demand distribution is stationary (S5's experience stays relevant; S4's model stays correct).
- The model given to S4 is exact (in a real shop it would have to be estimated).
- The endless-shop discounted return is the right yardstick (S2), though the lecture used 100-day episodes.

**Why this split.** It separates *what produces a policy* (S3–S5) from *what judges it* (S2), so
every route is judged identically and the routes are interchangeable boxes with the same output type.

**Alternative considered and rejected:** splitting by *model-based vs. model-free* (one part = value
iteration, one part = "everything that simulates", with scoring done by simulation inside each part).
Rejected because scoring by simulation adds noise to the yardstick itself, and because the sweep and
Q-learning differ in what they can represent (only order-up-to rules vs. any table), which that split hides.

**The assumption that would hurt most if wrong:** that the stock on the shelf is the whole state. With
a delivery delay, orders on the road matter, the 11-state table is no longer Markov, and both value
iteration and tabular Q-learning would be optimising the wrong problem.
