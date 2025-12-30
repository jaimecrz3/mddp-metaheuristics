# Minimum Differential Dispersion Problem (MDDP) — Metaheuristics (C++)

This repository contains a C++ implementation of several trajectory-based metaheuristics to solve the **Minimum Differential Dispersion Problem (MDDP)**, a combinatorial optimization problem that is **NP-complete** and becomes expensive even for medium-sized instances.

## Problem statement


Given:
- a set of $`n`$ elements ($`n > m`$),
- a target subset size $`m`$,
- and a distance matrix $`D = (d_{ij})`$ of size $`n × n`$,

the goal is to select a subset $`S`$ of $`m`$ elements minimizing the **differential dispersion**.

### Definitions

For each selected element $`v ∈ S`$, define:

- **Delta value**
  - $`Δ(v) = Σ_{u ∈ S} d_{uv}`$

The dispersion of a solution $`S`$ is:

- **Differential dispersion**
  - $`diff(S) = max_{v ∈ S} Δ(v) − min_{v ∈ S} Δ(v)`$

Objective:

- **Minimize**
  - min diff(S) subject to $`S ⊆ V`$ and $`|S| = m`$


---

## Implemented algorithms

The project implements and evaluates the following methods (plus baselines used in the experimental analysis):

### Metaheuristics
- **BMB** — Basic Multi-Start (multiple restarts + local search)
- **ILS** — Iterated Local Search (mutation + local search)
- **SA (ES)** — Simulated Annealing (Cauchy-like cooling scheme; accepts worse moves probabilistically)
- **ILS + SA (ILS-ES)** — ILS variant using SA instead of local search
- **GRASP** — Greedy Randomized Adaptive Search Procedure
  - **GRASP + LS** (with local search)
  - **GRASP (no LS)** (construction-only, for comparison)


### Baselines (for reference/comparison)
- Random construction
- Greedy construction
- Random local search (LSrandom)
- (Referenced in the report) best previous memetic approach: **AM-(10, 0.1)**


---

## Project structure

The code is organized into four main folders:

- `common/`  
  Reusable infrastructure shared across problems and algorithms:
  - `problem.h` (abstract problem interface)
  - `mh.h` / `mhtrayectory.h` (base metaheuristic interfaces)
  - `solution.h` (type aliases for solution/fitness)
  - `random.h` (RNG utilities)
  - `util.h` (printing helpers)

- `data/`  
  Problem instances (text files) that encode:
  - problem size `n`,
  - subset size `m`,
  - distance matrix `D`.

- `inc/`  
  Header files (`.h`) for the implemented algorithms and the `mddp` problem class.

- `src/`  
  Source files (`.cpp`) for the implemented algorithms and the `mddp` problem class.


---

## Core design (how the code fits together)

### `solution.h` and `util.h`
- `solution.h` defines consistent type aliases (fitness type, domain type, and the solution container).
- `util.h` overloads `<<` for vectors to print solutions cleanly.


### `MH` and `MHTrajectory`
- Most algorithms derive from `MH` and implement:
  - `optimize(Problem* problem, int maxevals) -> ResultMH`
- Simulated Annealing derives from `MHTrajectory` so it can also be called with an existing solution + fitness (useful for the **ILS+SA** hybrid).


### `mddp` (problem implementation)
`mddp` stores:
- `n` (instance size),
- `m` (solution size),
- the distance matrix,

and implements:
- objective `fitness(solution)` (computes `diff(S)`),
- `createSolution()` (random feasible subset of size `m`),
- and supporting methods (including delta computation and factoring-related utilities discussed in the report).


---

## Build and run (CMake)

### Compile
From the folder containing `CMakeLists.txt`:

```bash
rm -f CMakeCache.txt
cmake .
make
```

## Run

- If you run without arguments, the program uses the default seeds defined in main.

- To use custom seeds, pass exactly five integer seeds as arguments. Example:

```bash
./main 42 123 1 24 98
```

- Note: results are written to a .csv file (the program may print little or nothing to stdout depending on configuration).


## Experimental setup

- **Instances:** 50 cases from **MDPLIB**, group **GKD-b** (random / real-valued distances)
- **Sizes:** `n ∈ {25, 50, 75, 100, 125, 150}`
- **Subset size:** `m` between **2 and 45** (depends on the instance)
- **Budget:** `maxevals = 100000` per run
- **Seeds:** `42, 123, 1, 24, 98`

## Results summary (high-level takeaways)

The report compares **solution quality** (deviation from best known / best obtained) and **runtime** (ms). Main conclusions:

- **Trajectory-based methods** (BMB, ILS, GRASP+LS) consistently improve over simple baselines (Random / Greedy / LSrandom) by reducing premature convergence to poor local optima.
- **ILS** provides one of the best **quality–time trade-offs**, achieving strong deviations with comparatively low runtime.
- **GRASP without local search** performs poorly in quality (as expected), while **GRASP + LS** substantially improves results.
- **Simulated Annealing (SA/ES)** does not clearly outperform LSrandom in these experiments, likely due to the neighborhood design (single swap) and/or the cooling configuration.
- The referenced **memetic algorithm AM-(10, 0.1)** achieves better average deviation but at a much higher runtime cost, so **ILS** is often preferable when efficiency matters.

