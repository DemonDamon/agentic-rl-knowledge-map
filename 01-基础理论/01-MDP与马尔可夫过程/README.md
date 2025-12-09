---

# 1.1 MDP与马尔可夫过程 (MDP and Markov Processes)

---

## 1. 概述 (Overview)

马尔可夫决策过程 (Markov Decision Process, MDP) 是强化学习用于对环境进行形式化描述的核心数学框架 [1]。它建立在马尔可夫过程 (Markov Process) 或马尔可夫链 (Markov Chain) 的概念之上，通过引入决策者（即智能体）的动作和与之关联的奖励信号，将纯粹的随机过程转变为一个序贯决策问题。

理解 MDP 的关键在于掌握其核心假设——**马尔可夫性质 (Markov Property)**。该性质断言，系统的未来状态只依赖于当前状态，而与导致当前状态的历史路径无关。这个看似简单的假设极大地简化了决策问题的建模，使得我们可以基于当前可观测的信息来制定最优决策。

本节内容将从最基础的马尔可夫性质出发，逐步构建完整的 MDP 框架。主要内容包括：

1.  **马尔可夫性质:** 从概率论的角度严格定义马尔可夫性质，即给定现在，未来与过去独立。我们将使用条件概率来形式化这一概念。

2.  **MDP 形式化定义:** 在马尔可夫过程的基础上，引入动作和奖励，给出一个五元组 $(\mathcal{S}, \mathcal{A}, P, R, \gamma)$ 来完整定义一个 MDP。我们将详细解释每个组成部分的数学意义，包括状态空间、动作空间、状态转移概率函数、奖励函数以及折扣因子。

3.  **状态转移与奖励:** 深入分析状态转移函数 $P(s'|s, a)$ 和奖励函数 $R(s, a, s')$ 的数学形式与内涵。这二者共同定义了环境的动态模型 (dynamics)。

4.  **贝尔曼方程:** 推导强化学习中最核心的方程——贝尔曼方程。该方程以递归形式建立了当前状态（或状态-动作对）的值函数与后继状态值函数之间的关系。我们将分别介绍状态值函数 $V^\pi(s)$ 和动作值函数 $Q^\pi(s, a)$ 的贝尔曼期望方程，以及最优值函数 $V^*(s)$ 和 $Q^*(s, a)$ 的贝尔曼最优性方程。贝尔曼方程是后续所有动态规划和时间差分学习算法的理论基础。

通过本节的学习，读者将能够运用数学语言精确地描述一个强化学习问题，并理解其底层动态和优化目标。

## 2. 目录 (Table of Contents)

- [**01-马尔可夫性质**](./01-马尔可夫性质/README.md): 决策过程的基础假设。
- [**02-MDP形式化定义**](./02-MDP形式化定义/README.md): 强化学习问题的数学框架。
- [**03-状态转移与奖励**](./03-状态转移与奖励/README.md): 环境动态的数学描述。
- [**04-贝尔曼方程**](./04-贝尔曼方程/README.md): 求解 MDP 的核心递归方程。

## 3. 核心参考文献 (Core References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapter 3)
2.  Puterman, M. L. (2014). *Markov decision processes: discrete stochastic dynamic programming*. John Wiley & Sons.
3.  Bellman, R. (1957). A Markovian decision process. *Journal of Mathematics and Mechanics*, 673-684.
