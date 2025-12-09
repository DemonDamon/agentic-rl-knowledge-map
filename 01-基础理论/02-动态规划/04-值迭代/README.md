---

# 1.2.4 值迭代 (Value Iteration)

---

## 1. 概述 (Overview)

值迭代是另一种求解 MDP 最优策略的核心动态规划算法 [1]。与策略迭代不同，值迭代通过一种更直接的方式来寻找最优值函数 $V^*$，它将策略评估和策略改进两个步骤有效地结合在了一起。

值迭代的核心思想是直接迭代贝尔曼最优性方程：
$$ 
V^*(s) = \max_{a \in \mathcal{A}} \left( R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V^*(s') \right) 
$$

值迭代算法将这个方程转化为一个迭代更新规则。从一个任意的初始值函数 $V_0$ 开始，算法在每一步 $k$ 都应用贝尔曼最优性算子来生成新的值函数 $V_{k+1}$：
$$ 
V_{k+1}(s) \leftarrow \max_{a \in \mathcal{A}} \left( R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V_k(s') \right) 
$$

这个过程可以被看作是策略迭代的一个变体，其中策略评估的步骤被极大地简化了：它只进行**一次**迭代更新，而不是等待值函数完全收敛。换句话说，值迭代在每一步都将策略改进（通过 `max` 操作）和策略评估（通过使用上一轮的 $V_k$）紧密地结合在一起。

由于贝尔曼最优性算子是一个收缩映射，这个迭代过程保证会收敛到唯一的最优值函数 $V^*$。一旦值函数收敛（或足够接近最优值），就可以通过一次贪婪策略提取来获得一个近似最优的策略。

与策略迭代相比，值迭代的每次迭代计算量更小（因为它不包含一个完整的策略评估内循环），但它通常需要更多的迭代次数才能收敛。在状态空间巨大时，值迭代通常是更受青睐的选择。

本节将详细介绍值迭代的算法流程，提供其基于巴拿赫不动点定理的收敛性证明，并将其与策略迭代进行深入的比较。

## 2. 目录 (Table of Contents)

- [**01-算法流程**](./01-算法流程/README.md): 值迭代的具体实现步骤和伪代码。
- [**02-收敛性证明**](./02-收敛性证明/README.md): 证明算法为何能保证收敛到最优值函数。
- [**03-与策略迭代的比较**](./03-与策略迭代的比较/README.md): 分析两种核心 DP 算法的优缺点和适用场景。

## 3. 核心参考文献 (Core References)

1.  Bellman, R. (1957). A Markovian decision process. *Journal of Mathematics and Mechanics*, 673-684.
2.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 4.4)
3.  Bertsekas, D. P. (2012). *Dynamic programming and optimal control: Volume I*. Athena scientific.
