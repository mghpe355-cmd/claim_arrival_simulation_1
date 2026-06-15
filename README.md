# Insurance Claim Arrival & Ruin Probability Simulation

This repository implements a quantitative risk analytics project using **Monte Carlo Simulation** and **Stochastic Processes** to model an insurance company's surplus dynamics and estimate its financial ruin probability.

The methodology applied in this project is **Event-Based Approach**. 

---

## 1. Theoretical Framework & Notations

### The Insurance Surplus Process (Cramér-Lundberg Model)
We model the insurance financial pool's capital movement over time $t$ using a classical surplus process:
$$U(t) = u + ct - \sum_{i=1}^{N(t)} X_i$$

Where:
*   $U(t)$: The surplus (remaining capital) at time $t$.
*   $u$: Initial capital ($U(0) = u$).
*   $c$: Premium income rate per unit time.
*   $N(t)$: A **Counting Process** representing the total number of claim arrivals up to time $t$.
*   $X_i$: The claim size (severity) of the $i$-th claim.

### Counting Process (Homogeneous Poisson Process)
We assume that claim arrivals follow a **Homogeneous Poisson Process** with a constant arrival intensity $\lambda$. 
*   The interarrival times between successive claims, $\{ \tau_i \}$, are Independent and Identically Distributed (i.i.d.) exponential random variables: $\tau_i \sim \text{Exp}(\lambda)$.
*   Consequently, the probability of having exactly $n$ claims within time $t$ follows a Poisson distribution: $N(t) \sim \text{Po}(\lambda t)$.

*Simple Example:* An arrival sequence of arrival epochs $\{T_i\}_{i=0}^{5}=(0,2,5,8,14,26)$ indicates that the first claim occurs at month 2, and the second at month 5. The continuous simulated paths are generated in [`count.py`](./count.py) and visualized in [`graph_1.jpg`](./graph_1.jpg).

### Monte Carlo Method & Ruin Probability
By the **Weak Law of Large Numbers (WLLN)**, the sample mean of independent trials converges to the theoretical expected value as the sample size $n \to \infty$. We utilize an **Indicator Function** ($\mathbb{1}_A$) to define the financial ruin event $A$ (where surplus drops below zero, $U(t) < 0$):

$$\mathbb{1}_A(\omega) = \begin{cases} 1 & \text{if } U(t) < 0 \text{ for some } t \in [0, T] \\ 0 & \text{otherwise} \end{cases}$$

The total ruin probability $\psi(u)$ is estimated by generating $n$ simulated capital paths:
$$\psi(u) \approx \frac{1}{n} \sum_{k=1}^{n} \mathbb{1}_{A}(\omega_k) = \frac{\text{Total Ruin Events}}{n}$$

---



## 2. Project Structure & Variables

### Variable Initialization
To maintain reproducibility across simulations, a pseudo-random number generator seed is locked at `123` (`np.random.seed(123)`). The baseline stochastic model parameters are initialized as follows:

*   `initial_capital`($u$) : The initial liquidity pool available to the insurer at time $t = 0$. Set to `2` units (representing 2000 HKD).
*   `premium_rate` ($c$): The continuous premium inflow rate collected from policyholders per unit time. Set to `0.1` units per month.
*   `arrival_rate` ($\lambda$): The hazard rate (intensity parameter) of the homogeneous Poisson process governing claim frequencies. Set to `0.5`, indicating an expected arrival frequency of 0.5 claims per month.
*   `max_time` ($T$): The finite evaluation time horizon for each Monte Carlo trajectory path. Set to `20` units (months).
*   `claim_amt`: The deterministic severity magnitude ($X_i$) assigned per incoming insurance claim. Set to `0.4` units.
*   `claim_rate`: The  parameter of Poisson process describing the arrival of insurance claims. Set to `0.3`.
*   `num_simulations` ($num_sim$): The total number of independent Monte Carlo path trajectories simulated to compute statistical expectations. Set to `10,000` iterations for robust convergence.

---


## 3. Simulation Framework

### Method 1: Event-Based Simulation Approach
The simulation framework employs an event-driven discrete simulation engine rather than a fixed-interval time grid. The model shifts the continuous time clock directly to the next earliest occurring stochastic event. 

The analytical workflow executes in a two-stage pipeline: **Single-Path Trajectory Diagnostics** to visually profile the system behavior, followed by a scaled **Monte Carlo Simulation** to compute the empirical default/ruin probability.

---

### Stage 1: Single-Path Trajectory Diagnostics
Before running large-scale aggregations, single trajectories are isolated and evaluated using step plots (`where='post'`) to adhere to real-world operational dynamics. 

The script below generates two visual benchmarks:
1. **Capital Trajectory Profile**: Plots continuous reserve capital against time to depict 1 possible path of change in capital reserve
2. **Operational Volume Correlation**: Co-plots the active customer pool size and accumulated claims over the time horizon to visualize asset-liability coupling.


### Stage 2: Algorithmic Execution Matrix (Monte Carlo)
To compute the overall default metrics, the simulation scales up to 10⁵ independent paths. The algorithm systematically avoids edge-case infinite loops and timeline distortions by maintaining synchronized structural updates across all states:

1. **Interarrival Sampling**: Generate stochastic waiting times for customer arrivals (\(\tau_a\)) and claims (\(\tau_c\)) using inverse transform sampling from memoryless Exponential distributions.
2. **Dynamic Event Dispatching**: Evaluate \(\min(\tau_a, \tau_c)\) to determine if the next systemic transition is a new policy onboarding or a claim payout.
3. **Temporal Progression**: Advance the continuous global clock directly to the winning epoch: \(t_{new} = t_{old} + \min(\tau_a, \tau_c)\).
4. **Endogenous Capital Updating**: Accrue interest-bearing premiums based on the active customer footprint during the elapsed interval, then apply discrete subtractions if a valid claim occurs.
5. **Boundary Solvency Evaluation**: Assess the system against the strict ruin boundary (U(t) ≤ 0). If triggered, flag the trajectory instantly (\(\mathbb{1}_{\text{Ruin}} = 1\)) and break the loop.
6. **Monte Carlo Aggregation**: Scale the loop across 10⁵ paths to converge on a stable empirical 
```


## 4. Results and Discussion

### 4.1 Parameter Rationalization & Mathematical Viability
To ensure the simulation reflects realistic macro-financial risk pooling, the hyperparameters were engineered to capture a micro-insurance or extended-warranty portfolio exhibiting steady-state viability under high stochastic variance:
*   **Operating Horizon ($T = 20$):** Simulates a bounded 20-month product lifecycle typical of high-end electronics extended warranty programs.
*   **Asset-Liability Equilibrium:** Under an infinite-server Markovian queue queueing setup ($M/M/\infty$), the long-term expected active customer size converges to $\mathbb{E}[N] = \lambda / \mu = 0.5 / 0.3 \approx 1.67$ active policies.
*   **Expected P&L Drift:** The expected continuous premium collection rate is $\mathbb{E}[c] = 0.1 \times 1.67 = 0.167$ per unit time, while the expected claim payout drain is $\mathbb{E}[S] = 0.3 \times 0.4 = 0.120$. 

Because $\mathbb{E}[c] > \mathbb{E}[S]$, the risk pool satisfies the **Net Profit Condition**, establishing a positive drift where the business is structurally profitable over a long horizon.

### 4.2 Empirical Solvency Baseline
Running $10^5$ independent stochastic trajectories yields an empirical **Ruin Probability of $\hat{P}(\text{Ruin}) = 0.00662$**. 

This indicates that over a 20-month operational lifecycle, the portfolio maintains a **0.662% probability of insolvency**. In institutional risk frameworks (such as Solvency II), keeping default risk under 1% proves that an initial capital allocation of $U(0) = 2.0$ provides an exceptionally resilient safety margin against catastrophic left-tail claim clusters. Using the standard error formula for Bernoulli trials:
$$SE = \sqrt{\frac{\hat{p}(1-\hat{p})}{N}} = \sqrt{\frac{0.00662 \times 0.99338}{100,000}} \approx 0.000256$$
The resulting **95% Confidence Interval is $[0.00612, 0.00712]$**, proving the numerical stability and statistical convergence of the engine.

---

### 4.3 Algorithmic Paradigm: Event-Based vs. Time-Based Discretization
There are 3 dimensions to discuss whether Event-Based apprach or Time-based approach should be used.

*   **Temporal Precision:** The event-based approach guarantees **exact precision**. It jumps directly to event epochs ($\tau = \min(\tau_a, \tau_c)$), capturing the precise sub-interval premium revenue earned. Conversely, time-based grids constrain events to fixed intervals ($\Delta t$), truncating intra-step compound events and skewing the ruin time.
*   **Mathematical Rigor:** By tracking exact continuous-time stochastic horizons, the system preserves the true memoryless Markovian properties of the Poisson and Exponential processes, eliminating numerical grid leakage entirely.
*   **Computational Efficiency:** The event-based algorithm scales at $\mathcal{O}(E)$, where $E$ is the total number of events. Loops execute *only* when a systemic state change occurs. Time-based stepping scales at $\mathcal{O}(T/\Delta t)$, wasting massive CPU cycles processing empty time blocks when event density is low. Conversely, if the frequency of events is high, time-based approach should be applied to estimate the total number of events, in the expense of mathematical rigor.
---

### Script Descriptions
*   [`count.py`](./count.py): Simulates the Poisson arrival process and logs timestamps.
*   [`simulation.ipynb`](./simulation.ipynb): Demonstration on the Simulation framework of Event-based approach of the project.
*   [`graph_1.jpg`](./graph_1.jpg): Visualized sample trajectories of the capital pool.
