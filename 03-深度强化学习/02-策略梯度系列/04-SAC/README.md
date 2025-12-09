---

# 3.2.4 SAC: 软 Actor-Critic (Soft Actor-Critic)

---

## 1. 概述 (Overview)

软 Actor-Critic (Soft Actor-Critic, SAC) [1, 2] 是一种为连续动作空间设计的、基于最大熵强化学习框架的离策略 (off-policy) Actor-Critic 算法。SAC 以其出色的**样本效率**和**稳定性**而闻名，被认为是当前最先进的无模型深度强化学习算法之一。

传统强化学习的目标是最大化累积奖励的期望值。而**最大熵强化学习 (Maximum Entropy RL)** 的目标则是在最大化累积奖励的同时，也最大化策略的**熵 (entropy)**。

**最大熵目标函数:**
$$ 
J(\pi) = \sum_{t=0}^{T} \mathbb{E}_{(s_t, a_t) \sim \rho_\pi} \left[ r(s_t, a_t) + \alpha H(\pi(\cdot|s_t)) \right] 
$$

-   $H(\pi(\cdot|s_t))$ 是策略 $\pi$ 在状态 $s_t$ 的熵。
-   $\alpha > 0$ 是一个**温度参数 (temperature parameter)**，它控制了熵项相对于奖励的重要性。$\alpha$ 越大，算法越倾向于探索和采取更随机的动作。

这种目标函数带来了两个主要好处：
1.  **鼓励探索**: 通过最大化熵，算法会激励策略探索更多样的动作，避免过早地收敛到局部最优。这使得 SAC 在具有复杂奖励结构或需要大量探索的任务中表现出色。
2.  **提高鲁棒性**: 学习到的策略更加随机，对环境的微小变化或扰动具有更好的鲁棒性。

## 2. 核心思想与组件 (Core Ideas and Components)

SAC 建立在 Actor-Critic 架构之上，并引入了几个关键的创新来适应最大熵目标。

**1. 软 Q 函数 (Soft Q-Function):**
在最大熵框架下，值函数的定义也需要包含熵项。软 Q 函数 $Q(s, a)$ 的贝尔曼方程变为：
$$ 
Q(s_t, a_t) = r(s_t, a_t) + \gamma \mathbb{E}_{s_{t+1}} [V(s_{t+1})] 
$$
其中，软状态值函数 $V(s)$ 定义为：
$$ 
V(s) = \mathbb{E}_{a \sim \pi} [Q(s, a) - \alpha \log \pi(a|s)] 
$$
将 $V(s)$ 代入 $Q(s, a)$ 的方程，我们得到**软贝尔曼方程**：
$$ 
Q(s_t, a_t) = r(s_t, a_t) + \gamma \mathbb{E}_{(s_{t+1}, a_{t+1}) \sim \rho_\pi} [Q(s_{t+1}, a_{t+1}) - \alpha \log \pi(a_{t+1}|s_{t+1})] 
$$

**2. Clipped Double Q-Learning (双 Q 网络裁剪):**
为了解决 Q 值高估问题并稳定训练，SAC 采用了与 TD3 类似的技术。它学习**两个**独立的软 Q 函数网络 ($Q_{\phi_1}, Q_{\phi_2}$)，并在计算 TD 目标时，取两者中的**较小值**。
$$ 
Y_t = r(s_t, a_t) + \gamma (\min_{i=1,2} Q_{\phi_{i, \text{targ}}}(s_{t+1}, a_{t+1}) - \alpha \log \pi_\theta(a_{t+1}|s_{t+1})) 
$$
其中 $a_{t+1}$ 是根据当前策略 $\pi_\theta$ 从 $s_{t+1}$ 采样得到的动作。

**3. 随机策略 (Stochastic Policy):**
与 DDPG 和 TD3 不同，SAC 的演员是一个**随机策略** $\pi_\theta(a|s)$，它输出一个动作的概率分布（通常是高斯分布），而不是一个确定的动作。这对于最大化熵是至关重要的。策略的更新目标是最小化 KL 散度，使其逼近 Q 函数的指数形式，可以推导出如下的策略损失：
$$ 
L(\theta) = \mathbb{E}_{s_t \sim D, \epsilon_t \sim N} [\alpha \log \pi_\theta(f_\theta(s_t, \epsilon_t)|s_t) - Q_{\phi_1}(s_t, f_\theta(s_t, \epsilon_t))] 
$$
其中 $f_\theta$ 是一个可重参数化的采样过程（例如，对于高斯策略，$a = \mu_\theta(s) + \sigma_\theta(s) \cdot \epsilon$）。

**4. 自动调整温度参数 $\alpha$ (Automated Temperature Tuning):**
温度参数 $\alpha$ 对 SAC 的性能至关重要，但手动调整很困难。SAC 的一个重要创新是**自动调整** $\alpha$。这是通过构建一个关于 $\alpha$ 的优化问题来实现的，其目标是使策略的平均熵保持在一个预设的目标熵 $\mathcal{H}$ 以上。
$$ 
J(\alpha) = \mathbb{E}_{a_t \sim \pi_t} [-\alpha \log \pi_t(a_t|s_t) - \alpha \mathcal{H}] 
$$
通过对 $\alpha$ 进行梯度下降，可以自动地找到合适的温度值。

## 3. 算法伪代码 (Pseudocode)

```
Algorithm: Soft Actor-Critic

Initialize policy parameters θ, Q-function parameters φ₁, φ₂, and target network parameters φ_targ1, φ_targ2.
Initialize temperature α and replay buffer D.

For each iteration do:
  For each environment step do:
    a_t ~ π_θ(·|s_t)
    s_{t+1} ~ p(·|s_t, a_t)
    D ← D ∪ {(s_t, a_t, r_t, s_{t+1})}
  End for

  For each gradient step do:
    s, a, r, s' ~ D (sample a minibatch)

    // Update Q-functions (critics)
    y = r + γ (min_{i=1,2} Q_{φ_targ,i}(s', a') - α log π_θ(a'|s')) where a' ~ π_θ(·|s')
    L_Q = (1/2) Σ_{i=1,2} (Q_{φ_i}(s, a) - y)²
    Update φ₁, φ₂ by one step of gradient descent on L_Q.

    // Update policy (actor)
    a_new ~ π_θ(·|s)
    L_π = E [α log π_θ(a_new|s) - Q_{φ₁}(s, a_new)]
    Update θ by one step of gradient ascent on L_π.

    // (Optional) Update temperature α
    L_α = E [-α log π_θ(a|s) - α * target_entropy]
    Update α by one step of gradient descent on L_α.

    // Update target networks
    φ_targ,i ← τφ_i + (1-τ)φ_targ,i for i=1,2
  End for
End for
```

## 4. 总结 (Summary)

SAC 通过将最大熵框架与先进的 Actor-Critic 技术（如双 Q 网络、软更新）相结合，取得了卓越的性能。

**优点:**
-   **极高的样本效率**: 通常比其他离策略和在策略算法学习得更快。
-   **非常稳定**: 对超参数的敏感性较低，训练过程稳定，不易发散。
-   **强大的探索能力**: 内置的熵最大化机制使其能够有效地探索复杂的环境。

**缺点:**
-   **实现相对复杂**: 相比 PPO 等算法，SAC 涉及到更多的组件（两个 Q 网络，策略网络，以及可选的温度参数优化），实现起来更复杂。

由于其强大的性能和稳定性，SAC 已成为连续控制任务中的首选算法之一。

## 5. 参考文献 (References)

1.  Haarnoja, T., Zhou, A., Abbeel, P., & Levine, S. (2018, July). Soft actor-critic: Off-policy maximum entropy deep reinforcement learning with a stochastic actor. In *International conference on machine learning* (pp. 1861-1870).
2.  Haarnoja, T., Zhou, A., Hartikainen, K., Tucker, G., Ha, S., Tan, J., ... & Levine, S. (2018). Soft actor-critic algorithms and applications. *arXiv preprint arXiv:1812.05905*.
