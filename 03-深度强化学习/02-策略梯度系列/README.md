---

# 3.2 策略梯度系列 (The Policy Gradient Family)

---

## 1. 概述 (Overview)

策略梯度 (Policy Gradient, PG) 方法是深度强化学习中与基于值的（如 DQN）方法并列的一大分支。与学习一个值函数然后隐式地导出策略不同，PG 方法直接对策略 $\pi_\theta(a|s)$ 进行参数化，并直接在策略空间中通过梯度上升来优化一个性能目标函数 $J(\theta)$ [1]。

最基础的策略梯度定理给出了目标函数梯度的一个通用形式：
$$ 

abla_\theta J(\theta) = \mathbb{E}_{s \sim d^\pi, a \sim \pi_\theta} [\nabla_\theta \log \pi_\theta(a|s) \Psi_t] 
$$
其中，$\Psi_t$ 是一个用于评估动作 $a$ 在状态 $s$ 下“有多好”的标量。对 $\Psi_t$ 的不同选择，导致了不同的策略梯度算法：

-   **REINFORCE**: 使用完整的蒙特卡罗回报 $G_t$ 作为 $\Psi_t$。这是最基础的策略梯度算法，但由于 $G_t$ 的高方差，其学习过程通常不稳定且效率低下。
-   **Actor-Critic**: 使用 TD 误差 $\delta_t$ 或优势函数 $A(s,a)$ 作为 $\Psi_t$。这大大降低了梯度的方差，是现代 PG 算法的主流方向。

近年来，为了解决传统 PG 方法的样本效率低和对超参数敏感等问题，研究人员提出了一系列重要的改进算法，它们在稳定性、收敛速度和最终性能上都取得了巨大突破。

本节将重点介绍几个在深度强化学习领域具有里程碑意义的策略梯度算法，它们被广泛应用于机器人控制、游戏 AI 等复杂任务中。

## 2. 目录 (Table of Contents)

- [**01-TRPO**](./01-TRPO/README.md): **信赖域策略优化 (Trust Region Policy Optimization)**，通过在策略更新上施加 KL 散度约束来保证更新的稳定性，避免了灾难性的策略崩溃。
- [**02-PPO**](./02-PPO/README.md): **近端策略优化 (Proximal Policy Optimization)**，作为 TRPO 的一种简化实现，通过一个裁剪的目标函数 (Clipped Objective) 来限制策略更新的幅度，在实现简单性和性能之间取得了极佳的平衡，是目前最流行和最常用的 DRL 算法之一。
- [**03-A3C**](./03-A3C/README.md): **异步优势 Actor-Critic (Asynchronous Advantage Actor-Critic)**，通过并行运行多个智能体并异步更新全局参数，打破了样本相关性，极大地提升了训练速度和稳定性。

## 3. 核心参考文献 (Core References)

1.  Sutton, R. S., McAllester, D. A., Singh, S. P., & Mansour, Y. (2000). Policy gradient methods for reinforcement learning with function approximation. In *Advances in neural information processing systems, 12*.
2.  Schulman, J., Levine, S., Abbeel, P., Jordan, M., & Moritz, P. (2015, June). Trust region policy optimization. In *International conference on machine learning* (pp. 1889-1897).
3.  Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*.
4.  Mnih, V., Badia, A. P., Mirza, M., Graves, A., Lillicrap, T., Harley, T., ... & Kavukcuoglu, K. (2016, June). Asynchronous methods for deep reinforcement learning. In *International conference on machine learning* (pp. 1928-1937).
