---

# 2.2.1 表格Q-Learning (Tabular Q-Learning)

---

## 1. 算法概述 (Algorithm Overview)

表格 Q-Learning 是由 Watkins 在1989年提出的原始 Q-Learning 算法 [1]。它适用于状态和动作空间均为离散且有限的场景，其核心思想是维护一张 Q-table（一个二维数组或哈希表），用于存储每一个状态-动作对 $(s, a)$ 的价值估计 $Q(s, a)$。

Q-Learning 是一种无模型 (model-free)、离策略 (off-policy) 的时间差分 (TD) 控制算法。它的目标是直接学习最优动作值函数 $Q^*(s, a)$，而无需知道环境的状态转移概率 $P(s'|s,a)$ 和奖励函数 $R(s,a)$。

**核心更新规则:**

智能体与环境交互，在状态 $S_t$ 执行动作 $A_t$，观察到奖励 $R_{t+1}$ 和新状态 $S_{t+1}$。然后，Q-table 中的对应条目将根据以下规则进行更新：
$$ 
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[ \underbrace{R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a)}_{\text{TD Target}} - Q(S_t, A_t) \right] 
$$

-   **$\alpha \in (0, 1]$**: 学习率 (Learning Rate)，控制新信息覆盖旧信息的程度。
-   **$\gamma \in [0, 1)$**: 折扣因子 (Discount Factor)，权衡即时奖励和未来奖励。
-   **$R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a)$**: 这是 **TD 目标 (TD Target)**。它使用当前对下一状态 $S_{t+1}$ 的最优价值的估计（通过 $\max_a$ 操作）来构建一个对当前状态-动作对价值的更准确的估计。
-   **$\delta_t = R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a) - Q(S_t, A_t)$**: 这是 **TD 误差 (TD Error)**，表示当前估计与更准确的目标之间的差距。学习过程就是不断减小这个误差。

**离策略 (Off-Policy) 特性:**

Q-Learning 的关键在于其离策略性质。用于生成行为的策略（行为策略，通常是 ε-贪婪策略）和用于评估和改进的策略（目标策略，即贪婪策略）是不同的。
-   **行为策略 (Behavior Policy)**: 为了保证充分的探索，智能体通常采用 ε-贪婪策略来选择动作。即以 $1-\epsilon$ 的概率选择当前认为最优的动作（$\arg\max_a Q(S_t, a)$），以 $\epsilon$ 的概率随机选择一个动作。
-   **目标策略 (Target Policy)**: 在更新规则中，TD 目标总是使用贪婪策略（通过 $\max_a$ 操作）来估计下一状态的价值，无论行为策略实际选择了哪个动作。这使得 Q-Learning 能够从探索性的、次优的行为中学习到最优的策略。

## 2. 算法伪代码 (Pseudocode)

```
Algorithm: Tabular Q-Learning

Initialize Q(s, a) for all s ∈ S, a ∈ A(s), arbitrarily, and Q(terminal-state, ·) = 0
Initialize learning rate α ∈ (0, 1] and exploration rate ε > 0

Loop for each episode:
  Initialize S
  Loop for each step of episode:
    Choose A from S using policy derived from Q (e.g., ε-greedy)
    Take action A, observe R, S'
    
    Q(S, A) ← Q(S, A) + α [R + γ max_a Q(S', a) - Q(S, A)]
    
    S ← S'
  until S is terminal
```

## 3. 收敛性证明 (Proof of Convergence)

Q-Learning 的收敛性证明是一个经典结果，它表明在一定条件下，Q-table 中的值会以概率 1 收敛到最优动作值函数 $Q^*$ [2]。

**核心条件:**

1.  **无限访问**: 每一个状态-动作对 $(s, a)$ 都被无限次地访问。
2.  **学习率条件 (Robbins-Monro aequence)**: 学习率 $\alpha_t$ 必须满足随机近似理论的条件：
    $$ 
    \sum_{t=1}^{\infty} \alpha_t(s,a) = \infty \quad \text{and} \quad \sum_{t=1}^{\infty} \alpha_t^2(s,a) < \infty 
    $$
    第一个条件保证了学习步长足够大，可以克服任意的初始值。第二个条件保证了学习最终会收敛，不会持续振荡。

**证明思路:**

证明的核心在于将 Q-Learning 的更新过程看作是一个随机近似 (Stochastic Approximation) 过程，它试图求解贝尔曼最优性方程 $Q^* = B^* Q^*$。更新规则可以重写为：
$$ 
Q_{t+1}(s,a) = (1 - \alpha_t) Q_t(s,a) + \alpha_t \left( R_{t+1} + \gamma \max_{a'} Q_t(S_{t+1}, a') \right) 
$$

可以证明，贝尔曼最优性算子 $B^*$ 是一个在无穷范数下的收缩映射。Q-Learning 的更新可以被看作是对这个算子的一个带噪声的、异步的迭代。Watkins 和 Dayan 的工作表明，只要满足上述条件，这个随机近似过程最终会收敛到算子的唯一不动点，即 $Q^*$。

## 4. 复杂度分析 (Complexity Analysis)

-   **空间复杂度**: 算法需要存储整个 Q-table，因此空间复杂度为 $O(|\mathcal{S}| |\mathcal{A}|)$。这是表格方法的主要局限性，当状态或动作空间巨大时，这是不可行的。
-   **时间复杂度 (每步更新)**: 在每一步交互后，算法需要执行一次更新。更新操作包括：
    1.  在下一状态 $S'$ 上找到具有最大 Q 值的动作，需要 $O(|\mathcal{A}|)$ 的时间。
    2.  执行一次简单的算术运算来更新 $Q(S, A)$，时间为 $O(1)$。
    因此，每一步更新的时间复杂度为 $O(|\mathcal{A}|)$。

## 5. 参考文献 (References)

1.  Watkins, C. J. C. H. (1989). *Learning from delayed rewards*. (Doctoral dissertation, University of Cambridge).
2.  Watkins, C. J., & Dayan, P. (1992). Q-learning. *Machine learning, 8*(3-4), 279-292.
3.  Tsitsiklis, J. N. (1994). Asynchronous stochastic approximation and Q-learning. *Machine learning, 16*(3), 185-202.
