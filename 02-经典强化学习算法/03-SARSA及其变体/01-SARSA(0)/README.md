---

# 2.3.1 SARSA(0): 单步在策略TD控制 (Single-Step On-Policy TD Control)

---

## 1. 算法概述 (Algorithm Overview)

SARSA(0) 是最基础的 SARSA 算法，也是一种经典的在策略 (on-policy) 时间差分 (TD) 控制方法 [1]。这里的 `(0)` 表示它是一种单步 (one-step) 更新算法，即其 TD 目标只向前看一步，不使用资格迹 (eligibility traces)。

SARSA 的名字来源于其更新规则所依赖的经验五元组：**State, Action, Reward, State', Action'**，即 $(S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1})$。

与 Q-Learning 不同，SARSA 的目标是学习当前正在执行的行为策略 $\pi$ 的动作值函数 $Q^\pi(s, a)$，而不是直接学习最优值函数 $Q^*(s, a)$。这意味着 SARSA 评估的策略和用于生成行为的策略是同一个，通常是一个 ε-贪婪策略。

**核心更新规则:**

在时间步 $t$，智能体处于状态 $S_t$ 并根据其策略 $\pi$ (例如，ε-贪婪于当前的 Q 值) 选择了动作 $A_t$。在执行 $A_t$ 后，它观察到奖励 $R_{t+1}$ 和新状态 $S_{t+1}$。接着，它**再次根据策略 $\pi$** 从新状态 $S_{t+1}$ 中选择下一个动作 $A_{t+1}$。这个未来的动作 $A_{t+1}$ 被用来构建 TD 目标，并更新 $Q(S_t, A_t)$：
$$ 
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[ \underbrace{R_{t+1} + \gamma Q(S_{t+1}, A_{t+1})}_{\text{TD Target}} - Q(S_t, A_t) \right] 
$$

**与 Q-Learning 的关键区别:**

| 特性 | SARSA(0) | Q-Learning |
| :--- | :--- | :--- |
| **TD 目标** | $R_{t+1} + \gamma Q(S_{t+1}, A_{t+1})$ | $R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a)$ |
| **策略类型** | **在策略 (On-policy)** | **离策略 (Off-policy)** |
| **学习目标** | 学习当前行为策略 (如 ε-贪婪) 的 Q 值 | 学习最优策略的 Q 值 |

因为 SARSA 的更新依赖于策略在下一步**实际会采取**的动作 $A_{t+1}$，所以它把探索（例如，ε-贪婪策略中的随机动作）的成本也计算在内。这使得 SARSA 在学习过程中表现得比 Q-Learning 更为“保守”。

**示例：悬崖行走 (Cliff Walking)**

在一个经典的悬崖行走环境中，智能体需要从起点走到终点，但路径旁边是悬崖，掉下去会受到巨大的负奖励。
-   **Q-Learning** 会学习到贴着悬崖边缘走的最短路径。因为它只关心最优价值，不关心探索过程中可能发生的失误。
-   **SARSA** 会学习到一条离悬崖有一定距离的安全路径。因为它知道自己的 ε-贪婪策略在悬崖边有 $\epsilon$ 的概率会随机选择一个动作掉下悬崖，这个潜在的巨大负奖励会被计入 Q 值中，从而使它避开危险区域。

## 2. 算法伪代码 (Pseudocode)

```
Algorithm: SARSA(0) for estimating Q ≈ q_π

Initialize Q(s, a) for all s ∈ S, a ∈ A(s), arbitrarily, and Q(terminal-state, ·) = 0
Initialize learning rate α ∈ (0, 1] and exploration rate ε > 0

Loop for each episode:
  Initialize S
  Choose A from S using policy derived from Q (e.g., ε-greedy)
  
  Loop for each step of episode:
    Take action A, observe R, S'
    Choose A' from S' using policy derived from Q (e.g., ε-greedy)
    
    Q(S, A) ← Q(S, A) + α [R + γ Q(S', A') - Q(S, A)]
    
    S ← S'; A ← A'
  until S is terminal
```

## 3. 收敛性分析 (Convergence Analysis)

SARSA 的收敛性条件与 Q-Learning 类似，但有一个关键区别。SARSA 要收敛到最优策略，其行为策略必须随着学习的进行而逐渐变得更加贪婪。

**收敛条件:**

1.  **无限访问**: 所有状态-动作对被无限次访问。
2.  **学习率条件**: 学习率 $\alpha_t$ 满足 Robbins-Monro 条件。
3.  **策略收敛到贪婪**: 行为策略 $\pi_t$ 在极限情况下收敛到一个贪婪策略，即 $\epsilon_t \to 0$。例如，可以设置 $\epsilon_t = 1/t$。

如果满足这些条件，SARSA(0) 保证会收敛到最优动作值函数 $Q^*$。

这个过程本质上是在**广义策略迭代 (Generalized Policy Iteration, GPI)** 的框架下运作的。Q 函数不断地逼近当前策略 $\pi_t$ 的真实值函数，而策略 $\pi_t$ 又随着 Q 函数的改进而变得更加贪婪。只要这两个过程保持一个合适的步调，算法最终就能找到最优解。

## 4. 参考文献 (References)

1.  Rummery, G. A., & Niranjan, M. (1994). *Online Q-learning using connectionist systems*. (Technical Report, Cambridge University Engineering Department).
2.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 6.4)
3.  Singh, S., Jaakkola, T., Littman, M. L., & Szepesvári, C. (2000). Convergence results for single-step on-policy reinforcement-learning algorithms. *Machine learning, 38*(3), 287-308.
