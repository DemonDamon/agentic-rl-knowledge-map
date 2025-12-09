---

# 1.3 蒙特卡罗方法 (Monte Carlo Methods)

---

## 1. 概述 (Overview)

当环境模型未知时，动态规划 (DP) 的方法便不再适用。蒙特卡罗 (Monte Carlo, MC) 方法是第一类专门为无模型 (model-free) 场景设计的强化学习算法。与 DP 不同，MC 方法不依赖于状态转移概率和奖励函数的先验知识，而是直接从与环境交互得到的完整经验片段 (episodes) 中学习值函数 [1]。

一个经验片段是指从某个起始状态开始，遵循某一策略，直至到达终止状态的完整状态-动作-奖励序列。MC 方法的核心思想是基于“均值”的概念：一个状态的值函数被估计为从该状态开始的所有经验片段的累积回报 (return) 的平均值。根据大数定律 (Law of Large Numbers)，只要我们收集足够多的经验片段，这个平均回报就会收敛到真实的值函数。

本节将详细介绍 MC 方法的基本原理及其在策略评估和控制中的应用。主要内容包括：

1.  **首次访问与每次访问 MC (First-Visit vs. Every-Visit MC):** 在估计一个状态的值函数时，对于一个经验片段中多次出现该状态的情况，我们有两种处理方式。首次访问 MC 只考虑该片段中第一次访问该状态后的回报，而每次访问 MC 则考虑每一次访问后的回报。我们将分析这两种方法的偏差和收敛性质。

2.  **探索与利用 (Exploration and Exploitation):** 在使用 MC 方法进行策略改进（即控制）时，会遇到一个经典的两难问题：是为了获得更精确的当前最优动作的估计而“利用” (exploit) 已知信息，还是为了发现可能更优的未知动作而“探索” (explore) 新的选择？我们将介绍两种处理这一问题的技术：**探索起始 (Exploring Starts)** 和 **ε-贪婪策略 (ε-greedy policies)**。

3.  **重要性采样 (Importance Sampling):** 为了实现离策略 (off-policy) 学习——即从一个行为策略 (behavior policy) 生成的经验中学习一个目标策略 (target policy)——我们需要一种方法来修正两种策略分布不匹配带来的影响。重要性采样正是实现这一目标的数学工具。我们将推导 **普通重要性采样 (Ordinary Importance Sampling)** 和 **加权重要性采样 (Weighted Importance Sampling)** 的更新规则，并分析它们在偏差和方差上的权衡。

MC 方法的主要优点是其估计是无偏的 (unbiased)，并且不依赖于贝尔曼方程的自举 (bootstrapping)，因此在处理非马尔可夫环境时具有一定的鲁棒性。然而，它也存在明显的缺点，如只能应用于片段式任务 (episodic tasks)，并且通常具有较高的方差，学习效率较低。理解 MC 方法的原理和局限性，对于理解下一节更高效的时间差分学习至关重要。

## 2. 目录 (Table of Contents)

- [**01-首次访问MC**](./01-首次访问MC/README.md): 基于片段中第一次出现的状态进行估计。
- [**02-每次访问MC**](./02-每次访问MC/README.md): 基于片段中所有出现的状态进行估计。
- [**03-探索与利用**](./03-探索与利用/README.md): 平衡学习与决策的核心问题。
- [**04-重要性采样**](./04-重要性采样/README.md): 实现离策略学习的数学工具。

## 3. 核心参考文献 (Core References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapter 5)
2.  Singh, S. P., & Sutton, R. S. (1996). Reinforcement learning with replacing eligibility traces. *Machine learning, 22*(1), 123-158.
3.  Precup, D., Sutton, R. S., & Singh, S. (2000). Eligibility traces for off-policy policy evaluation. In *International Conference on Machine Learning*.
