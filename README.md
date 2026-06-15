# Insurance Claim Arrival & Ruin Probability Simulation

This repository implements a quantitative risk analytics project using **Monte Carlo Simulation** and **Stochastic Processes** to model an insurance company's surplus dynamics and estimate its financial ruin probability.

Two distinct methodologies are explored: an **Event-Based Approach** and a **Time-Step Approach**.

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

*Example Migration:* An arrival sequence of arrival epochs $\{T_i\}_{i=0}^{5}=(0,2,5,8,14,26)$ indicates that the first claim occurs at month 2, and the second at month 5. The continuous simulated paths are generated in [`count.py`](./count.py) and visualized in [`graph_1.jpg`](./graph_1.jpg).

### Monte Carlo Method & Ruin Probability
By the **Weak Law of Large Numbers (WLLN)**, the sample mean of independent trials converges to the theoretical expected value as the sample size $n \to \infty$. We utilize an **Indicator Function** ($\mathbb{1}_A$) to define the financial ruin event $A$ (where surplus drops below zero, $U(t) < 0$):

$$\mathbb{1}_A(\omega) = \begin{cases} 1 & \text{if } U(t) < 0 \text{ for some } t \in [0, T] \\ 0 & \text{otherwise} \end{cases}$$

The total ruin probability $\psi(u)$ is estimated by generating $n$ simulated capital paths:
$$\psi(u) \approx \frac{1}{n} \sum_{k=1}^{n} \mathbb{1}_{A}(\omega_k) = \frac{\text{Total Ruin Events}}{n}$$

---

## 2. Project Structure & Variables

### Variable Initialization
*   `initial_capital` ($u$): Starting funds of the insurance pool.
*   `premium_rate` ($c$): Continuous premium inflows collected from policyholders.
*   `arrival_rate` ($\lambda$): Poisson intensity controlling claim frequencies.
*   `num_simulations` ($n$): Total Monte Carlo path trajectories (e.g., $n = 10,000$).

### Script Descriptions
*   [`count.py`](./count.py): Simulates the Poisson arrival process and logs timestamps.
*   [`simulation.py`]: Core function executing the event-based simulation and computing ruin frequencies.
*   [`graph_1.jpg`](./graph_1.jpg): Visualized sample trajectories of the capital pool.
