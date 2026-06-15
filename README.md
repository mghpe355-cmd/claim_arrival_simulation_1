# claim_arrival_simulation_1
Imagine that you owe an insurance company that a certain amount of people has bought one single insurance package. Simulate change in capital and/or default probability of the package using monte carlo method.
假設有個保險公司只賣一個保險產品，欲以 Monte Carlo (蒙地卡羅) 方法估計保險資金池的 _金額變化_ 及/或 _倒閉概率_.

I will first do a method called *event-based*  approach, followed by another method.

## 1. Background and Notations
Assume the *interarrival time* between insurance claim follows a (homogenous) Poisson process, where the interarrival time follows Exp($\lambda$), which is the exponential distribution with a constant $\lambda$. Alternately, the number *N* of events occured during a period of time t follows a Poisson distribution, denoted as $Po(\lambda t)$.


The remaining section describes the basic definitions of Monte Carlo method and Counting Process (=Poisson Process).
### Monte Carlo method
Nothing but Weak law of Large Numbers (WLLN): when the sample size n is large enough (ideally n-> $\infty$), sample mean converges to (will be approximately the same as)  population mean. We apply this to the indicator function.

Define the *indicator function* ($\mathbf{1}_{A}$) as below:<br>

$$
\mathbb{1}_A(x) =
\begin{cases}
1 & \text{if } x \in A \\
0 & \text{if } x \notin A
\end{cases}
$$

Simply put down a number 1 if event A occurs, 0 otherwise. Expectation $mathbb{E}(mathbb{1}_A(x)) =Pr(A happens)\times 1 + Pr(A does not happen) \times 0 =Pr(A)$. 

To estimate a probability using Monte Carlo simulation, a large number n of experimental results are generated. The desired quantity is $Pr(A)=\frac{\text{Number of times A occured in the n trials}}{n}$. <br>

大白話:
*Indicator function* 即係如果事件A發生左,函數值就係1,否則就係0;
用 Monte Carlo 估算概率 Pr(A) 就係先做n次隨機實驗 (如扔骰子 rolling a die),觀察 A總共發生左幾多次 (記次數為$n_A$), 再輸出答案
$Pr(A)=\frac{n_A}{n}$.

### Counting Process
Suppose a sequence $\{T_i\}_{i=0}^{\infty}$ denotes the time (a positive continuous random variable) of $i^{th}$ event happening. Initially, when time t=0 month, there are no insurance claims. 


Example:
$\{T_i\}_{i=0}^{5}=(0,2,5,8,14,26)$ incidaces the first insurance claim happens at 2 months after the product is released, the second insurance claim arrives in 5 months after the product is established, etc. Relevant Python codes can be seen in count.py and grpah_1.jpeg. 

## Variable initialization



## The simulation function



## Results

