---

# 2.1 值迭代与策略迭代 (Value and Policy Iteration)

---

## 1. 概述 (Overview)

值迭代和策略迭代是动态规划 (DP) 中用于求解已知模型 MDP 的两种奠基性算法。虽然它们的核心思想在第一章已经介绍，但作为经典强化学习算法的起点，本节将从算法实现和扩展的角度对它们进行更深入的探讨。

在经典算法的范畴里，我们关注的是这些算法的变体以及它们如何启发了后续更复杂的算法，特别是在处理大规模问题和分布式计算方面。

1.  **同步与异步 DP (Synchronous and Asynchronous DP):**
    -   **同步值迭代 (Synchronous Value Iteration):** 这是标准的值迭代算法，在每次迭代中，需要使用一个“旧”的值函数向量 $V_k$ 来计算一个全新的值函数向量 $V_{k+1}$。这需要两份存储空间，并且更新是并行的。
    -   **异步值迭代 (Asynchronous Value Iteration):** 为了提高效率和灵活性，异步 DP 算法允许以任意顺序、就地 (in-place) 更新状态的值。例如，一个状态的值被更新后，会立即用于计算其他状态的值。这种方法不仅节省了存储空间，而且通常能更快地传播价值信息，加速收敛。异步 DP 的思想是后续许多强化学习算法（如 A3C 中的异步更新）的重要灵感来源。

2.  **修正策略迭代 (Modified Policy Iteration):**
    -   标准的策略迭代算法要求在每次策略改进之前，必须进行一次**完全的**策略评估，即迭代直至值函数完全收敛。这在计算上是非常昂贵的。
    -   修正策略迭代（或称为“不精确策略迭代”）放宽了这一要求。它允许在策略评估步骤只进行有限次数的迭代（甚至只有一次，这就变成了值迭代），然后就进行策略改进。这种方法在策略迭代的稳定收敛和值迭代的快速迭代之间取得了平衡，提供了更灵活的计算方案。

本节将深入分析这些经典 DP 算法的变体，探讨它们的收敛性质、计算效率以及它们如何为现代强化学习算法（特别是分布式和大规模算法）铺平了道路。

## 2. 目录 (Table of Contents)

- [**01-同步值迭代**](./01-同步值迭代/README.md): 标准的并行更新值迭代算法。
- [**02-异步值迭代**](./02-异步值迭代/README.md): 更高效的、就地更新的值迭代变体。
- [**03-修正策略迭代**](./03-修正策略迭代/README.md): 在策略迭代和值迭代之间取得平衡的灵活算法。

## 3. 核心参考文献 (Core References)

1.  Bertsekas, D. P., & Tsitsiklis, J. N. (1989). *Parallel and distributed computation: numerical methods*. Prentice-Hall, Inc.
2.  Puterman, M. L., & Shin, M. C. (1978). Modified policy iteration algorithms for discounted Markov decision problems. *Management Science, 24*(11), 1127-1137.
3.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapter 4)
