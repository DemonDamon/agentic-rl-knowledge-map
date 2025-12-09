---

# 3.3 确定性策略梯度系列 (The Deterministic Policy Gradient Family)

---

## 1. 概述 (Overview)

在处理具有连续动作空间的任务时（如机器人控制），传统的随机策略梯度方法面临挑战。因为在连续空间中，通过采样来估计期望梯度变得非常困难。确定性策略梯度 (Deterministic Policy Gradient, DPG) 理论的提出，为解决这一问题提供了坚实的数学基础 [1]。

与随机策略 $\pi_\theta(a|s)$ 输出一个动作的概率分布不同，确定性策略 $\mu_\theta(s)$ 直接输出一个确定的动作 $a = \mu_\theta(s)$。DPG 定理证明，确定性策略的梯度可以表示为一个更简单的形式，它不依赖于对动作的积分，而是直接对 Q 函数的梯度进行期望计算：
$$ 

abla_\theta J(\theta) = \mathbb{E}_{s \sim \rho^\mu} [\nabla_\theta \mu_\theta(s) \nabla_a Q^\mu(s, a) |_{a=\mu_\theta(s)}] 
$$
其中 $\rho^\mu$ 是在策略 $\mu$ 下的状态访问分布。这个梯度形式直观地告诉我们：应该朝着能使 Q 值增加最快的方向调整策略。

基于 DPG 理论，研究人员开发了一系列强大的离策略 (off-policy) Actor-Critic 算法，它们结合了 DQN 的经验回放机制，从而极大地提高了样本效率。这些算法在连续控制任务中取得了巨大的成功。

本节将重点介绍 DPG 理论及其最重要的算法实现。

## 2. 目录 (Table of Contents)

- [**01-DPG**](./01-DPG/README.md): **确定性策略梯度 (Deterministic Policy Gradient)** 的核心理论和数学推导。
- [**02-DDPG**](./02-DDPG/README.md): **深度确定性策略梯度 (Deep Deterministic Policy Gradient)**，将 DPG 与 DQN 的思想（经验回放、目标网络）相结合，是第一个在连续控制领域取得重大成功的 DRL 算法。
- [**03-TD3**](./03-TD3/README.md): **双延迟深度确定性策略梯度 (Twin Delayed Deep Deterministic Policy Gradient)**，作为 DDPG 的直接改进，通过引入三个关键技术（Clipped Double Q-Learning, Delayed Policy Updates, Target Policy Smoothing）来解决 DDPG 中 Q 值被高估和策略更新不稳定的问题。
- [**04-SAC**](./04-SAC/README.md): **软 Actor-Critic (Soft Actor-Critic)**，一种基于最大熵强化学习框架的离策略算法。它通过在目标函数中加入一个熵正则项来鼓励探索，从而获得了极高的样本效率和稳定性，是目前连续控制领域的 SOTA (State-of-the-Art) 算法之一。

## 3. 核心参考文献 (Core References)

1.  Silver, D., Lever, G., Heess, N., Degris, T., Wierstra, D., & Riedmiller, M. (2014, June). Deterministic policy gradient algorithms. In *International conference on machine learning*.
2.  Lillicrap, T. P., Hunt, J. J., Pritzel, A., Heess, N., Erez, T., Tassa, Y., ... & Wierstra, D. (2015). Continuous control with deep reinforcement learning. *arXiv preprint arXiv:1509.02971*.
3.  Fujimoto, S., Hoof, H., & Meger, D. (2018, July). Addressing function approximation error in actor-critic methods. In *International conference on machine learning* (pp. 1587-1596).
4.  Haarnoja, T., Zhou, A., Abbeel, P., & Levine, S. (2018, July). Soft actor-critic: Off-policy maximum entropy deep reinforcement learning with a stochastic actor. In *International conference on machine learning* (pp. 1861-1870).
