---

# 1.5 函数逼近 (Function Approximation)

---

## 1. 概述 (Overview)

前面章节讨论的表格型 (tabular) 方法，其前提是状态和动作空间足够小，使得值函数或策略可以显式地存储在表格中。然而，在许多现实世界的应用中，状态空间可能是巨大的（如棋类游戏）甚至是连续的（如机器人控制），这使得表格方法在存储和计算上都变得不可行。函数逼近 (Function Approximation) 为解决这一“维度灾难” (curse of dimensionality) 提供了强大的工具 [1]。

其核心思想是，不再为每个状态单独存储一个值，而是用一个参数化的函数 $\hat{v}(s, \mathbf{w})$ 或 $\hat{q}(s, a, \mathbf{w})$ 来近似真实的值函数 $V^\pi(s)$ 或 $Q^\pi(s, a)$。其中 $\mathbf{w} \in \mathbb{R}^d$ 是一个维度远小于状态空间大小 $|\mathcal{S}|$ 的参数向量。通过这种方式，我们希望智能体能够将从有限的、已访问过的状态中学到的知识**泛化**到未曾见过的状态。

本节将介绍将函数逼近与强化学习相结合的基本原理和挑战。主要内容包括：

1.  **线性函数逼近 (Linear Function Approximation):** 作为最简单也是理论分析最成熟的一类函数逼近方法，线性方法将值函数表示为状态特征的线性组合 $\hat{v}(s, \mathbf{w}) = \mathbf{w}^\top \mathbf{x}(s)$。我们将讨论如何设计有效的**特征 (features)**，以及如何使用随机梯度下降 (Stochastic Gradient Descent, SGD) 等优化算法来更新权重向量 $\mathbf{w}$。

2.  **非线性函数逼近 (Non-linear Function Approximation):** 深度神经网络是目前最强大的非线性函数逼近器。本节将简要介绍如何使用深度神经网络来表示值函数或策略，这构成了深度强化学习 (DRL) 的基础。我们将为第三章的深入探讨做铺垫。

3.  **致命三元组 (The Deadly Triad):** 当函数逼近、自举 (bootstrapping) 和离策略 (off-policy) 训练这三个元素同时出现时，学习过程可能会变得不稳定甚至发散。这一现象被称为“致命三元组” [1]。理解这一问题是设计稳定高效的 DRL 算法的关键，例如 DQN 中的经验回放和目标网络正是为了缓解这一问题。

4.  **经验回放 (Experience Replay):** 为了打破数据之间的时序相关性，提高样本利用率，经验回放机制被引入。它将智能体的经验存储在一个回放缓冲区中，并在训练时从中随机采样。这一技术是稳定基于函数逼近的离策略学习的关键组件。

通过本节的学习，读者将理解为何函数逼近是强化学习走向大规模应用的关键一步，掌握其基本方法，并认识到它所带来的新的理论和实践挑战。

## 2. 目录 (Table of Contents)

- [**01-线性函数逼近**](./01-线性函数逼近/README.md): 使用特征的线性组合来近似值函数。
- [**02-非线性函数逼近**](./02-非线性函数逼近/README.md): 使用神经网络等模型进行近似。
- [**03-致命三元组**](./03-致命三元组/README.md): 函数逼近、自举和离策略结合时的不稳定性问题。
- [**04-经验回放**](./04-经验回放/README.md): 稳定离策略学习的关键技术。

## 3. 核心参考文献 (Core References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapters 9-11)
2.  Tsitsiklis, J. N., & Van Roy, B. (1997). An analysis of temporal-difference learning with function approximation. *IEEE transactions on Automatic Control, 42*(5), 674-690.
3.  Mnih, V., Kavukcuoglu, K., Silver, D., et al. (2015). Human-level control through deep reinforcement learning. *Nature, 518*(7540), 529-533.
