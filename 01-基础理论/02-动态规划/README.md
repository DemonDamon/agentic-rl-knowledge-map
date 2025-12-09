---

# 1.2 动态规划 (Dynamic Programming)

---

## 1. 概述 (Overview)

动态规划 (Dynamic Programming, DP) 是一类用于求解复杂问题的优化方法，它将问题分解为更小的子问题，并存储子问题的解以避免重复计算。在强化学习的背景下，当马尔可夫决策过程 (MDP) 的完整模型已知时——即状态转移概率 $P(s'|s, a)$ 和奖励函数 $R(s, a, s')$ 均可知——DP 提供了一套具有坚实理论基础的算法来计算最优策略 [1]。

DP 算法的核心思想是利用值函数来组织和构建搜索最优策略的过程。通过迭代应用贝尔曼方程，DP 方法可以逐步逼近最优值函数和最优策略。尽管“完全已知的模型”这一假设在许多现实应用中难以满足，但 DP 的理论重要性无可替代。它不仅为后续的无模型 (model-free) 学习算法提供了理论目标和分析工具，其核心思想——策略评估和策略改进的交替进行——也构成了广义策略迭代 (Generalized Policy Iteration, GPI) 模式，这是几乎所有强化学习算法的通用框架 [2]。

本节将详细探讨用于求解 MDP 的两种主要 DP 算法：

1.  **策略评估 (Policy Evaluation):** 给定一个固定的策略 $\pi$，计算其对应的状态值函数 $V^\pi(s)$。这是一个迭代过程，通过反复应用贝尔曼期望方程，直至值函数收敛。

2.  **策略改进 (Policy Improvement):** 在获得当前策略的值函数后，通过贪婪地选择能够带来更高期望回报的动作来改进策略。**策略改进定理 (Policy Improvement Theorem)** 保证了这一过程将产生一个不差于原策略的新策略。

3.  **策略迭代 (Policy Iteration):** 将策略评估和策略改进结合起来，形成一个完整的迭代循环。从一个任意策略开始，交替进行策略评估和策略改进，直至策略不再发生变化，此时即收敛到最优策略。我们将提供其收敛性的严格证明。

4.  **值迭代 (Value Iteration):** 策略迭代的一个变体，它将策略评估的迭代过程缩减为仅一步，然后立即与策略改进相结合。值迭代通过直接应用贝尔曼最优性方程进行迭代，通常具有更快的收敛速度。我们将从巴拿赫不动点定理 (Banach Fixed-Point Theorem) 的角度证明其收敛性，并将其与策略迭代进行比较。

通过本节的学习，读者将掌握在已知模型下求解最优策略的根本方法，并深刻理解支撑现代强化学习算法的广义策略迭代思想。

## 2. 目录 (Table of Contents)

- [**01-策略评估**](./01-策略评估/README.md): 计算给定策略的值函数。
- [**02-策略改进**](./02-策略改进/README.md): 根据值函数提升当前策略。
- [**03-策略迭代**](./03-策略迭代/README.md): 交替评估和改进直至收敛到最优策略。
- [**04-值迭代**](./04-值迭代/README.md): 一种更高效的 DP 算法，直接求解最优值函数。

## 3. 核心参考文献 (Core References)

1.  Bellman, R. (1957). *Dynamic programming*. Princeton university press.
2.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapter 4)
3.  Bertsekas, D. P. (2012). *Dynamic programming and optimal control: Volume I*. Athena scientific.
