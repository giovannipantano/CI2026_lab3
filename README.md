# Computational Intelligence — Lab 3: Children in a Circle

This repository contains the study, implementation, and empirical comparison of optimization algorithms applied to the **"Children in a Circle"** combinatorial problem. The project benchmarks a local search baseline (**Random Mutation Hill Climbing**) against multiple configurations of a population-based **Evolutionary Algorithm (EA)** operating under the *Hyper-Modern GA / GP* workflow.

---

## 1. Problem Formulation

The problem requires arranging $N = 30$ children in a closed circle (round dance). Each child is defined by an identifier and an array of $C = 15$ boolean features.

### Objective Function (Circular Fitness)

The goal is to **maximize the overall phenotypic diversity** between adjacent children along the circle. Diversity between two individuals is quantified using the **Hamming distance** (the number of differing boolean traits).

Given a circular arrangement $\mathbf{S} = [c_0, c_1, \dots, c_{N-1}]$, the fitness function with circular wrap-around is defined as:

$$
\text{Fitness}(\mathbf{S}) = \sum_{i=0}^{N-1} \text{HammingDistance}\left(c_i,\, c_{(i+1) \pmod N}\right)
$$

### State Space & Symmetry Breaking

A naive permutation of $N$ children yields a search space of size $N! \approx 2.65 \times 10^{32}$. Because the arrangement forms an unoriented, closed loop, numerous distinct permutations correspond to geometrically identical arrangements:

1. **Rotational Invariance ($N$ equivalent permutations):**
   * *Resolution:* We anchor the first child (`child_00`) at index $0$ across all solutions. Only the sublist of the remaining $N-1$ elements is subject to permutation, reducing the space to $(N-1)!$.
2. **Reflectional / Mirror Invariance ($2$ orientations: clockwise vs. counter-clockwise):**
   * *Resolution:* We apply a canonical mapping function (`canonicalize`). If the right-hand neighbor (`solution[1]`) has a lexicographically greater identifier than the left-hand neighbor (`solution[-1]`), the inner sequence `solution[1:]` is reversed:
   $$\text{if } \text{id}(\text{solution}[1]) > \text{id}(\text{solution}[-1]) \implies \text{reverse}(\text{solution}[1:])$$
   This guarantees a single unique canonical representation for every circular orbit.

---

## 2. Algorithmic Frameworks

### Random Mutation Hill Climbing (RMHC) Baseline
* Operates on a single candidate solution.
* Generates a neighbor via random 2-swap mutation (`tweak`) and accepts it if and only if it strictly improves fitness (first-improvement local search).
* Stops when reaching a predefined steady state (e.g., no improvement for $800$ steps).

### Hyper-Modern Evolutionary Algorithm (EA) Architecture
All evolutionary implementations follow the **Hyper-Modern GA / GP reproduction workflow**:
1. **Operator Selection:** An operator is selected stocastically based on a probability distribution $\{p_i\}$.
2. **Dynamic Parent Selection:** Parents are sampled via **Tournament Selection** ($\tau = 3$) :
   * **(Mutations):** Samples 1 parent.
   * **(Crossovers):** Samples 2 parents.
3. **Survivor Selection:** Deterministic $(\mu + \lambda)$ **Steady-State model** ($\mu = 50$, $\lambda = 30$), where parents and offspring compete directly and only the top $\mu$ individuals survive to the next generation.

---

## 3. Comparative Summary of Implementations

| Notebook File | Strategy / Methodology | Convergence Speed | Trap / Local Optimum | Best Fitness |
| :--- | :--- | :--- | :--- | :---: |
| `random_mutation_hill_climbing.ipynb` | Single-state RMHC (Swap Tweak) | Stops at step 1306 | Local optimum (no diversity) | **302** |
| `lab3_greedy_xover.ipynb` | Pure Greedy Triplet Crossover | Very fast initial convergence | Premature convergence ($\sim$Gen 180) | **318** |
| `lab3_greedy_xover_stochastics.ipynb` | Stochastic Top-$K$ Triplet Crossover | Fastest convergence | Premature convergence ($\sim$Gen 160) | **318** |
| `lab3_xover_mix.ipynb` | Portfolio: Greedy Triplet + OX Ensemble | Balanced convergence | Plateau ($\sim$Gen 420) | **320** |
| `lab3.ipynb` | Blind Order Crossover (OX) | Slow and steady convergence | Evades early local optima | **322** |

---

## 5. Key Takeaways

1. **Exploitation vs. Exploration Trade-off:** Hyper-greedy operators yield rapid early gains but dramatically shorten takeover time, leading to premature convergence.
2. **Circular Combinatorial Nature:** In circular permutation problems, information is predominantly stored in **edges/adjacencies** rather than absolute positions or isolated building blocks.
3. **Statistical Validity:** In stochastic optimization, observing higher performance in a single run (e.g., with fixed seed $42$) provides qualitative behavioral insights, but formal conclusions require multi-run benchmarking across independent seeds with statistical testing.