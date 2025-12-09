---

# 2.5 资格迹 (Eligibility Traces)

---

## 1. 概述 (Overview)

资格迹 (Eligibility Traces) 是一种在强化学习中用于连接蒙特卡罗 (MC) 方法和时间差分 (TD) 方法的优雅机制 [1]。它通过在时间序列上向后传播信用，使得当前的 TD 误差能够影响到过去访问过的状态-动作对，从而在单步 TD 学习的低方差和 MC 学习的无偏性之间提供了一个平滑的权衡。

想象一下，在一个长长的经验片段中，智能体在最后一步获得了一个巨大的正奖励。对于单步 TD 学习（如 TD(0)），这个奖励信号只会直接影响到倒数第二个状态。而我们直觉上认为，这个奖励很可能是由整个序列中的多个关键决策共同导致的。资格迹正是实现这种“信用回溯分配”的机制。

资格迹是一个与每个状态（或状态-动作对）相关联的短期记忆向量 $E_t(s)$。在每个时间步，所有状态的资格迹都会以一个速率 $\gamma\lambda$ 进行衰减，而当前访问的状态的资格迹则会被增强。其中 $\lambda \in [0, 1]$ 是一个关键参数，控制着衰减的速率，也即信用回溯的长度。

-   当 $\lambda = 0$ 时，只有当前状态的资格迹为 1，其他均为 0。此时，算法退化为单步 TD 学习，如 TD(0)。
-   当 $\lambda = 1$ 时，资格迹的衰减非常缓慢，其效果近似于将整个片段的累积误差用于更新，从而接近于蒙特卡罗方法。

通过调节 $\lambda$ 参数，我们可以在偏差（由自举引入）和方差（由多步回报采样引入）之间进行权衡。资格迹与 TD 学习的结合，催生了一系列强大的算法，如 TD(λ), SARSA(λ) 和 Q(λ)，它们通常比其单步版本的学习效率更高。

本节将探讨资格迹的数学原理和不同类型的实现方式。

## 2. 目录 (Table of Contents)

- [**01-累积迹**](./01-累积迹/README.md): 最常见的资格迹实现方式，每次访问状态时迹增加1。
- [**02-替换迹**](./02-替换迹/README.md): 一种将状态的资格迹重置为1的实现方式，在某些情况下更有效。
- [**03-Dutch迹**](./03-Dutch迹/README.md): 结合了累积迹和替换迹的优点。

## 3. 核心参考文献 (Core References)

1.  Sutton, R. S. (1988). Learning to predict by the methods of temporal differences. *Machine learning, 3*(1), 9-44.
2.  Singh, S. P., & Sutton, R. S. (1996). Reinforcement learning with replacing eligibility traces. *Machine learning, 22*(1), 123-158.
3.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapter 12)
