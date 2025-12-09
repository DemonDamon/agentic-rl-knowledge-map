# 第一章：强化学习的基础理论 (Foundations of Reinforcement Learning)

---

## 1. 概述 (Overview)

本章旨在为强化学习 (Reinforcement Learning, RL) 领域奠定坚实的数学基础。我们将从最核心的建模工具——马尔可夫决策过程 (Markov Decision Process, MDP)——入手，系统性地介绍用于解决序贯决策问题的基本理论框架和经典方法。对于以数学为背景的研究者而言，理解这一章的内容是掌握现代高级强化学习算法的先决条件。

强化学习的本质是在一个不确定的动态环境中，通过与环境的交互来学习一个最优策略，以最大化累积奖励信号。这一过程的数学抽象便是 MDP。本章将涵盖以下核心主题：

1.  **马尔可夫决策过程 (MDPs):** 形式化定义智能体 (Agent) 与环境 (Environment) 交互的数学框架，引入状态、动作、转移概率、奖励函数和折扣因子等核心概念，并推导贝尔曼方程 (Bellman Equation)，它是几乎所有强化学习算法的理论基石。

2.  **动态规划 (Dynamic Programming, DP):** 在假设环境模型完全已知的前提下，介绍求解 MDP 的经典方法。我们将深入探讨策略评估 (Policy Evaluation) 和策略改进 (Policy Improvement) 的迭代过程，并由此引出两种核心算法：策略迭代 (Policy Iteration) 和值迭代 (Value Iteration)。本节将包含严格的收敛性证明，如巴拿赫不动点定理的应用。

3.  **蒙特卡罗方法 (Monte Carlo, MC):** 转向模型未知 (Model-Free) 的场景，介绍第一类主要的学习方法。MC 方法通过从完整的经验片段 (Episode) 中学习值函数，避免了对环境动态的显式建模。我们将讨论其无偏估计的性质以及方差问题。

4.  **时间差分学习 (Temporal-Difference, TD):** 结合了动态规划的自举 (Bootstrapping) 特性和蒙特卡罗方法的无模型学习能力，TD 学习是现代强化学习中最为核心和广泛应用的范式。本节将详细推导 TD(0)、SARSA 和 Q-Learning 等算法，并分析其收敛条件与偏差-方差权衡。

5.  **函数逼近 (Function Approximation):** 为了将上述理论扩展到具有巨大或连续状态空间的问题，本节将引入函数逼近的概念。我们将讨论如何使用线性模型或非线性模型（如神经网络）来近似值函数或策略，并分析由此带来的挑战，如“致命三元组” (The Deadly Triad)。

通过本章的学习，读者将能够从第一性原理出发，理解强化学习问题的数学本质、核心方程以及求解这些问题的基本思路，为后续深入学习更复杂的深度强化学习和多智能体算法打下坚实的基础。

## 2. 目录 (Table of Contents)

- [**01-MDP与马尔可夫过程**](./01-MDP与马尔可夫过程/README.md): 强化学习问题的形式化描述。
- [**02-动态规划**](./02-动态规划/README.md): 基于已知模型的精确求解方法。
- [**03-蒙特卡罗方法**](./03-蒙特卡罗方法/README.md): 从完整经验中学习的无模型方法。
- [**04-时间差分学习**](./04-时间差分学习/README.md): 结合了DP和MC优点的核心无模型方法。
- [**05-函数逼近**](./05-函数逼近/README.md): 将RL扩展到大规模问题的方法。

## 3. 核心参考文献 (Core References)

本章内容主要基于以下经典教材，建议读者深入阅读以获得更全面的理解：

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press.
2.  Bertsekas, D. P. (2019). *Reinforcement learning and optimal control*. Athena Scientific.
3.  Puterman, M. L. (2014). *Markov decision processes: discrete stochastic dynamic programming*. John Wiley & Sons.
