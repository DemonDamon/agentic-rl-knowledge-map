---

# 1.1.4 贝尔曼方程 (The Bellman Equation)

---

## 1. 概述 (Overview)

贝尔曼方程，以其提出者、美国数学家理查德·贝尔曼 (Richard Bellman) 的名字命名，是强化学习乃至整个动态规划领域中最为核心的数学关系式 [1]。它以一种优美的递归形式，将一个状态或状态-动作对的价值与其后继状态的价值关联起来，从而为求解最优策略提供了理论基础和计算途径。

贝尔曼方程的本质思想是“最优性原理” (Principle of Optimality) 的体现：一个最优策略的子策略也必须是最优的。换言之，无论过去的状态和决策如何，对于由它们所导致的当前状态，余下的决策必须构成一个最优策略。贝尔曼方程将这一原理形式化，建立了当前价值与未来价值之间的联系。

本节将深入推导和分析贝尔曼方程的两种主要形式：

1.  **贝尔曼期望方程 (Bellman Expectation Equation):** 这种形式描述了在给定一个特定策略 $\pi$ 的情况下，一个状态或状态-动作对的价值。它是一个线性方程组，原则上可以直接求解，从而完成对策略 $\pi$ 的**评估 (evaluation)**。我们将分别推导状态值函数 $V^\pi(s)$ 和动作值函数 $Q^\pi(s, a)$ 的贝尔曼期望方程。

2.  **贝尔曼最优性方程 (Bellman Optimality Equation):** 这种形式描述了最优策略下的价值函数，即在所有可能的策略中能达到的最大价值。与期望方程不同，最优性方程由于包含了最大化操作 `max`，因此是非线性的。它无法直接求解，但构成了值迭代 (Value Iteration) 等动态规划算法的基础。我们将推导最优状态值函数 $V^*(s)$ 和最优动作值函数 $Q^*(s, a)$ 的贝尔曼最优性方程。

此外，我们还将从数学分析的角度探讨贝尔曼算子 (Bellman Operator) 的性质，特别是其作为**收缩映射 (Contraction Mapping)** 的特性。基于巴拿赫不动点定理 (Banach Fixed-Point Theorem)，这一性质保证了贝尔曼最优性方程存在唯一的不动点（即最优值函数），并且像值迭代这样的迭代算法能够收敛到这个不动点。这是保证许多强化学习算法收敛性的关键数学工具。

## 2. 目录 (Table of Contents)

- [**01-状态值函数**](./01-状态值函数/README.md): 贝尔曼期望方程对于 $V^\pi(s)$ 的推导。
- [**02-动作值函数**](./02-动作值函数/README.md): 贝尔曼期望方程对于 $Q^\pi(s, a)$ 的推导。
- [**03-最优性方程**](./03-最优性方程/README.md): 描述最优值函数的非线性贝尔曼方程。
- [**04-收缩映射定理**](./04-收缩映射定理/README.md): 保证值迭代算法收敛的数学原理。

## 3. 核心参考文献 (Core References)

1.  Bellman, R. (1957). *Dynamic programming*. Princeton university press.
2.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 3.5, 3.6, 3.7)
3.  Bertsekas, D. P. (2012). *Dynamic programming and optimal control: Volume I*. Athena scientific.
