---

# 2.3.2 SARSA(λ): 结合资格迹的在策略TD控制 (On-Policy TD Control with Eligibility Traces)

---

## 1. 算法概述 (Algorithm Overview)

SARSA(λ) 是对基础 SARSA 算法的一个重要推广，它通过引入**资格迹 (Eligibility Traces)** 机制，将单步的 SARSA(0) 与多步的蒙特卡罗方法联系起来，从而在偏差和方差之间提供了一个灵活的权衡 [1]。这通常能显著提高学习效率。

在单步 TD 学习（如 SARSA(0)）中，一个时间步的 TD 误差只会用来更新刚刚访问过的前一个状态-动作对。然而，直觉上，一个好的或坏的结果（体现在 TD 误差中）应该归功于导致它的整个序列中的多个决策。资格迹就是实现这种“信用回溯分配”的机制。

SARSA(λ) 维护一个与 Q-table 大小相同的资格迹矩阵 $E(s, a)$。这个矩阵记录了每个状态-动作对在近期被访问的“轨迹”。

-   在每个时间步，所有 $(s, a)$ 对的资格迹都会以 $\gamma\lambda$ 的速率衰减。
-   当前访问的 $(S_t, A_t)$ 对的资格迹会被增加（通常是增加1）。

当在时间步 $t$ 产生一个 TD 误差 $\delta_t$ 时，这个误差将不再仅仅用于更新 $Q(S_t, A_t)$，而是会用来更新**所有**状态-动作对，更新的幅度由它们各自的资格迹 $E_t(s, a)$ 决定。
$$ 
Q(s, a) \leftarrow Q(s, a) + \alpha \delta_t E_t(s, a), \quad \forall s \in \mathcal{S}, a \in \mathcal{A} 
$$

参数 $\lambda \in [0, 1]$ 控制了信用回溯的长度：
-   **λ = 0**: 只有当前状态-动作对的资格迹不为零 ($E_t(S_t, A_t)=1$)。此时，算法退化为单步的 **SARSA(0)**。
-   **λ = 1**: 资格迹衰减非常慢，使得当前的 TD 误差几乎会同等地影响历史路径上的所有状态-动作对。其效果近似于**蒙特卡罗方法**，即每个状态-动作对的更新都考虑了直到片段结束的完整回报。

通过选择一个介于 0 和 1 之间的 $\lambda$，SARSA(λ) 能够综合利用不同步长的 TD 回报，通常能找到一个比单步 TD 或纯 MC 方法更优的偏差-方差平衡点，从而加速学习。

## 2. 前向视图与后向视图 (Forward and Backward Views)

理解 SARSA(λ) 有两种等价的视角。

### 2.1 前向视图 (Forward View) 与 λ-回报

前向视图从理论上定义了 SARSA(λ) 的更新目标。它定义了一个新的目标回报，称为 **λ-回报 ($G_t^\lambda$)**。λ-回报是所有不同步长的 n-步回报的加权平均值。

-   1-步回报: $G_{t:t+1} = R_{t+1} + \gamma Q(S_{t+1}, A_{t+1})$
-   2-步回报: $G_{t:t+2} = R_{t+1} + \gamma R_{t+2} + \gamma^2 Q(S_{t+2}, A_{t+2})$
-   ...
-   n-步回报: $G_{t:t+n} = \sum_{k=1}^{n} \gamma^{k-1} R_{t+k} + \gamma^n Q(S_{t+n}, A_{t+n})$

λ-回报是这些 n-步回报的几何加权平均，权重为 $(1-\lambda)\lambda^{n-1}$：
$$ 
G_t^\lambda = (1-\lambda) \sum_{n=1}^{\infty} \lambda^{n-1} G_{t:t+n} 
$$
SARSA(λ) 的更新可以被看作是朝着这个 λ-回报进行的：
$$ 
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha [G_t^\lambda - Q(S_t, A_t)] 
$$
前向视图在理论上很清晰，但在计算上是不可行的，因为它需要等到未来很多步之后才能进行更新。

### 2.2 后向视图 (Backward View) 与资格迹

后向视图提供了一种在线的、增量的实现方式，它在每个时间步都能进行更新，并且与前向视图在数学上是等价的。这正是通过资格迹实现的。

1.  在时间步 $t$，计算单步 TD 误差：
    $$ 
    \delta_t = R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) 
    $$
2.  衰减所有资格迹，并为当前访问的对增加资格：
    $$ 
    E_t(s, a) = \begin{cases} \gamma\lambda E_{t-1}(s, a) + 1 & \text{if } s=S_t, a=A_t \\ \gamma\lambda E_{t-1}(s, a) & \text{otherwise} \end{cases} 
    $$
    这被称为**累积迹 (accumulating traces)**。
3.  使用 TD 误差和资格迹更新所有状态-动作对的 Q 值：
    $$ 
    Q(s, a) \leftarrow Q(s, a) + \alpha \delta_t E_t(s, a), \quad \forall s, a 
    $$

## 3. 算法伪代码 (Pseudocode)

```
Algorithm: Tabular SARSA(λ)

Initialize Q(s, a) arbitrarily, for all s, a
Initialize E(s, a) = 0, for all s, a
Initialize α, γ, λ, ε

Loop for each episode:
  Initialize S, A (chosen using policy from Q, e.g., ε-greedy)
  
  Loop for each step of episode:
    Take action A, observe R, S'
    Choose A' from S' using policy from Q (e.g., ε-greedy)
    
    δ ← R + γ Q(S', A') - Q(S, A)
    E(S, A) ← E(S, A) + 1
    
    For all s ∈ S, a ∈ A:
      Q(s, a) ← Q(s, a) + α δ E(s, a)
      E(s, a) ← γ λ E(s, a)
      
    S ← S'; A ← A'
  until S is terminal
```

## 4. 参考文献 (References)

1.  Sutton, R. S. (1988). Learning to predict by the methods of temporal differences. *Machine learning, 3*(1), 9-44.
2.  Rummery, G. A., & Niranjan, M. (1994). *Online Q-learning using connectionist systems*. (Technical Report, Cambridge University Engineering Department).
3.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapter 12)
