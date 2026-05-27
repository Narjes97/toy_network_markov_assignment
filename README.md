# Semi-Dynamic Markovian PFE Toy Network

This repository contains a Python/Jupyter implementation of a semi-dynamic Markovian path flow estimator (PFE) on the grid toy network from Shimamoto and Kondo (2020), which builds on Chen et al. (2009). The notebook reproduces the grid-network setup, validates assignment results using the paper's true OD matrix, and extends the assignment analysis with congestion-sensitive BPR/logit-SUE and finite-horizon Markov truncation experiments.

## Project purpose

The main goal is to study how OD demand is translated into link flows under an absorbing Markov/logit assignment framework, and how different modeling choices affect the resulting link-flow accuracy. The notebook includes:

- Grid network construction using the paper's link capacities and free-flow travel times.
- True OD matrix and observed link-count scenarios from the paper.
- Absorbing Markov/logit assignment without explicit path enumeration.
- Semi-dynamic residual-flow carryover between time periods.
- Iterative balancing for inconsistent observed traffic counts.
- BPR-based congestion-sensitive stochastic user equilibrium (SUE) validation.
- Finite-horizon/truncated Markov assignment experiments using a maximum path depth `K`.
- OD-order and starting-point sensitivity tests.

## Notebook

Main notebook:

```text
toy_network_main_pfe_with_bpr.ipynb
```

## Main methods implemented

### 1. Absorbing Markov/logit assignment

The assignment method computes destination-specific transition probabilities without enumerating all paths. For each destination, the model calculates downstream attractiveness and then loads OD demand forward through the network.

Main functions:

```python
build_W_matrix()
compute_markov_value_matrix()
compute_transition_probabilities_matrix()
assign_single_od_absorbing_markov()
assign_all_od_absorbing_markov_by_origin()
```

### 2. BPR congestion-sensitive assignment

The BPR function converts link flow into congested travel time:

```text
t_a = t_a^0 * (1 + alpha * (x_a / C_a)^beta)
```

where:

- `t_a` = congested link travel time
- `t_a^0` = free-flow travel time
- `x_a` = link flow
- `C_a` = link capacity
- `alpha = 0.15`
- `beta = 4.0`

Main functions:

```python
bpr_travel_time()
update_bpr_travel_times()
assign_true_od_bpr_sue_validation()
run_bpr_logit_equilibrium_assignment()
```

### 3. Semi-dynamic PFE with iterative balancing

The PFE component estimates OD demand by minimizing a link-based objective while accounting for observed link-count inconsistency. Residual demand is carried from one time period to the next.

Main functions:

```python
solve_subproblem_iterative_balancing()
solve_pfe_subproblem_with_bpr_outer_loop()
pfe_objective_for_q()
estimate_period_q()
run_scenario_paper_exact()
main()
```

### 4. Finite-horizon Markov assignment

To reduce the unrealistic infinite-propagation assumption in the absorbing Markov formulation, the notebook includes a truncated assignment version:

```text
V_K = I + W + W^2 + ... + W^K
```

where `K` is the maximum number of transitions allowed.

Main functions:

```python
compute_destination_value_functions_truncated()
compute_transition_probabilities_truncated()
assign_single_od_markov_truncated()
assign_all_od_markov_truncated()
test_truncated_assignment()
build_truncated_flow_table()
```

### 5. Sensitivity tests

The notebook includes experiments for:

- Different initial flow assumptions under BPR/logit-SUE.
- Different finite-horizon values of `K`.
- Different OD loading orders.

Main functions:

```python
compare_bpr_assignment_starting_points()
test_bpr_truncated_K_values_safe_v2()
test_od_order_sensitivity()
```

## Data used

The grid network is hard-coded in the notebook based on the paper's toy network tables.

### Network inputs

Each link includes:

- link ID
- from-node
- to-node
- capacity
- free-flow travel time

Defined in:

```python
build_grid_network()
```

### True OD matrix

Defined in:

```python
build_true_od()
```

OD pairs:

```text
(1,6), (1,8), (1,9),
(2,6), (2,8), (2,9),
(4,6), (4,8), (4,9)
```

### Observed-count scenarios

Defined in:

```python
build_grid_scenarios()
```

Scenarios include:

1. Error-free observed link volumes in all time periods.
2. Period-1 observed link volumes reused in later periods.
3. Noisy period-1 observed link volumes reused in later periods.

## Key outputs

The notebook reports:

- Estimated link flows.
- OD estimates by time period and scenario.
- Link RMSEP for all, observed, and unobserved links.
- OD RMSEP and coefficient of determination.
- Total estimated OD volume.
- Flow tables by finite-horizon `K`.
- Sensitivity tables for initial flows and OD assignment order.

## Evaluation metric

The main accuracy metric is RMSEP:

```text
RMSEP = 100 * sqrt( mean( ((estimated - true) / true)^2 ) )
```

This measures the average percentage deviation between estimated and true values.

Implemented in:

```python
compute_rmsep()
compute_od_rmsep()
```

## How to run

1. Open the notebook in Jupyter, JupyterLab, or VS Code.
2. Install required Python packages if needed:

```bash
pip install numpy pandas scipy
```

3. Run all cells from top to bottom.

## Suggested workflow

1. Run the base network and OD setup cells.
2. Run the absorbing Markov assignment validation.
3. Run the BPR/logit-SUE validation.
4. Run the full PFE scenario analysis with:

```python
all_results = main()
```

5. Generate paper-style tables using:

```python
build_paper_table4_like()
build_paper_table5_like()
build_link_rmsep_table()
```

6. Run sensitivity experiments for:
   - starting flow assumptions,
   - finite-horizon `K`,
   - OD assignment order.

## Important modeling notes

- The absorbing Markov assignment is the core route-choice mechanism.
- BPR is used as a congestion-sensitive travel-time extension.
- The finite-horizon `K` experiment is an extension that limits the maximum number of allowed transitions.
- In fixed-cost assignment, OD loading order does not affect final link flows because costs do not update during assignment.
- In BPR equilibrium assignment, starting flows may affect convergence speed, but the tested final flows were stable across multiple initializations.
- Very small `K` values may be infeasible or unstable because they do not allow enough path depth to represent realistic OD movements.

## Repository structure suggestion

```text
.
├── README.md
├── toy_network_main_pfe_with_bpr.ipynb
└── results/
    └── optional_output_tables.csv
```

## Citation

This work is based on the grid-network example and modeling framework discussed in:

Shimamoto, H., & Kondo, A. (2020). *Semi-dynamic Markovian path flow estimator considering the inconsistencies of traffic counts*. Asian Transport Studies, 6, 100017.

The notebook also references ideas from Chen et al. (2009) on PFE with inconsistent traffic counts.

## Author note

This notebook is intended for research exploration and academic discussion. Some components, such as the BPR-based equilibrium extension and finite-horizon Markov truncation, are methodological extensions beyond the strict toy-network reproduction.
