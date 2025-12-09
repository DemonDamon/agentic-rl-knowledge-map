---

# 1.3.2 每次访问蒙特卡罗预测 (Every-Visit Monte Carlo Prediction)

---

## 1. 算法概述 (Algorithm Overview)

每次访问蒙特卡罗 (Every-Visit MC) 预测是 MC 预测的另一种形式，它与首次访问 MC 的主要区别在于如何处理在一个经验片段中多次出现的状态 [1]。

-   **首次访问 MC**: 只考虑片段中状态 $s$ **第一次**出现后的回报。
-   **每次访问 MC**: 考虑片段中状态 $s$ **每一次**出现后的回报。

具体来说，如果在一个片段中，状态 $s$ 在时间步 $t_1, t_2, \dots, t_k$ 被访问了 $k$ 次，那么每次访问 MC 会将对应的回报 $G_{t_1}, G_{t_2}, \dots, G_{t_k}$ 全部纳入状态 $s$ 的回报列表中，并计算它们的平均值作为 $V(s)$ 的估计。

$$ 
V^\pi(s) \approx \frac{\sum_{i=1}^{N_{episodes}} \sum_{t=0}^{T_i-1} \mathbb{I}(S_{i,t}=s) G_{i,t}}{\sum_{i=1}^{N_{episodes}} \sum_{t=0}^{T_i-1} \mathbb{I}(S_{i,t}=s)} 
$$
其中 $\mathbb{I}(\cdot)$ 是指示函数，$i$ 是片段索引，$t$ 是片段内的时间步。

## 2. 算法伪代码 (Pseudocode)

每次访问 MC 的算法与首次访问 MC 非常相似，唯一的区别在于我们不再需要检查一个状态是否是首次被访问。

```
Algorithm: Every-Visit MC Prediction, for estimating V ≈ v_π

Input: a policy π to be evaluated

Initialize:
  V(s) ∈ ℝ, arbitrarily, for all s ∈ S
  Returns(s) ← an empty list, for all s ∈ S

Loop forever (for each episode):
  Generate an episode following π: S₀, A₀, R₁, S₁, A₁, R₂, ..., S_{T-1}, A_{T-1}, R_T
  G ← 0
  Loop for each step of episode, t = T-1, T-2, ..., 0:
    G ← γG + R_{t+1}
    Append G to Returns(S_t)
    V(S_t) ← average(Returns(S_t))
```

**增量式实现的伪代码:**

```
Algorithm: Incremental Every-Visit MC Prediction

Initialize:
  V(s) ∈ ℝ, arbitrarily, for all s ∈ S
  N(s) = 0, for all s ∈ S

Loop forever (for each episode):
  Generate an episode following π: S₀, A₀, R₁, ..., R_T
  G ← 0
  Loop for each step of episode, t = T-1, T-2, ..., 0:
    G ← γG + R_{t+1}
    N(S_t) ← N(S_t) + 1
    V(S_t) ← V(S_t) + (1/N(S_t)) * (G - V(S_t))
```

## 3. 与首次访问 MC 的比较 (Comparison with First-Visit MC)

| 特性 | 首次访问 MC (First-Visit MC) | 每次访问 MC (Every-Visit MC) |
| :--- | :--- | :--- |
| **定义** | 估计 $V^\pi(s)$ 为从 $s$ **首次**访问开始的回报的均值。 | 估计 $V^\pi(s)$ 为从 $s$ **所有**访问开始的回报的均值。 |
| **实现** | 需要一个额外的机制（如一个 `visited` 集合）来跟踪每个片段中哪些状态已被访问。 | 实现更简单，无需跟踪访问历史。 |
| **偏差** | **无偏 (Unbiased)**。每个样本都是对 $V^\pi(s)$ 的一个独立同分布的无偏估计。 | **有偏 (Biased)**，但偏差会随着样本数量的增加而消失。 |
| **收敛性** | 均方误差收敛到 0。 | 均方误差收敛到 0。两种方法都会收敛到真实值 $V^\pi(s)$。 |
| **方差** | 通常方差略高，因为它使用的样本数量可能更少。 | 通常方差略低，因为它利用了更多的样本。 |
| **理论分析** | 更容易进行理论分析，因为样本是独立的。 | 理论分析更复杂，因为来自同一片段的样本是相关的。 |

**偏差的来源:**

每次访问 MC 的偏差是一个微妙的点。虽然它最终会收敛到无偏的估计，但在有限样本的情况下，它是有偏的。这是因为在一个片段中，对一个状态的多次访问的回报是相关的（例如，在 $t_2$ 访问的回报是 $t_1$ 访问回报的一部分）。这种相关性使得简单的平均操作在统计上不是最优的。然而，在实践中，这种偏差通常很小，并且随着数据量的增加而迅速减小。

**实践中的选择:**

在实践中，首次访问和每次访问 MC 方法的性能差异通常不大。由于每次访问 MC 实现起来更简单，并且在某些情况下可能因为使用了更多数据而学习得稍快一些，因此它也经常被使用。

Sutton 和 Barto 在其著作中指出，尽管存在这些差异，但两种方法的收敛性都得到了保证，并且在应用中没有明确的偏好 [1]。

## 4. 参考文献 (References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 5.1)
2.  Singh, S., & Sutton, R. S. (1996). Reinforcement learning with replacing eligibility traces. *Machine Learning, 22*(1-3), 123-158. (This paper discusses the bias/variance trade-offs of different MC methods in the context of eligibility traces).
