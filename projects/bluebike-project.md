# BlueBike-Project

Applies Adaptive Robust Optimization (ARO) to the BlueBike (Boston) station rebalancing problem. The model determines how many bikes to deploy at each station to minimise worst-case rebalancing cost under an uncertainty set for ridership demand. Full methodology and results in `RO_Bluebike_Project.pdf`.

## Contents

- `RO_Bluebike_Project.pdf` — full project report

## Problem formulation

Given a set of stations, historical trip demand, and a truck-based rebalancing operation, the goal is to find a bike allocation policy that:
1. Minimises expected rebalancing trips in the nominal case
2. Remains feasible under the worst-case demand realisation within an uncertainty budget

The ARO framework produces a here-and-now first-stage allocation decision that is robust to an adversarial demand perturbation, while allowing a wait-and-see recourse (rebalancing) action.

## Roadmap

- [ ] Add the optimization model code (Python + CVXPY or Gurobi)
- [ ] Add data pipeline to pull current BlueBike trip data from the public GBFS feed (`https://gbfs.bluebikes.com/gbfs/gbfs.json`)
- [ ] Add a README section summarising the key results (cost reduction, service level) from the PDF
- [ ] Add a reproducibility script that installs dependencies and replicates the main figures
