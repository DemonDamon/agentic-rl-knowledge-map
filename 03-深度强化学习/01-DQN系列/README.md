---

# 3.1 DQN 系列 (The DQN Family)

---

## 1. 概述 (Overview)

深度Q网络 (Deep Q-Network, DQN) 是深度强化学习领域的开创性工作，由 DeepMind（当时）的研究人员在2013年首次提出，并于2015年发表在《自然》杂志上 [1]。DQN 首次成功地将深度卷积神经网络 (CNN) 与经典的 Q-Learning 算法相结合，并直接从高维原始像素输入（如 Atari 2600 游戏画面）中学习控制策略，达到了超越人类专业玩家的水平。

DQN 的成功关键在于它解决了将非线性函数逼近器（如神经网络）与离策略、自举的 TD 学习相结合时面临的“致命三元组” (Deadly Triad) 问题，即学习过程的不稳定性。它引入了两项核心技术来稳定训练：

1.  **经验回放 (Experience Replay):** 智能体将过去的经验（状态、动作、奖励、下一状态的元组）存储在一个称为“回放缓冲区” (replay buffer) 的大型内存中。在训练时，算法从缓冲区中随机采样一个小批量 (mini-batch) 的经验来更新网络。这打破了训练样本之间的时序相关性，使得样本更接近于独立同分布 (i.i.d.)，从而稳定了训练。

2.  **目标网络 (Target Network):** 算法使用一个独立的、周期性更新的“目标网络”来生成 TD 目标中的 Q 值。在计算 TD 目标 $R + \gamma \max_{a'} Q(s', a')$ 时，$\max$ 操作中的 Q 值由一个参数为 $\theta^-$ 的目标网络 $Q(s', a'; \theta^-)$ 计算，而要更新的 Q 值则由参数为 $\theta$ 的在线网络 $Q(s, a; \theta)$ 计算。目标网络的参数 $\theta^-$ 会定期从在线网络复制而来，并在一段时间内保持固定。这大大降低了 TD 目标与当前 Q 值之间的相关性，避免了策略的剧烈振荡和发散。

在 DQN 的原始成功之后，研究人员提出了一系列重要的改进，进一步提升了其性能和稳定性。本节将深入探讨原始的 DQN 算法及其最重要的变体，它们共同构成了现代基于值的深度强化学习的基础。

## 2. 目录 (Table of Contents)

- [**01-DQN**](./01-DQN/README.md): 原始的深度Q网络算法，包括其网络架构和核心技术。
- [**02-Double-DQN**](./02-Double-DQN/README.md): 旨在解决 DQN 中最大化偏差导致的值过高估计问题。
- [**03-Dueling-DQN**](./03-Dueling-DQN/README.md): 将 Q 值分解为状态值和优势值，以更有效地学习状态的内在价值。
- [**04-Rainbow**](./04-Rainbow/README.md): 集成了 DQN 的多种改进（包括 Double DQN, Dueling DQN, Prioritized Replay, Noisy Nets, Distributional RL, Multi-step learning）的集大成算法。
- [**05-其他变体**](./05-其他变体/README.md): 介绍如分布式 Q-Learning (Categorical DQN, QR-DQN) 等其他重要方向。

## 3. 核心参考文献 (Core References)

1.  Mnih, V., Kavukcuoglu, K., Silver, D., Rusu, A. A., Veness, J., Bellemare, M. G., ... & Hassabis, D. (2015). Human-level control through deep reinforcement learning. *Nature, 518*(7540), 529-533.
2.  Van Hasselt, H., Guez, A., & Silver, D. (2016, March). Deep reinforcement learning with double q-learning. In *Proceedings of the AAAI conference on artificial intelligence* (Vol. 30, No. 1).
3.  Wang, Z., Schaul, T., Hessel, M., Van Hasselt, H., Lanctot, M., & De Freitas, N. (2016, June). Dueling network architectures for deep reinforcement learning. In *International conference on machine learning* (pp. 1995-2003).
4.  Hessel, M., Modayil, J., Van Hasselt, H., Schaul, T., Ostrovski, G., Dabney, W., ... & Silver, D. (2018). Rainbow: Combining improvements in deep reinforcement learning. In *Proceedings of the AAAI conference on artificial intelligence* (Vol. 32, No. 1).
