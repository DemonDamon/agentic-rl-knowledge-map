---

# 1.2.4.1 值迭代算法流程 (Value Iteration Algorithm)

---

## 1. 算法描述 (Algorithm Description)

值迭代 (Value Iteration, VI) 是一种通过迭代地更新值函数来寻找最优策略的动态规划算法。与策略迭代不同，值迭代将策略评估和策略改进两个步骤融合在了一次更新中，直接迭代贝尔曼最优性方程。

算法从一个任意的初始值函数 $V_0$（通常为全零向量）开始。在每一次迭代 $k$ 中，它对所有状态 $s \in \mathcal{S}$ 并行地执行一次“贝尔曼更新”，生成新的值函数 $V_{k+1}$。

**核心更新规则:**

这个更新规则直接源于贝尔曼最优性方程：
$$ 
V_{k+1}(s) \leftarrow \max_{a \in \mathcal{A}} \left( R(s, a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s, a) V_k(s') \right) 
$$

这个操作可以被看作是同时完成了两件事：
1.  **一步策略改进**: `max` 操作隐式地找到了在当前值函数 $V_k$ 下的最优动作。
2.  **一步策略评估**: 使用旧的值函数 $V_k$ 来计算这个最优动作的价值，作为新值函数 $V_{k+1}$ 的估计。

**终止条件:**

这个迭代过程会一直持续，直到值函数的变化量足够小，即 $\max_{s \in \mathcal{S}} |V_{k+1}(s) - V_k(s)|$ 小于一个预设的阈值 $\theta$。此时，我们认为值函数已经收敛到了最优值函数 $V^*$ 的一个足够好的近似。

**策略提取 (Policy Extraction):**

当值迭代算法终止后，我们得到的是一个近似的最优值函数 $V^*$。最优策略 $\pi^*$ 并不是在迭代过程中显式维护的，而是可以在最后通过一次完整的策略改进步骤来提取。对于每个状态 $s$，最优策略就是选择那个能够最大化期望回报的动作：
$$ 
\pi^*(s) = \arg\max_{a \in \mathcal{A}} \left( R(s, a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s, a) V^*(s') \right) 
$$

## 2. 伪代码 (Pseudocode)

以下是值迭代算法的完整伪代码。

```
Algorithm: Value Iteration

Input: a threshold θ > 0, a small positive number determining the accuracy of estimation

Initialize V(s) = 0, for all s in S+

Loop:
  Δ ← 0
  Loop for each s in S:
    v ← V(s)
    V(s) ← max_a Σ_s' P(s'|s,a) [ R(s,a,s') + γV(s') ]
    Δ ← max(Δ, |v - V(s)|)
  until Δ < θ

Output a deterministic policy π ≈ π* such that
  π(s) = argmax_a Σ_s' P(s'|s,a) [ R(s,a,s') + γV(s') ]
```

## 3. 异步值迭代 (Asynchronous Value Iteration)

标准的（同步）值迭代算法需要两个数组来存储值函数——一个用于 $V_k$，一个用于 $V_{k+1}$。在每次迭代中，所有状态的更新都依赖于前一次迭代的完整值函数 $V_k$。

**异步值迭代**是一种更灵活、通常也更高效的变体。它不要求以固定的顺序对所有状态进行系统性的扫描，而是可以以任意顺序更新状态的值。更重要的是，它**就地 (in-place)** 进行更新，即计算出的新值会立即覆盖旧值。

例如，如果状态按 $s_1, s_2, \dots, s_{|\mathcal{S}|}$ 的顺序更新，那么在计算 $V(s_i)$ 的新值时，它会使用到部分已经更新过的 $V(s_1), \dots, V(s_{i-1})$ 和部分尚未更新的 $V(s_i), \dots, V(s_{|\mathcal{S}|})$。这种方式使得价值信息能够更快地在状态空间中传播。

只要所有状态都能被持续地访问和更新，异步值迭代同样保证收敛到最优值函数。这种思想在分布式计算和后续的强化学习算法中具有重要影响。

## 4. 参考文献 (References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 4.4)
2.  Bellman, R. (1957). A Markovian decision process. *Journal of Mathematics and Mechanics*, 673-684.
3.  Bertsekas, D. P. (1982). Distributed dynamic programming. *IEEE Transactions on Automatic Control, 27*(3), 610-616.
