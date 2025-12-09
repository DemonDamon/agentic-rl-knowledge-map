# 第二章：经典强化学习算法 (Classical Reinforcement Learning Algorithms)

---

## 1. 概述 (Overview)

在奠定了马尔可夫决策过程 (MDP) 和基本学习范式（动态规划、蒙特卡罗、时间差分）的理论基础之后，本章将深入探讨一系列经典的强化学习算法。这些算法构成了现代强化学习方法的基石，并且在许多状态和动作空间较小的“表格型”(tabular) 问题中，它们仍然是高效且理论完备的解决方案。

本章的核心是系统性地梳理和分析在模型未知 (model-free) 环境下，如何通过与环境的交互来学习最优策略。我们将重点关注值函数估计和策略改进这两个核心环节，并展示不同算法如何平衡探索 (exploration) 与利用 (exploitation) 的关系。主要内容包括：

1.  **值迭代与策略迭代 (Value and Policy Iteration):** 虽然在第一章动态规划部分已介绍，本节将从算法实现的角度重新审视它们，并讨论其在经典RL中的变体，如异步更新和修正策略迭代，这些思想对后续的分布式学习有重要启发。

2.  **Q-Learning及其变体:** 作为离策略 (off-policy) 时间差分学习的里程碑算法，Q-Learning 分离了行为策略和目标策略，使其能够从历史经验或探索性策略中学习到最优策略。我们将提供其最优性收敛的严格证明，并探讨其经典变体，如 Double Q-Learning，该变体旨在解决原始算法中的最大化偏差 (maximization bias) 问题。

3.  **SARSA及其变体:** 与 Q-Learning 相对，SARSA 是一种在策略 (on-policy) 学习算法，它直接评估并改进当前正在执行的策略。我们将分析其收敛性，并将其与 Q-Learning 进行对比，以揭示在策略和离策略学习在稳定性和收敛速度上的权衡。同时，我们也会介绍其与资格迹结合的 SARSA(λ) 算法。

4.  **Actor-Critic 方法:** 这是一类结合了值函数学习和策略学习的算法。Actor (策略) 负责选择动作，而 Critic (评论家) 负责评估 Actor 的表现并指导其更新。这种架构是现代深度强化学习中策略梯度方法的基础。本节将介绍基础的 Actor-Critic 框架、优势函数 (Advantage Function) 的概念，以及自然梯度在策略优化中的应用。

5.  **资格迹 (Eligibility Traces):** 作为一种连接蒙特卡罗和时间差分学习的机制，资格迹允许学习信号在时间序列上向后传播，从而更有效地分配信用。我们将探讨 TD(λ)、Q(λ) 和 SARSA(λ) 等算法，并分析不同类型的迹（如累积迹和替换迹）如何影响学习效率和计算复杂度。

通过本章的学习，读者将对经典强化学习算法的全貌有一个清晰的认识，理解它们各自的理论保证、适用场景以及内在联系，为第三章过渡到基于深度学习的大规模强化学习问题做好充分准备。

## 2. 目录 (Table of Contents)

- [**01-值迭代与策略迭代**](./01-值迭代与策略迭代/README.md): 经典DP算法的变体与扩展。
- [**02-Q-Learning及其变体**](./02-Q-Learning及其变体/README.md): 核心的离策略学习算法。
- [**03-SARSA及其变体**](./03-SARSA及其变体/README.md): 核心的在策略学习算法。
- [**04-Actor-Critic方法**](./04-Actor-Critic方法/README.md): 策略与值函数结合的混合方法。
- [**05-资格迹**](./05-资格迹/README.md): 统一MC和TD学习的通用机制。

## 3. 核心参考文献 (Core References)

1.  Watkins, C. J. C. H. (1989). *Learning from delayed rewards*. (Doctoral dissertation, University of Cambridge).
2.  Rummery, G. A., & Niranjan, M. (1994). *Online Q-learning using connectionist systems*. University of Cambridge, Department of Engineering.
3.  Hasselt, H. V. (2010). Double Q-learning. In *Advances in Neural Information Processing Systems, 23*.
4.  Sutton, R. S. (1988). Learning to predict by the methods of temporal differences. *Machine learning, 3*(1), 9-44.
