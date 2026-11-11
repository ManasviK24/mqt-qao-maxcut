# Experiment design and interpretation

## Research question

How do QAOA and simulated annealing perform on a small set of weighted Max Cut problems when checked against exact enumeration, and what happens to the recorded QAOA solve time when the depth increases?

This repository is a practical comparison and correctness demonstration. It is not a quantum speedup benchmark.

## Problem formulation

Given a weighted undirected graph with vertices V and edges E, choose binary vertex labels xᵢ. For an edge (i, j) of weight wᵢⱼ, the term wᵢⱼ(xᵢ + xⱼ − 2xᵢxⱼ) is equal to wᵢⱼ when xᵢ and xⱼ differ, and to zero when they agree. Summing these terms therefore gives the Max Cut objective. The implementation constructs this maximization objective with MQT QAO.

## What runs were recorded?

| Instance | Graph generation | QAOA settings | Exact check |
| --- | --- | --- | --- |
| Example graph | 4 vertices, 6 specified weighted edges in `src/maxcut_qao.py` and `src/sweep_qaoa.py` | repetitions 1, 2, 3; `num_runs=5` in sweep | Yes |
| Random graph A | `n=8`, edge probability `0.6`, seed `42`; uniform edge weights from 0.5 to 1.5 | repetitions 1, 2, 3; `num_runs=5` | Yes |
| Random graph B | `n=10`, edge probability `0.6`, seed `7`; uniform edge weights from 0.5 to 1.5 | repetitions 1, 2, 3; `num_runs=5` | Yes |

The random graph script seeds Python's random module and the NetworkX graph generator. It does **not** explicitly seed every stochastic component inside MQT QAO. Consequently the same graph can be regenerated, but identical solver outcomes and timings are not guaranteed.

## Validation

Each solver returns an assignment. The script computes the corresponding graph cut directly using `cut_value`. For graphs with up to 10 vertices, `brute_force_maxcut` enumerates all binary assignments and reports the largest observed cut value. This exhaustive value is used as an exact reference.

The printed exact check uses numerical tolerance in the demonstration script. The generated summary identifies equality to the exact value using tolerance `1e-6`.

## How to interpret the CSVs

The columns are:

* `reps`: QAOA circuit repetition count.
* `num_runs`: internal QAOA solver run parameter, not a count of independent observations stored in the CSV.
* `qaoa_cut`: cut score of the reported QAOA solution.
* `qaoa_time_s`: wall clock time around the QAOA solver invocation.
* `sa_cut`: cut score of the reported simulated annealing solution, reused across QAOA depth rows for that graph.
* `exact_cut`: exhaustive optimum when computed; empty for graphs beyond the exact check threshold.

The CSVs do not include all intermediate samples, QAOA optimizer trajectories, SA timings, environment specifications, confidence intervals or variability estimates.

## Findings

The committed CSV files show that each recorded QAOA depth reached the exact solution on the three graphs and that the SA baseline did so as well. Increasing depth from 1 to 3 raised recorded QAOA time:

| Instance | Depth 1 | Depth 3 |
| --- | ---: | ---: |
| Four vertex example | 2.93 s | 31.58 s |
| Eight vertex graph | 9.65 s | 71.22 s |
| Ten vertex graph | 19.55 s | 238.45 s |

The recorded best cut remained unchanged. It would be incorrect to infer that deeper QAOA is never useful, that the methods have equivalent success probabilities, or that either method has a general runtime advantage from this dataset.

## Directions for a stronger benchmark

1. Use a larger and more varied collection of weighted and unweighted graph instances.
2. Run independently seeded repetitions of each configuration and report success rates, distributions and confidence intervals.
3. Record classical and quantum solver timings consistently, including environment and optimizer configuration.
4. Save raw solution samples and optimizer traces, not only the best cut value.
5. Distinguish simulator benchmarks from hardware experiments, including shot count and noise assumptions when relevant.
6. Extend exact validation with scalable bounds or established reference solvers for larger graphs.

## Source of truth

The scripts in `src/` define the experimental workflow. The CSV files in `results/` preserve the recorded observations. The figures were generated during those runs. No new numerical experiment was performed for this documentation refresh.
