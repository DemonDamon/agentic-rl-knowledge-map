---

# 2.4 Actor-Critic 方法 (Actor-Critic Methods)

---

## 1. 概述 (Overview)

Actor-Critic (AC) 方法是一类巧妙结合了**策略梯度方法 (Policy Gradient Methods)** 和**值函数学习 (Value Function Learning)** 的强化学习算法 [1]。它通过引入两个独立的组件——**Actor** 和 **Critic**——来克服各自方法的缺点，是现代深度强化学习中许多前沿算法（如 A3C, DDPG, SAC）的基础架构。

在纯粹的策略梯度方法（如 REINFORCE）中，策略更新的梯度依赖于蒙特卡罗方法对完整回报 $G_t$ 的采样，这导致了非常高的方差，使得学习过程缓慢且不稳定。另一方面，纯粹的值函数方法（如 Q-Learning）虽然可以通过自举 (bootstrapping) 降低方差，但在连续动作空间或随机策略的学习上存在困难。

Actor-Critic 方法通过以下方式结合了两者的优点：

1.  **Actor (演员)**: Actor 是一个参数化的**策略** $\pi_\theta(a|s)$，负责根据当前状态 $s$ 选择动作 $a$。它的目标是学习一个最优策略，其更新方向由 Critic 提供。

2.  **Critic (评论家)**: Critic 是一个参数化的**值函数**，通常是状态值函数 $V_w(s)$ 或动作值函数 $Q_w(s, a)$。它负责评估 Actor 所选择的动作“有多好”，并向 Actor 提供一个低方差的、信息丰富的学习信号。这个信号通常是 TD 误差 $\delta_t$。

基础的 Actor-Critic 算法流程如下：在状态 $S_t$，Actor 选择动作 $A_t$。环境响应并给出 $R_{t+1}$ 和 $S_{t+1}$。Critic 计算 TD 误差：
$$ 
\delta_t = R_{t+1} + \gamma V_w(S_{t+1}) - V_w(S_t) 
$$
这个 TD 误差同时用于更新 Actor 和 Critic 的参数：
-   **Critic 更新**: Critic 使用 TD 误差来更好地逼近真实值函数，其更新方向为 $\alpha_w \delta_t \nabla_w V_w(S_t)$。
-   **Actor 更新**: Actor 使用 TD 误差作为对动作 $A_t$ 表现的评价。如果 $\delta_t > 0$，说明这个动作比预期的要好，就增加选择它的概率；反之则减少。其更新方向为 $\alpha_\theta \delta_t \nabla_\theta \log \pi_\theta(A_t|S_t)$。

通过使用 TD 误差代替蒙特卡罗回报，Actor-Critic 方法显著降低了策略梯度的方差。本节将探讨 Actor-Critic 的基础架构及其重要改进。

## 2. 目录 (Table of Contents)

- [**01-基础AC架构**](./01-基础AC架构/README.md): 介绍最基本的 Actor-Critic 算法流程和数学原理。
- [**02-优势Actor-Critic**](./02-优势Actor-Critic/README.md): 引入优势函数 $A(s,a) = Q(s,a) - V(s)$ 作为更有效的学习信号。
- [**03-自然梯度AC**](./03-自然梯度AC/README.md): 使用自然梯度代替标准梯度来优化策略，以获得更稳定和高效的更新。

## 3. 核心参考文献 (Core References)

1.  Konda, V. R., & Tsitsiklis, J. N. (2000). Actor-critic algorithms. In *Advances in neural information processing systems, 12*.
2.  Sutton, R. S., McAllester, D. A., Singh, S. P., & Mansour, Y. (2000). Policy gradient methods for reinforcement learning with function approximation. In *Advances in neural information processing systems, 12*.
3.  Kakade, S. M. (2002). A natural policy gradient. In *Advances in neural information processing systems, 14*.
