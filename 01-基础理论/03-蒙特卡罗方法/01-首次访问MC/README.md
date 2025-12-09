---

# 1.3.1 首次访问蒙特卡罗预测 (First-Visit Monte Carlo Prediction)

---

## 1. 算法概述 (Algorithm Overview)

首次访问蒙特卡罗 (First-Visit MC) 预测是一种使用蒙特卡罗方法来估计给定策略 $\pi$ 的状态值函数 $V^\pi(s)$ 的经典算法 [1]。其核心思想非常直观：通过模拟大量的经验片段 (episodes)，并将一个状态的值估计为其在所有片段中**首次**出现后所获得的累积回报的平均值。

**关键概念:**

1.  **经验片段 (Episode)**: 一个从起始状态开始，遵循策略 $\pi$ 直至终止状态的完整序列：$S_0, A_0, R_1, S_1, A_1, R_2, \dots, S_T, A_T, R_{T+1}$。
2.  **回报 (Return)**: 从时间步 $t$ 开始的累积折扣回报 $G_t = \sum_{k=0}^{T-t-1} \gamma^k R_{t+k+1}$。
3.  **首次访问 (First Visit)**: 在一个经验片段中，状态 $s$ 可能被访问多次。首次访问 MC 方法只关心该片段中第一次访问状态 $s$ 的那个时间步 $t$。从该时间步 $t$ 开始的回报 $G_t$ 将被用于更新 $V(s)$。

根据大数定律，只要我们收集足够多的、从状态 $s$ 首次访问开始的回报，它们的均值就会收敛到真实的状态值函数 $V^\pi(s)$。

$$ 
V^\pi(s) = \mathbb{E}_\pi[G_t | S_t = s] \approx \frac{1}{N(s)} \sum_{i=1}^{N(s)} G_t^{(i)}(s) 
$$
其中 $N(s)$ 是状态 $s$ 被首次访问的总次数，$G_t^{(i)}(s)$ 是第 $i$ 个片段中状态 $s$ 首次出现后的回报。

## 2. 增量式实现 (Incremental Implementation)

在实践中，我们通常不需要存储所有的回报然后一次性计算平均值。我们可以使用一种增量式的均值更新方法。对于一个序列 $x_1, x_2, \dots, x_n$，其均值 $\mu_n$ 可以被增量式地计算：
$$ 
\mu_n = \frac{1}{n} \sum_{i=1}^n x_i = \mu_{n-1} + \frac{1}{n}(x_n - \mu_{n-1}) 
$$

将这个思想应用到 MC 预测中，我们可以得到如下的更新规则。在处理完一个包含状态 $s$ 的新片段后，我们得到一个新的回报 $G$。令 $N(s)$ 是之前 $s$ 被访问的次数，更新规则为：

1.  $N(s) \leftarrow N(s) + 1$
2.  $V(s) \leftarrow V(s) + \frac{1}{N(s)}(G - V(s))$

这个更新规则可以被写成一个更通用的形式，其中 $\alpha = 1/N(s)$ 可以被看作是一个随时间递减的学习率：
$$ 
V(s) \leftarrow V(s) + \alpha (G - V(s)) 
$$

## 3. 算法伪代码 (Pseudocode)

```
Algorithm: First-Visit MC Prediction, for estimating V ≈ v_π

Input: a policy π to be evaluated

Initialize:
  V(s) ∈ ℝ, arbitrarily, for all s ∈ S
  Returns(s) ← an empty list, for all s ∈ S

Loop forever (for each episode):
  Generate an episode following π: S₀, A₀, R₁, S₁, A₁, R₂, ..., S_{T-1}, A_{T-1}, R_T
  G ← 0
  Loop for each step of episode, t = T-1, T-2, ..., 0:
    G ← γG + R_{t+1}
    If S_t does not appear in S₀, S₁, ..., S_{t-1}:
      Append G to Returns(S_t)
      V(S_t) ← average(Returns(S_t))
```

**增量式实现的伪代码:**

```
Algorithm: Incremental First-Visit MC Prediction

Initialize:
  V(s) ∈ ℝ, arbitrarily, for all s ∈ S
  N(s) = 0, for all s ∈ S

Loop forever (for each episode):
  Generate an episode following π: S₀, A₀, R₁, ..., R_T
  G ← 0
  visited_states ← an empty set
  Loop for each step of episode, t = T-1, T-2, ..., 0:
    G ← γG + R_{t+1}
    If S_t ∉ visited_states:
      visited_states.add(S_t)
      N(S_t) ← N(S_t) + 1
      V(S_t) ← V(S_t) + (1/N(S_t)) * (G - V(S_t))
```

## 4. 特性分析 (Analysis)

-   **无偏性 (Unbiased)**: 首次访问 MC 估计是无偏的。每个回报 $G_t$ 都是对 $V^\pi(S_t)$ 的一个独立同分布的无偏采样。因此，它们的均值也收敛到真实值。
-   **高方差 (High Variance)**: 由于回报 $G_t$ 包含了从 $t$ 到片段结束的所有随机奖励，其方差通常很大。这会导致学习过程收敛缓慢，尤其是在随机性强的环境中。
-   **仅适用于片段式任务 (Episodic Tasks)**: MC 方法需要等待一个完整的片段结束后才能进行更新，因此它不能直接应用于连续的、没有终止状态的任务。
-   **不自举 (No Bootstrapping)**: MC 方法的更新不依赖于其他状态的值的估计，它完全依赖于实际观测到的完整回报。这与 TD 学习形成鲜明对比。

## 5. 参考文献 (References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 5.1)
2.  Singh, S. P., & Sutton, R. S. (1996). Reinforcement learning with replacing eligibility traces. *Machine learning, 22*(1), 123-158.
