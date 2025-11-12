# Fair Round-Robin Tournament Scheduling

**Academic Project (INFO-H3000) | Université Libre de Bruxelles (ULB)**

*This repository details the methodology and analysis for an academic project. The source code is hosted on a private university GitLab due to academic policy.*

**Authors:**
* **Rafael Palma Santos** ([@Rafael-jar](https://github.com/Rafael-jar))
* **Alex Bataille ([@Alexei-x](https://github.com/Alexei-X))**

This project develops a system to generate fair schedules for Round-Robin tournaments. The main challenge was tackling the computational complexity of balancing home-field advantage against opponent strength.

---

### 1. The Core Problem: Defining "Fairness"

Standard tournament schedules are often unfair. A player might randomly get a "home" game advantage against all the weakest opponents, or face a draining "away-away-away" streak.

To solve this, we first had to quantify "fairness." We created a **weighted fairness score** based on three factors:

1.  **Home vs. Difficulty (60% weight):** Our most crucial factor. Schedules that allow a player to play at every strong opponent's home (or every weak opponent home) are penalized.
2.  **Pattern Homogeneity (30% weight):** Ensures all players have a similar home/away experience. It's unfair if Player A has a perfect `H-A-H-A` schedule while Player B has `H-H-A-A-H-H`.
3.  **Home/Away Alternation (10% weight):** A small penalty for streaks like `H-H` or `A-A` (ideally the best would be `H-A-H-A`). We found this was less important than the other two factors.

### 2. A Two-Part Solution

The problem is computationally hard (combinatorial explosion). We quickly found that one single algorithm couldn't solve all cases efficiently. We split our approach into two parts.

#### Part 1: The Exact Solver (for $n < 8$ players)

For a small number of players, we could find the *perfectly* optimal solution.

* **Method:** We modeled the problem as a **Linear Program**.
* **Tool:** Implemented in Python using the `PuLP` library.
* **Key Finding:** A "strict" alternation constraint (`H-A-H-A`) was too strong and often made a solution impossible. We had to relax this constraint to get realistic schedules.

#### Part 2: Metaheuristics (for $n \ge 8$ players)

For 8 players or more, the exact solver became far too slow. We developed and compared two well-known metaheuristic algorithms to find a "good enough" (near-optimal) solution in a limited time.

* **Initial Solution:** We generated a valid starting schedule using the classic **Circle Method**.
* **Exploration:** We explored new schedules using four **mutation strategies** (`swap_players`, `swap_matches`, `swap_rounds`, `flip_home_away`) and an adaptive intensity (starting high, then decreasing) to fine-tune the search.
* **Algorithms Compared:**
    1.  **Genetic Algorithm (GA)**
    2.  **Simulated Annealing (SA)**

### 3. Key Findings & Analysis

We ran extensive tests to compare the performance of GA vs. SA, both as single-objective (using our weighted score) and multi-objective (using Pareto fronts).

* **Single vs. Multi-Objective:** The multi-objective (Pareto) approaches were consistently **too slow**. They spent too much time calculating the front and not enough time exploring. The **single-objective** approach (using our weighted fairness score) found better solutions much faster.
* **GA vs. SA Performance:** We found a clear split in performance based on the number of players ($n$) and the time allowed.
    * The **Genetic Algorithm** was very fast at finding good solutions for small-to-medium problems.
    * The **Simulated Annealing** algorithm was slower to start but *outperformed* the GA in a specific "sweet spot" (around 40-70 players), given enough time (e.g., > 30 seconds).
    * For very large problems ($n > 70$), the GA became dominant again.

### 4. Our Final Hybrid Strategy

Based on this analysis, our final application automatically selects the best algorithm for the job:

| Player Count ($n$) | Best Algorithm |
| :--- | :--- |
| **$n < 8$** | **Exact Solver (PuLP)** |
| **$8 \le n \le 40$** | **Genetic Algorithm** (Single-Objective) |
| **$40 < n < 70$** | **Simulated Annealing** (Single-Objective) |
| **$n \ge 70$** | **Genetic Algorithm** (Single-Objective) |

### 5. Sources and complete analysis

The complete analysis and the bibliography can be found in the full report:
[report.pdf](https://github.com/user-attachments/files/23455237/report.pdf)
