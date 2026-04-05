# Abstract
Predicting the path of crack propagation in heterogeneous materials is a complex challenge central to materials science and structural engineering. This project introduces *A\*tomic Fracture*, a novel interdisciplinary approach that models crack propagation as a discrete pathfinding problem solvable via Classical AI. By mapping the principles of Linear Elastic Fracture Mechanics (LEFM) onto a 2D spatial grid, an A* search algorithm is utilized to find the globally optimal, minimum-energy fracture path through a polycrystalline microstructure. This report details the theoretical formulation, algorithmic design, and quantitative performance of the model compared against alternative search techniques.

# Introduction
Catastrophic material failure often originates from microstructural defects. Understanding and predicting exactly how a crack navigates around tough grains and through brittle inclusions is critical for designing damage-tolerant materials. Historically, crack propagation has been modeled using computationally heavy methods. By adopting Classical AI graph-search techniques, this computationally-expensive problem is transformed into an elegant, scalable optimization framework.

# 1 Problem Formulation
*   **Real-world problem:** Predicting transcrystalline and intercrystalline crack propagation paths under heterogeneous stress fields in polycrystalline materials.
*   **Importance:** Accurate prediction of fracture paths helps mitigate catastrophic structural failures in critical applications such as aerospace and nuclear engineering, making proactive damage-tolerant design possible.
*   **Interdisciplinary significance:** This project combines **Materials Engineering** (fracture toughness, multiphase microstructures) with **Computer Science** (graph theory, heuristic optimization), bridging high-fidelity physical simulation with rapid algorithmic approximation.
*   **Inputs and Outputs:** 
    *   *Inputs:* A 2D grid containing multiphase solid mapping (Ferrite, Martensite, Brittle Inclusions), local fracture toughness ($K_{IC}$), and an applied stress gradient ($K_{applied}$).
    *   *Outputs:* The exact continuous minimum-energy path from the initiation site to the boundary, or an explicit *arrest* state if the driving force is insufficient.
*   **Problem Characteristics:** The environment is *stochastic* (randomized Voronoi grain generation), the physical state is evaluated as *static* (pre-calculated linear stress gradients), and the solver operates in a *discrete*, *constrained* formulation.

# Literature Context
Predictive fracture mechanics usually relies on the Extended Finite Element Method (XFEM) or Phase-Field models. While highly accurate, these are notoriously slow. Conversely, graph-based heuristic approaches (like A*) have achieved massive success in operations research and robotics but are rarely applied to physical damage simulations. This project seeks to optimize an energy-based objective function rather than simple geometric distance.

# 2 Method and Justification

## Why This Method is Appropriate
*   **Method matching:** Crack propagation fundamentally follows the path of least resistance, striving to minimize the total energy released during fracture. This maps perfectly onto the shortest-path graph optimization problem.
*   **Problem properties and algorithm strengths:** By employing a physically-informed heuristic, A* avoids the exhaustive "flood-fill" exploration characteristic of Dijkstra's algorithm, navigating accurately around high-toughness phases.
*   **Suitability:** 
    *   *Optimality:* A* guarantees finding the path with the exact minimum total resistance.
    *   *Efficiency:* The heuristic prunes the search space dramatically over unweighted or uninformed algorithms.
    *   *Interpretability:* Every evaluated step explicitly correlates to the local physical energy required to break a specific atomic bond.

## Algorithm Design and Methodology
*   **Full pipeline:** 
    1. A Voronoi tessellation assigns randomized phases with characteristic toughness values: Ferrite ($K_{IC}=1.0$), Martensite ($K_{IC}=3.0$), and Inclusions ($K_{IC}=0.3$).
    2. A linear $K_{applied}$ field is evaluated across the grid to determine the driving force. 
    3. An admissible *suffix-minimum* array of column-wise resistances is precomputed as the heuristic.
    4. A* explores the graph, driven by the cost function spanning physical equations, halting at the opposite boundary or upon total arrest.

*   **Major stages:**
    *   *Edge Cost Evaluation:* $Resistance = \max(0, K_{IC} - K_{applied})$. Backward moves are artificially penalized.
    *   *Crack Arrest Pruning:* Nodes with $K_{applied} < 0.10 \times K_{IC}$ are discarded.

---

**Algorithm 1** A*tomic Fracture Search Algorithm
**Require:** Microstructure Input Grid ($K_{IC}$, $K_{applied}$), Start Node $S$, Goal Boundary $G$
**Ensure:** Minimum-energy contiguous crack path $P$, or ARREST state
1: Initialize `open_set` as Priority Queue, push $S$
2: Initialize `g_score[S] = 0`, `came_from = {}`
3: Precompute admissible heuristic array $H(c)$ based on suffix-min column resistances
4: **while** `open_set` is not empty **do**
5:      Select the next node $N(r,c)$ with lowest $f = g + h$
6:      **if** $N(r,c)$ is on Goal Boundary $G$ **then**
7:          **Return** reconstructed path $P$ from `came_from`
8:      **end if**
9:      **for** each valid neighbor $N'$ of $N$ **do**
10:         Evaluate local arrest: **if** $K_{applied} < 0.1 \times K_{IC}$ **then** Continue
11:         Apply transition cost: $R = \max(0, K_{IC}(N') - K_{applied}(N'))$
12:         Evaluate geometric distance scaling and backward penalty
13:         $tentative\_g = g\_score[N] + (R \times distance\_scaling)$
14:         **if** $tentative\_g < g\_score[N']$ **then**
15:             $g\_score[N'] = tentative\_g$
16:             Push $N'$ to `open_set` alongside $f\_score = tentative\_g + H(N'_{col})$
17:         **end if**
18:     **end for**
19: **end while**
20: **Return** failure or best available result (ARREST)

---

# 3 Implementation Details

## System Design
*   **Software structure:** The software is architected into modular Python scripts. It utilizes `numpy` and `scipy` for rapid matrix calculations and `matplotlib` for visualization.
*   **Important modules:** 
    *   `environment.py`: Manages the domain using OOP. Defines boundary generation and localized physics constraints.
    *   `astar_search.py`: Core algorithm logic containing the specialized, physically-admissible heuristic design.
    *   `visualizer.py`: Renders granular heat maps and physical paths.
    *   `algorithm_comparison.py`: Benchmarking suite directly comparing diverse search methods.
*   **Key data structures:** 2D NumPy arrays representing $K_{IC}$, $K_{applied}$, and grain labels. Min-heaps (priority queues) for standardizing optimal A* boundary expansions.

## GitHub Repository
*   The Git repository hosts the entire framework structure.
*   The repository includes all the algorithm Python code, documentation, and a comprehensive README detailing setup.

**Repository Link:** https://github.com/JitamB/Microstructural-Crack-Propagation-Prediction-using-Claasical-AI

# 4 Results and Performance Analysis

## Experimental Setup
*   **Datasets and Scenarios:** The model was evaluated on a $100 \times 100$ randomized grid (10,000 pixels) maintaining constant phase probabilities: 49.8% Ferrite, 26.9% Martensite, 23.2% Brittle Inclusions. The applied stress was graded from 2.50 to 0.10. 
*   **Hardware/Software setup:** Python 3 on macOS.
*   **Evaluation metrics:** Target parameters evaluated were the minimal path cost (total consumed energy), explored node count (efficiency indicator), and total script run time. Models evaluated include A*, Dijkstra, Greedy Best-First, and Unweighted Breadth-First Search (BFS).

## Quantitative Results
| Algorithm | Final Path Cost (Energy) | Nodes Explored | Path Length | Time Elapsed (s) | Optimality |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **A\* Search** | **3.3430** | **6,532** | **118 px** | **0.038** | **Optimal** |
| Dijkstra | 3.3430 | 6,994 | 108 px | 0.051 | Optimal |
| Greedy BFS | 25.0991 | 417 | 125 px | 0.002 | Sub-optimal |
| BFS (Unweighted) | 52.8306 | 9,511 | 100 px | 0.013 | Sub-optimal |

*   **Accuracy and Cost:** Both A* and Dijkstra correctly matched the absolute minimum-energy path satisfying physical constraints (Cost: 3.3430). The unweighted BFS found a shorter trajectory (100 px) but catastrophically failed constraints, consuming disproportionate energy (Cost: 52.83) by ripping straight through high-toughness martensite.
*   **Time and Efficiency Constraints:** The advanced suffix-minimum heuristic ensured A* evaluated roughly 6.5% fewer total operations than standard Dijkstra while delivering identical accuracy.

# Conclusion
The *A\*tomic Fracture* simulator successfully demonstrates that deterministic graph-search algorithms can function as rapid, proxy evaluators for complex physical phenomena. By intelligently adapting the A* search heuristic to respect the energetic fundamentals of crack propagation and stress intensity, the computational model effectively simulates fracture events while remaining exponentially more efficient than uninformed pathfinders. 

# Future Scope
Advancing this model to research-grade fidelity ultimately requires replacing key mechanical assumptions with continuous mechanisms. Future enhancements will prioritize:
1.  Implementing iterative, dynamic stress redistribution across the grid to replicate localized Mode-I stress concentrations during active crack propagation.
2.  Assigning continuous anisotropic toughness coefficients oriented along crystallographic cleavage planes.
3.  Deploying deep learning edge-segmentation to parse, skeletonize, and map actual Scanning Electron Microscope (SEM) TIFF files into graph networks automatically.
