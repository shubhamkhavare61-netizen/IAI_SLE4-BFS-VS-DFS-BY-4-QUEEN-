# SLE-4: Final Viva & Architecture Decision Record (ADR)

## 4-Queens Solver --- Choosing DFS with Recursive Backtracking

**Course:** 02AML204 -- Introduction to Artificial Intelligence\
**Programme:** SY B.Tech. CSE (AI & ML), Semester VI\
**GitHub Repository:**
[IAI-BFS-VS-DFS-BY-4-QUEEN-](https://github.com/shubhamkhavare61-netizen/IAI-BFS-VS-DFS-BY-4-QUEEN-)

------------------------------------------------------------------------

## 1. Project Overview

The 4-Queens problem asks us to place four queens on a 4 × 4 chessboard
so that no two queens attack each other. Queens must not share the same
column or diagonal.

This project compares two uninformed search strategies:

-   **Breadth-First Search (BFS):** explores states level by level using
    a FIFO queue.
-   **Depth-First Search (DFS):** explores a path deeply using recursion
    (an implicit LIFO stack), then backtracks when needed.

The chosen main strategy is **DFS with recursive backtracking**. BFS is
retained as the comparison baseline.

## 2. State Representation and Safety Check

A state is a tuple containing the column position of each queen placed
so far, one row at a time.

Examples: - `(1, 3)` means the first queen is in column 1 and the second
queen is in column 3, using zero-based column indexes. - The two
complete solutions found are `(1, 3, 0, 2)` and `(2, 0, 3, 1)`.

The shared `is_safe()` check rejects a placement if a queen conflicts
with an existing queen in the same column or diagonal. Invalid partial
placements are not expanded.

## 3. Architecture Decision Record (ADR)

### Context

Both BFS and DFS can solve this problem. The valid partial-placement
search tree contains 17 nodes in total:

`1 + 4 + 6 + 4 + 2 = 17`

All goal states are at depth 4, so BFS's advantage of finding a
shallowest goal is not useful for this particular problem.

### Decision

Use **DFS with recursion and backtracking** as the primary search
strategy. Keep BFS for comparison. Both strategies use the same safety
rule, so the search strategy can be changed without changing the
constraint-checking logic.

### Alternatives Considered

#### Alternative 1 --- BFS (FIFO queue)

BFS explores the search tree level by level and can find a shallowest
goal. However, all 4-Queens solutions are at the same depth, so this
advantage does not help here. BFS also stores a frontier of states in
its queue.

**Decision:** Keep BFS as the baseline, but use DFS as the main
strategy.

#### Alternative 2 --- Brute Force / Generate-and-Test

With one queen assigned to each row and four possible columns per row,
brute force considers:

`4 × 4 × 4 × 4 = 4⁴ = 256` possible placements.

It generates complete placements and checks them afterward. The
row-by-row backtracking approach checks safety while building a
placement and stops exploring a branch as soon as a conflict is found.
This avoids work on invalid branches.

**Decision:** Rejected because it does not prune invalid partial
placements early.

#### Alternative 3 --- Informed Search (A\* / Greedy)

A\* and Greedy use a heuristic to guide the search. In this formulation,
the task is to find a valid arrangement, not to minimize a meaningful
path cost. The valid search tree has only 17 nodes, so introducing a
heuristic would add complexity without a clear benefit.

**Decision:** Rejected for this small problem.

## 4. Performance Evaluation

The measurements below come from the project's profiling report. Each
timing is an average for a batch of 10,000 searches, with 3 runs per
case, measured using `time.perf_counter()`.

  ---------------------------------------------------------------------------------
  Test case           BFS nodes    DFS nodes  BFS average  DFS average     DFS time
                                                     (ms)         (ms)        saved
  ---------------- ------------ ------------ ------------ ------------ ------------
  Best --- stop at           17            9      248.474      112.582      ≈ 54.7%
  `(1, 3, 0, 2)`                                                       

  Average --- stop           17           13      244.400      182.458      ≈ 25.3%
  at                                                                   
  `(2, 0, 3, 1)`                                                       

  Worst ---                  17           17      263.250      240.879       ≈ 8.5%
  enumerate all                                                        
  solutions                                                            
  ---------------------------------------------------------------------------------

### Interpretation

-   When stopping at a solution, DFS expanded fewer nodes in the best
    and average cases.
-   When enumerating all solutions, both strategies expanded 17 nodes.
-   In these measurements, DFS was faster in all three cases. This is a
    result of this experiment, not a claim that DFS is always faster for
    every problem.
-   The absolute runtime for a 4 × 4 board is small, so the repeated
    batch was used to measure it more reliably.

## 5. Profiling and Memory

The project used `py-spy` at a 200 Hz sampling rate to inspect where
Python execution time was concentrated.

-   BFS samples were spread across queue handling, state generation, and
    `is_safe()` checks.
-   DFS samples were concentrated in recursive descent, showing how it
    follows a branch toward a goal.
-   BFS stores a frontier in a queue, with typical space complexity
    `O(b^d)`.
-   DFS stores the current path, with typical space complexity `O(bd)`
    for this search-tree formulation.

Here, `b` is the branching factor and `d` is the maximum search depth.
Both algorithms have worst-case time complexity `O(b^d)`.

## 6. C4 Architecture Summary

The solver is separated into clear parts:

-   **Context:** User provides the problem/input and receives the
    result.
-   **Containers:** Input, Search Engine, Constraint Checker, Profiler,
    Output.
-   **Components:** Frontier, Goal Test, State Expander, Safety Filter,
    Solution Collector.
-   **Code-level functions:** `is_safe()`, `goal_test()`, `expand()`,
    `bfs()`, `dfs()`, `profile_run()`, and output/visualization
    functions.

This separation lets the search strategy change while the
safety-checking logic remains shared.

## 7. Consequences of the Decision

### Benefits

-   Fewer node expansions when DFS finds a solution early.
-   Lower memory use because DFS keeps the current path rather than a
    full frontier.
-   Recursive backtracking gives a direct way to abandon invalid
    branches and try alternatives.

### Trade-offs

-   DFS's advantage depends on the order in which solutions are
    encountered.
-   When all solutions are enumerated, the measured node count is the
    same for BFS and DFS.
-   DFS does not guarantee the shallowest goal.
-   The reported timings apply to the 4-Queens experiment; they should
    not be generalized to all problem sizes.

## 8. Conclusion

Based on the measured node counts, runtimes, memory characteristics, and
the small search space, **DFS with recursive backtracking is the
selected strategy** for this 4-Queens solver. BFS remains useful as a
comparison baseline. The decision is specific to this problem
formulation and the experiment's results.

## 9. Viva Quick Answers

**Why did you choose DFS?**\
DFS expanded 9 nodes in the best case and 13 in the average case,
compared with 17 for BFS. It also had lower measured runtime in all
three test cases.

**Why not brute force?**\
Brute force checks up to 256 complete placements, while backtracking
rejects unsafe partial placements early.

**Why not A\* or Greedy?**\
The problem has a small 17-node valid search tree and no meaningful
path-cost objective in this formulation, so a heuristic adds complexity
without a clear advantage.

**What is backtracking?**\
It means returning to an earlier choice when the current partial
solution cannot work, then trying another choice.

**What does `is_safe()` do?**\
It checks whether a queen can be placed without sharing a column or
diagonal with an existing queen.

**What is the main conclusion?**\
DFS with recursive backtracking is the best fit for this experiment
based on the measured results and its lower memory requirement.

## 10. AI Contribution Note

AI assistance was used to organize and word parts of the ADR and
supporting explanations. The project report states that the student
wrote and ran the BFS and DFS code, designed the test cases, measured
timings and node counts, captured the profiling flamegraphs, and chose
the decision to defend.
