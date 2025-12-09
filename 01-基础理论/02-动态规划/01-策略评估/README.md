---

# 1.2.1 策略评估 (Policy Evaluation)

---

## 1. 问题定义 (Problem Definition)

策略评估，又称为**预测问题 (prediction problem)**，其目标是计算一个给定的、固定的策略 $\pi$ 的状态值函数 $V^\pi(s)$。即，对于 MDP $(\mathcal{S}, \mathcal{A}, P, R, \gamma)$ 中的任意状态 $s \in \mathcal{S}$，我们希望计算出从状态 $s$ 开始，遵循策略 $\pi$ 所能获得的期望累积折扣回报。

$$ 
V^\pi(s) \triangleq \mathbb{E}_\pi[G_t | S_t = s] = \mathbb{E}_\pi \left[ \sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \Big| S_t = s \right] 
$$

这个计算是后续进行策略改进的基础。只有准确地知道了当前策略“有多好”，我们才能知道如何去改进它。

## 2. 迭代策略评估 (Iterative Policy Evaluation)

根据贝尔曼期望方程，我们知道 $V^\pi(s)$ 满足以下关系：
$$ 
V^\pi(s) = \sum_{a \in \mathcal{A}} \pi(a|s) \left( R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V^\pi(s') \right) 
$$

对于一个包含 $|\mathcal{S}|$ 个状态的有限 MDP，这构成了一个包含 $|\mathcal{S}|$ 个未知数 ($V^\pi(s)$ for all $s \in \mathcal{S}$) 的 $|\mathcal{S}|$ 个线性方程组。在状态空间较小时，可以直接求解该线性方程组：
$$ 
\mathbf{v}^\pi = \mathbf{r}^\pi + \gamma \mathbf{P}^\pi \mathbf{v}^\pi \implies (\mathbf{I} - \gamma \mathbf{P}^\pi) \mathbf{v}^\pi = \mathbf{r}^\pi \implies \mathbf{v}^\pi = (\mathbf{I} - \gamma \mathbf{P}^\pi)^{-1} \mathbf{r}^\pi 
$$
其中 $\mathbf{v}^\pi$ 是值函数的向量表示，$\mathbf{r}^\pi$ 是期望奖励向量，$\mathbf{P}^\pi$ 是策略 $\pi$ 下的状态转移矩阵。然而，矩阵求逆的计算复杂度为 $O(|\mathcal{S}|^3)$，对于大规模问题是不可行的。

因此，我们通常采用一种迭代方法，称为**迭代策略评估 (Iterative Policy Evaluation)**。该方法将贝尔曼期望方程转化为一个迭代更新规则。从一个任意的初始值函数 $V_0$ 开始（例如，所有值均为0），我们反复应用以下更新：

$$ 
V_{k+1}(s) \leftarrow \sum_{a \in \mathcal{A}} \pi(a|s) \left( R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V_k(s') \right) 
$$

这个更新过程会持续进行，直到值函数的变化量小于某个预设的阈值 $\theta > 0$，即 $\max_{s \in \mathcal{S}} |V_{k+1}(s) - V_k(s)| < \theta$。

## 3. 算法伪代码 (Pseudocode)

以下是迭代策略评估的完整算法流程。

```
Algorithm: Iterative Policy Evaluation

Input: a policy π to be evaluated
Input: a threshold θ > 0, a small positive number determining the accuracy of estimation

Initialize V(s) = 0, for all s in S+

Loop:
  Δ ← 0
  Loop for each s in S:
    v ← V(s)
    V(s) ← Σ_a π(a|s) [ R(s,a) + γ Σ_s' P(s'|s,a) V(s') ]
    Δ ← max(Δ, |v - V(s)|)
  until Δ < θ

Output V ≈ V^π
```

## 4. 收敛性分析 (Convergence Analysis)

迭代策略评估的收敛性可以由**巴拿赫不动点定理 (Banach Fixed-Point Theorem)** 或收缩映射原理来保证。

我们定义一个**贝尔曼期望算子 (Bellman Expectation Operator)** $B^\pi$：
$$ 
(B^\pi V)(s) = \sum_{a \in \mathcal{A}} \pi(a|s) \left( R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V(s') \right) 
$$
那么，迭代更新规则可以写为 $V_{k+1} = B^\pi V_k$。而 $V^\pi$ 是该算子的不动点，即 $V^\pi = B^\pi V^\pi$。

可以证明，在无穷范数 $||\cdot||_\infty$ 下，贝尔曼期望算子是一个折扣因子为 $\gamma$ 的**收缩映射 (contraction mapping)**。

> **证明**: 对于任意两个值函数 $U$ 和 $V$，以及任意状态 $s$：
> $$ 
> |(B^\pi U)(s) - (B^\pi V)(s)| = \left| \sum_a \pi(a|s) \gamma \sum_{s'} P(s'|s,a) (U(s') - V(s')) \right| 
> $$ 
> $$ 
> \leq \gamma \sum_a \pi(a|s) \sum_{s'} P(s'|s,a) |U(s') - V(s')| 
> $$ 
> $$ 
> \leq \gamma \sum_a \pi(a|s) \sum_{s'} P(s'|s,a) \max_{s''}|U(s'') - V(s'')| 
> $$ 
> $$ 
> = \gamma \, ||U - V||_\infty \, \sum_a \pi(a|s) \sum_{s'} P(s'|s,a) = \gamma \, ||U - V||_\infty 
> $$ 
> 因此，我们有 $||B^\pi U - B^\pi V||_\infty \leq \gamma ||U - V||_\infty$。由于 $\gamma < 1$，$B^\pi$ 是一个收缩映射。

根据巴拿赫不动点定理，对于任何一个收缩映射，从任意初始点开始的迭代序列都将收敛到其唯一的不动点。因此，迭代策略评估保证收敛到唯一的 $V^\pi$。

## 5. 参考文献 (References)

1. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 4.1)
2. Bertsekas, D. P. (2012). *Dynamic programming and optimal control: Volume I*. Athena scientific.
