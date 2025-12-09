# 第三章：深度强化学习 (Deep Reinforcement Learning)

---

## 1. 概述 (Overview)

随着深度学习 (Deep Learning) 在计算机视觉和自然语言处理等领域取得巨大成功，将其与强化学习相结合的深度强化学习 (Deep Reinforcement Learning, DRL) 已成为现代人工智能研究的核心驱动力。本章将全面介绍 DRL 的关键算法和思想，重点关注如何利用深度神经网络作为强大的非线性函数逼近器来解决具有高维、连续状态和动作空间的问题。

DRL 的核心思想是用深度神经网络来参数化策略 (policy) $\pi_\theta(a|s)$ 或值函数 (value function) $V_\theta(s)$, $Q_\theta(s,a)$，然后通过梯度优化方法来更新网络参数 $\theta$。本章将系统性地梳理 DRL 的三大支柱：基于值的 (Value-based)、基于策略的 (Policy-based) 和混合方法 (Actor-Critic)。主要内容包括：

1.  **DQN (Deep Q-Network) 系列:** 作为 DRL 的开创性工作，DQN 成功地将深度卷积网络与 Q-Learning 相结合，在 Atari 游戏上达到了超越人类的水平。我们将详细剖析其两大关键技术：**经验回放 (Experience Replay)** 和 **目标网络 (Target Network)**，并从数学上分析它们如何打破数据相关性、稳定训练过程。此外，我们还将深入探讨其重要变体，如 **Double DQN** (解决Q值过高估计问题)、**Dueling DQN** (解耦状态值和优势函数) 以及集大成者 **Rainbow**。

2.  **策略梯度方法 (Policy Gradient Methods):** 此类方法直接对策略进行参数化并优化。我们将从 **策略梯度定理 (Policy Gradient Theorem)** 的严格证明出发，推导经典的 **REINFORCE** 算法。为了解决其高方差问题，我们将引入 **基线 (Baseline)** 的概念，并最终过渡到现代 Actor-Critic 架构。本节的重点是两种主流的策略优化算法：**TRPO (Trust Region Policy Optimization)**，它通过在策略更新上施加 KL 散度约束来保证单调性能改进；以及其简化版 **PPO (Proximal Policy Optimization)**，它通过一个裁剪的目标函数 (Clipped Objective) 在实现稳定更新的同时大大降低了计算复杂度。

3.  **确定性策略梯度 (Deterministic Policy Gradient):** 针对连续动作空间，我们将介绍确定性策略梯度 (DPG) 算法，特别是其深度学习版本 **DDPG (Deep Deterministic Policy Gradient)**。随后，我们将探讨其重要改进，如 **TD3 (Twin Delayed Deep Deterministic Policy Gradient)**，它通过使用双 Q 网络、延迟策略更新和目标策略平滑来缓解 DDPG 的过估计和收敛不稳问题。最后，我们将详细介绍当前最先进的离策略算法之一 **SAC (Soft Actor-Critic)**，它在最大化累积奖励的同时也最大化策略的熵，从而实现了卓越的探索效率和鲁棒性。

4.  **分布式强化学习 (Distributed RL):** 为了进一步提升学习效率和扩展性，本节将介绍分布式训练框架，如 **A3C (Asynchronous Advantage Actor-Critic)**，它利用多个并行的智能体异步更新全局参数。我们还将讨论更现代的分布式架构，如 **Ape-X** 和 **IMPALA**，它们通过解耦数据采集和学习过程，实现了前所未有的规模和性能。

通过本章的学习，读者将掌握当前 DRL 领域最核心和最前沿的算法，理解其背后的数学原理、实现细节以及适用场景，为进行独立的 DRL 研究或解决复杂的实际问题打下坚实的基础。

## 2. 目录 (Table of Contents)

- [**01-DQN系列**](./01-DQN系列/README.md): 基于深度值函数学习的开创性工作。
- [**02-策略梯度方法**](./02-策略梯度方法/README.md): 直接优化参数化策略的核心方法。
- [**03-确定性策略梯度**](./03-确定性策略梯度/README.md): 适用于连续动作空间的高效算法。
- [**04-分布式强化学习**](./04-分布式强化学习/README.md): 大规模并行训练框架。

## 3. 核心参考文献 (Core References)

1.  Mnih, V., Kavukcuoglu, K., Silver, D., et al. (2015). Human-level control through deep reinforcement learning. *Nature, 518*(7540), 529-533.
2.  Schulman, J., Levine, S., Abbeel, P., Jordan, M., & Moritz, P. (2015). Trust region policy optimization. In *International conference on machine learning*.
3.  Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*.
4.  Haarnoja, T., Zhou, A., Abbeel, P., & Levine, S. (2018). Soft actor-critic: Off-policy maximum entropy deep reinforcement learning with a stochastic actor. In *International conference on machine learning*.
5.  Lillicrap, T. P., Hunt, J. J., Pritzel, A., et al. (2016). Continuous control with deep reinforcement learning. In *International Conference on Learning Representations*.
