---

# 2.2 Q-Learning及其变体 (Q-Learning and Its Variants)

---

## 1. 概述 (Overview)

Q-Learning 是由 Chris Watkins 在其1989年的博士论文中提出的，是强化学习领域最具里程碑意义的算法之一 [1]。它是一种无模型 (model-free)、离策略 (off-policy) 的时间差分 (TD) 控制算法，其目标是直接学习最优动作值函数 $Q^*(s, a)$，而无需知道环境的动态模型。

Q-Learning 的核心思想是使用一个简单的迭代更新规则来逼近最优 Q 函数。这个更新规则基于贝尔曼最优性方程：
$$ 
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[ R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a) - Q(S_t, A_t) \right] 
$$
其中，$\alpha$ 是学习率，$\gamma$ 是折扣因子。关键在于更新目标 $R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a)$，它使用了在下一状态 $S_{t+1}$ 上所有可能动作中具有最大 Q 值的动作来估计期望回报，而**不**是实际执行的动作。正是这个 `max` 操作，使得 Q-Learning 能够从一个探索性的行为策略（如 ε-贪婪策略）中学习到一个最优的贪婪策略，从而实现了离策略学习。

Q-Learning 的简洁性和强大的理论保证（在一定条件下保证收敛到最优值函数）使其成为应用最广泛的强化学习算法之一。然而，它也存在一些问题，最著名的是**最大化偏差 (Maximization Bias)**。由于在更新规则中使用了 `max` 操作，当 Q 值估计存在噪声时，Q-Learning 倾向于系统性地高估动作的价值，这可能导致学习到次优策略。

本节将深入探讨 Q-Learning 的原理、收敛性证明，以及为解决其核心问题而提出的重要变体。

## 2. 目录 (Table of Contents)

- [**01-表格Q-Learning**](./01-表格Q-Learning/README.md): 原始的、适用于离散状态和动作空间的 Q-Learning 算法。
- [**02-Double Q-Learning**](./02-Double-Q-Learning/README.md): 一种旨在解决 Q-Learning 最大化偏差问题的经典变体。
- [**03-Speedy Q-Learning**](./03-Speedy-Q-Learning/README.md): 一种旨在加速 Q-Learning 收敛速度的变体。

## 3. 核心参考文献 (Core References)

1.  Watkins, C. J. C. H. (1989). *Learning from delayed rewards*. (Doctoral dissertation, University of Cambridge).
2.  Watkins, C. J., & Dayan, P. (1992). Q-learning. *Machine learning, 8*(3-4), 279-292.
3.  Hasselt, H. V. (2010). Double Q-learning. In *Advances in Neural Information Processing Systems, 23*.
4.  Azar, M. G., Munos, R., Ghavamzadeh, M., & Kappen, H. J. (2011). Speedy Q-learning. In *Advances in Neural Information Processing Systems, 24*.
