---

# 3.3.1.1 PPO 损失函数: Clipped Surrogate Objective

---

## 1. 背景与动机 (Background and Motivation)

在策略梯度方法中，一个核心的挑战是如何在不破坏学习稳定性的前提下，选择一个合适的更新步长。TRPO (Trust Region Policy Optimization) 通过在策略更新上施加一个硬性的 KL 散度约束来解决这个问题，但这导致了算法实现上的复杂性。

PPO (Proximal Policy Optimization) 的目标是获得 TRPO 的稳定性和可靠性，但使用更简单的、一阶的优化方法。PPO-Clip 变体通过设计一个新颖的**替代目标函数 (Surrogate Objective Function)** 来实现这一目标，该函数通过**裁剪 (clipping)** 来隐式地限制策略更新的幅度。

这个裁剪后的替代目标函数是 PPO 算法最关键的创新。

## 2. 目标函数的推导 (Derivation of the Objective)

标准的策略梯度目标函数可以写作：
$$ 
L^{PG}(\theta) = \mathbb{E}_t [\log \pi_\theta(a_t|s_t) \hat{A}_t] 
$$

为了使用旧策略 $\pi_{\theta_{old}}$ 收集的数据来更新新策略 $\pi_\theta$，我们需要使用重要性采样 (Importance Sampling)。定义概率比 $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$，则目标函数变为：
$$ 
L^{IS}(\theta) = \mathbb{E}_t [r_t(\theta) \hat{A}_t] 
$$

然而，如果概率比 $r_t(\theta)$ 离 1 太远，这个估计的方差会非常大，导致更新不稳定。TRPO 通过约束 $D_{KL}(\pi_{\theta_{old}} || \pi_\theta) \leq \delta$ 来解决这个问题。

PPO-Clip 采用了不同的方法。它构建了一个“悲观”的（即下界的）目标函数，当策略变化过大时，这个目标函数会饱和，从而阻止策略更新得太远。

**PPO Clipped Surrogate Objective:**
$$ 
L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min(r_t(\theta) \hat{A}_t, \text{clip}(r_t(\theta), 1 - \epsilon, 1 + \epsilon) \hat{A}_t) \right] 
$$

-   **$r_t(\theta)$**: 概率比，衡量新旧策略的差异。
-   **$\hat{A}_t$**: 优势函数估计。
-   **$\epsilon$**: 裁剪范围的超参数 (e.g., 0.2)。
-   **$\text{clip}(r_t, 1-\epsilon, 1+\epsilon)$**: 将 $r_t$ 裁剪到 $[1-\epsilon, 1+\epsilon]$ 区间内。

## 3. 目标函数的工作原理 (How the Objective Works)

这个目标函数的行为取决于优势函数 $\hat{A}_t$ 的符号。

**Case 1: $\hat{A}_t > 0$ (这是一个比平均要好的动作)**

-   目标函数变为 $L = \min(r_t \hat{A}_t, (1 + \epsilon) \hat{A}_t)$。
-   为了最大化 $L$，我们会希望增加 $r_t$（即增加采取这个好动作的概率）。
-   但是，当 $r_t$ 增加到超过 $1+\epsilon$ 时，第一项 $r_t \hat{A}_t$ 会继续增长，但第二项 $(1+\epsilon)\hat{A}_t$ 是一个常数。由于 $\min$ 操作，整个目标函数的值将被“钳位”在 $(1+\epsilon)\hat{A}_t$，不再增加。
-   **效果**: 这阻止了策略为了一个有利的动作而进行过大的更新。一旦策略变化超出了由 $\epsilon$ 定义的“信赖域”，进一步的更新将被忽略。

**Case 2: $\hat{A}_t < 0$ (这是一个比平均要差的动作)**

-   目标函数变为 $L = \min(r_t \hat{A}_t, (1 - \epsilon) \hat{A}_t)$。
-   因为 $\hat{A}_t$ 是负数，所以 $\min$ 操作实际上是在取两个负值中更小（绝对值更大）的那个，这等价于 $\max(r_t \hat{A}_t, (1 - \epsilon) \hat{A}_t)$ (在梯度上升的语境下)。
-   为了最大化 $L$（即让负值更接近于0），我们会希望减小 $r_t$（即降低采取这个坏动作的概率）。
-   但是，当 $r_t$ 减小到低于 $1-\epsilon$ 时，第一项 $r_t \hat{A}_t$ 会变得更负，但第二项 $(1-\epsilon)\hat{A}_t$ 是一个固定的负值。由于 $\max$ 操作，整个目标函数的值将被“钳位”在 $(1-\epsilon)\hat{A}_t$，不再减小（即不再变得更负）。
-   **效果**: 这阻止了策略因为一个大的负优势而过度地惩罚某个动作，同样限制了策略的更新幅度。

**图示:**

![PPO-Clip Objective Function](https://spinningup.openai.com/en/latest/_images/ppo-clip.svg)
(Image credit: OpenAI Spinning Up)

上图清晰地展示了当优势为正（左）和为负（右）时，目标函数是如何被裁剪的。

## 4. 总结 (Summary)

PPO 的 Clipped Surrogate Objective 是一种非常巧妙的设计，它通过一个简单的、可微的目标函数，有效地实现了 TRPO 的信赖域思想，而无需进行复杂的二阶优化。

-   它通过**裁剪概率比**来限制策略更新的幅度。
-   它构建了一个**悲观的下界**，忽略了那些可能导致策略性能剧烈变化的更新。
-   它使得 PPO 可以使用简单的**一阶优化器**（如 Adam）进行稳定高效的训练。

这种简单性和有效性的结合，使得 PPO-Clip 成为当前深度强化学习中最流行和最可靠的算法之一。

## 5. 参考文献 (References)

1.  Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*.
