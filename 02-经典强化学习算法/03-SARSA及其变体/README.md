---

# 2.3 SARSA及其变体 (SARSA and Its Variants)

---

## 1. 概述 (Overview)

SARSA 是一种与 Q-Learning 并列的经典时间差分 (TD) 控制算法 [1]。与 Q-Learning 不同，SARSA 是一种**在策略 (on-policy)** 算法。这意味着它直接评估并改进当前正在执行的行为策略，而不是像 Q-Learning 那样试图学习一个独立于行为的最优策略。

算法的名字 SARSA 来源于其更新规则所依赖的经验五元组：**$(S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1})$**。在状态 $S_t$ 执行动作 $A_t$ 后，环境转移到 $S_{t+1}$ 并给出奖励 $R_{t+1}$，然后智能体在 $S_{t+1}$ 状态下，根据其当前策略 $\pi$（通常是 ε-贪婪策略）选择下一个动作 $A_{t+1}$。这个未来的动作 $A_{t+1}$ 被用来构建 TD 目标，从而更新当前的动作值函数 $Q(S_t, A_t)$。

SARSA 的更新规则如下：
$$ 
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[ R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) \right] 
$$

与 Q-Learning 的关键区别在于 TD 目标：SARSA 使用的是 $R_{t+1} + \gamma Q(S_{t+1}, A_{t+1})$，而 Q-Learning 使用的是 $R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a)$。这意味着 SARSA 的更新依赖于策略在下一步实际会选择的动作，因此它是在评估和改进当前的行为策略。

这种在策略的特性使得 SARSA 的行为通常比 Q-Learning 更“保守”或“安全”。例如，在一个悬崖边行走的问题中，Q-Learning 可能会学习到贴着悬崖边走的最优路径，因为它只关心最大化 Q 值，而不关心探索过程中可能掉下悬崖的风险。而 SARSA 在学习过程中会考虑到 ε-贪婪策略所带来的随机性，即它知道在悬崖边有一定概率会因为随机探索而失足，因此它会学习到一条离悬崖稍远一些、虽然不是绝对最优但更安全的路径。

本节将深入探讨 SARSA 算法的原理、收敛性，以及其与资格迹结合的重要变体。

## 2. 目录 (Table of Contents)

- [**01-SARSA(0)**](./01-SARSA(0)/README.md): 基础的单步 SARSA 算法。
- [**02-SARSA(λ)**](./02-SARSA(λ)/README.md): 将 SARSA 与资格迹结合，实现多步 TD 学习。
- [**03-True Online SARSA**](./03-True-Online-SARSA/README.md): 一种更高效、更理论完备的在线 SARSA(λ) 实现。

## 3. 核心参考文献 (Core References)

1.  Rummery, G. A., & Niranjan, M. (1994). *Online Q-learning using connectionist systems*. University of Cambridge, Department of Engineering. (Note: This paper introduced the SARSA algorithm, though it was named "Modified Q-learning" at the time).
2.  Sutton, R. S. (1996). Generalization in reinforcement learning: Successful examples using sparse coarse coding. In *Advances in Neural Information Processing Systems, 8*.
3.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 6.4)
4.  van Seijen, H., & Sutton, R. S. (2014). True online TD(λ). In *International Conference on Machine Learning*.
