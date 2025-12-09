---

# 2.4.1 Actor-Critic 基本原理 (Basic Principles of Actor-Critic)

---

## 1. 算法概述 (Algorithm Overview)

Actor-Critic (AC) 方法是一类结合了**策略梯度 (Policy Gradient)** 方法和**值函数学习 (Value Function Learning)** 方法的强化学习算法。它通过引入一个“评论家 (Critic)”来指导“演员 (Actor)”的动作选择，从而在纯策略方法和纯值方法之间取得平衡，旨在降低策略梯度的方差并提高样本效率。

AC 架构包含两个核心组件：

1.  **演员 (Actor)**: 演员是一个**策略 (policy)**，通常用 $\pi_\theta(a|s)$ 表示，其中 $\theta$ 是策略的参数。它的任务是根据当前状态 $s$ 来选择一个动作 $a$。学习的目标是调整参数 $\theta$ 以使得策略表现得更好。

2.  **评论家 (Critic)**: 评论家是一个**值函数 (value function)**，通常是状态值函数 $V_w(s)$ 或动作值函数 $Q_w(s, a)$，其中 $w$ 是值函数的参数。它的任务是“评论”或“评估”演员所选择的动作有多好。这个评估信息随后被用来指导演员的更新。

**核心思想:**

纯策略梯度方法（如 REINFORCE）使用完整的蒙特卡罗回报 $G_t$ 来更新策略。虽然这种方法是无偏的，但由于回报 $G_t$ 包含了从当前步到片段结束的所有随机奖励，其方差非常高，导致学习过程不稳定且缓慢。

AC 方法用评论家的值函数估计来替代蒙特卡罗回报。评论家学习一个对当前策略 $\pi_\theta$ 的值函数估计 $V_w(s) \approx V^{\pi_\theta}(s)$。这个估计通常比 $G_t$ 的方差小得多。

演员的更新不再依赖于完整的、高方差的回报，而是依赖于评论家给出的、更稳定的 **TD 误差 (TD Error)**：
$$ 
\delta_t = R_{t+1} + \gamma V_w(S_{t+1}) - V_w(S_t) 
$$

这个 TD 误差直观地告诉我们，在状态 $S_t$ 执行动作 $A_t$ 后的结果是比预期的要好（$\delta_t > 0$）还是差（$\delta_t < 0$）。演员根据这个信号来调整策略：如果结果比预期的好，就增加未来在 $S_t$ 选择 $A_t$ 的概率；反之则减少。

## 2. 算法流程 (Algorithm Flow)

一个典型的单步 Actor-Critic 算法流程如下：

1.  **初始化**: 初始化演员的参数 $\theta$ 和评论家的参数 $w$。
2.  **交互**: 在状态 $S_t$，演员根据策略 $\pi_\theta(a|S_t)$ 选择并执行动作 $A_t$。
3.  **观察**: 环境返回奖励 $R_{t+1}$ 和新状态 $S_{t+1}$。
4.  **评论家更新**: 评论家计算 TD 误差，并使用这个误差来更新自己的参数 $w$。这通常是一个标准的 TD(0) 学习过程，目标是最小化 TD 误差的平方：
    $$ 
    \delta_t = R_{t+1} + \gamma V_w(S_{t+1}) - V_w(S_t) 
    $$
    $$ 
    w \leftarrow w + \alpha_w \delta_t \nabla_w V_w(S_t) 
    $$
    其中 $\alpha_w$ 是评论家的学习率。

5.  **演员更新**: 演员使用评论家提供的 TD 误差来更新自己的参数 $\theta$。更新的方向是策略梯度的方向，但用 TD 误差来代替回报 $G_t$：
    $$ 
    \theta \leftarrow \theta + \alpha_\theta \delta_t \nabla_\theta \log \pi_\theta(A_t|S_t) 
    $$
    其中 $\alpha_\theta$ 是演员的学习率。

6.  **循环**: 令 $S_t \leftarrow S_{t+1}$，重复步骤 2-5。

## 3. 图示 (Illustration)

```mermaid
graph TD
    subgraph Environment
        S_t(State)
        R_t1(Reward)
        S_t1(New State)
    end

    subgraph Agent
        subgraph Actor
            Policy[π_θ(a|s)]
        end
        subgraph Critic
            ValueFunc[V_w(s)]
        end
    end

    S_t -- gives state --> Policy
    Policy -- selects action --> A_t(Action)
    A_t -- acts on --> Environment
    Environment -- gives --> R_t1 & S_t1

    R_t1 & S_t1 -- provide feedback --> ValueFunc
    ValueFunc -- computes TD Error --> δ_t(TD Error)
    
    δ_t -- updates --> ValueFunc
    δ_t -- updates --> Policy
```

## 4. 与其他方法的联系 (Connection to Other Methods)

-   **与 REINFORCE 的关系**: 如果将评论家看作是学习回报的基线 (baseline)，那么 AC 方法可以被视为 REINFORCE with Baseline 的一种自适应版本。在 REINFORCE with Baseline 中，基线通常是状态值函数的估计，而 AC 方法正是通过一个独立的参数化模型来学习这个基线。

-   **与 TD 学习的关系**: 评论家的学习过程本身就是一个标准的 TD 预测问题。它使用自举 (bootstrapping) 的方式来学习值函数，从而降低了方差。

-   **偏差-方差权衡**: AC 方法通过引入一个有偏但低方差的评论家（TD 误差）来代替无偏但高方差的蒙特卡罗回报，从而在偏差和方差之间做出了权衡。虽然 TD 误差是有偏的（因为它依赖于当前不完美的值函数估计 $V_w$），但这种偏差通常是可控的，并且所换来的方差降低往往能带来更快的整体收敛速度。

## 5. 参考文献 (References)

1.  Sutton, R. S., McAllester, D., Singh, S., & Mansour, Y. (2000). Policy gradient methods for reinforcement learning with function approximation. In *Advances in neural information processing systems*.
2.  Konda, V. R., & Tsitsiklis, J. N. (2000). Actor-critic algorithms. In *Advances in neural information processing systems*.
3.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Chapter 13)
