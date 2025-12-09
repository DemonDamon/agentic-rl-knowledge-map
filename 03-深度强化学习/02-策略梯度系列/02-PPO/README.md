---

# 3.2.2 PPO: 近端策略优化 (Proximal Policy Optimization)

---

## 1. 概述 (Overview)

近端策略优化 (Proximal Policy Optimization, PPO) [1] 是由 OpenAI 在2017年提出的一种策略梯度算法。它的目标是实现与 TRPO 类似的学习稳定性和可靠性，但使用更简单的实现方式，从而在易用性、性能和样本复杂度之间取得了出色的平衡。

PPO 已经成为深度强化学习领域的“默认”算法之一，因其强大的性能和相对简单的实现而被广泛应用。

TRPO 通过一个复杂的二阶优化方法（共轭梯度）来求解一个带 KL 散度约束的优化问题。PPO 的核心思想是，通过修改目标函数，将这个 KL 散度约束转化为一个惩罚项或一个“裁剪”操作，从而使得算法可以使用简单的一阶优化方法（如 Adam）进行训练。

PPO 主要有两种变体：

1.  **PPO-Penalty**: 将 KL 散度作为一个惩罚项加入到目标函数中。
2.  **PPO-Clip**: 使用一个裁剪的目标函数 (Clipped Objective) 来限制策略更新的幅度。这是最常用和最著名的版本。

本节将主要关注 **PPO-Clip**。

## 2. PPO-Clip 的核心思想 (The Core Idea of PPO-Clip)

PPO-Clip 的目标函数旨在通过一个简单的机制来限制新旧策略之间的差异。

首先，定义概率比 (probability ratio) $r_t(\theta)$：
$$ 
r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)} 
$$
这个比值衡量了新策略 $\pi_\theta$ 和旧策略 $\pi_{\theta_{old}}$ 在特定状态-动作对上的概率差异。$r_t(\theta) > 1$ 表示新策略更倾向于采取动作 $a_t$，$r_t(\theta) < 1$ 则表示更不倾向。

标准策略梯度的目标函数是 $\mathcal{L}^{PG}(\theta) = \mathbb{E}_t [r_t(\theta) \hat{A}_t]$。PPO-Clip 对此进行了修改，提出了一个新的目标函数 $\mathcal{L}^{CLIP}(\theta)$：
$$ 
\mathcal{L}^{CLIP}(\theta) = \mathbb{E}_t \left[ \min(r_t(\theta) \hat{A}_t, \text{clip}(r_t(\theta), 1 - \epsilon, 1 + \epsilon) \hat{A}_t) \right] 
$$

让我们来分解这个复杂的目标函数：

-   **$\hat{A}_t$**: 这是在时间步 $t$ 的优势函数估计 (e.g., using GAE)。
-   **$\epsilon$**: 这是一个小的超参数（例如，0.2），它定义了裁剪的范围。
-   **$\text{clip}(r_t(\theta), 1 - \epsilon, 1 + \epsilon)$**: 这个函数将概率比 $r_t(\theta)$ 限制在区间 $[1 - \epsilon, 1 + \epsilon]$ 内。
-   **$\min(\dots, \dots)$**: 这是整个目标函数的关键。它取两个项中的较小值。

**这个目标函数是如何工作的？**

我们分两种情况讨论，取决于优势函数 $\hat{A}_t$ 的符号：

1.  **当 $\hat{A}_t > 0$ (这是一个好动作)**:
    -   目标函数变为 $\mathcal{L} = \min(r_t \hat{A}_t, (1 + \epsilon) \hat{A}_t)$。
    -   为了最大化这个目标，我们会希望增加 $r_t$（即增加 $\pi_\theta(a_t|s_t)$）。
    -   但是，由于 $\min$ 操作的存在，当 $r_t$ 增长到超过 $1 + \epsilon$ 时，目标函数的值将不再增加，被“裁剪”在 $(1 + \epsilon) \hat{A}_t$。这有效地阻止了策略为了利用一个好的优势而更新得过远。

2.  **当 $\hat{A}_t < 0$ (这是一个坏动作)**:
    -   目标函数变为 $\mathcal{L} = \max(r_t \hat{A}_t, (1 - \epsilon) \hat{A}_t)$ (因为 $\hat{A}_t$ 是负数，所以 $\min$ 相当于 $\max$)。
    -   为了最大化这个目标（即让负值更接近0），我们会希望减小 $r_t$（即减小 $\pi_\theta(a_t|s_t)$）。
    -   但是，当 $r_t$ 减小到低于 $1 - \epsilon$ 时，目标函数的值将不再变化，被“裁剪”在 $(1 - \epsilon) \hat{A}_t$。这防止了策略因为一个大的负优势而过度惩罚某个动作。

**图示:**

![PPO-Clip Objective](https://lilianweng.github.io/lil-log/assets/images/ppo-objective.png)
(Image credit: Lilian Weng)

通过这种方式，PPO-Clip 创造了一个“悲观”的目标函数下界，它忽略了那些会导致策略变化过大的更新，从而将新策略维持在旧策略的一个信赖域内。

## 3. 完整的 PPO 损失函数 (The Full PPO Loss)

PPO 的最终损失函数通常还包含另外两项：

1.  **值函数损失 (Value Function Loss)**: PPO 通常与 Actor-Critic 架构一起使用，所以需要一个损失项来训练评论家（值函数 $V_\phi(s)$）。这通常是一个简单的均方误差损失：
    $$ 
    L_t^{VF}(\phi) = (V_\phi(s_t) - V_t^{targ})^2 
    $$
    其中 $V_t^{targ}$ 是回报的目标（例如，蒙特卡罗回报或 GAE 计算出的值）。

2.  **熵正则化 (Entropy Bonus)**: 为了鼓励探索，通常会加上一个熵项 $S[\pi_\theta](s_t)$，以防止策略过早地收敛到次优的确定性策略。

最终的损失函数是这三项的加权和：
$$ 
L_t(\theta, \phi) = \mathbb{E}_t [ -\mathcal{L}_t^{CLIP}(\theta) + c_1 L_t^{VF}(\phi) - c_2 S[\pi_\theta](s_t) ] 
$$
注意，策略目标 $\mathcal{L}^{CLIP}$ 是要最大化的，所以在梯度下降中我们取其负值。

## 4. 算法伪代码 (Pseudocode)

```
Algorithm: PPO-Clip

Initialize policy parameters θ and value function parameters φ

For k = 0, 1, 2, ... do
  1. Collect a set of trajectories D_k by executing policy π_{θ_old} for T timesteps.
  2. Estimate advantages Â_t and value targets V_t^targ using GAE.
  3. For epoch = 1, ..., K do
     For each minibatch in D_k:
       Compute the PPO loss L(θ, φ) for the minibatch.
       Update θ and φ by taking a gradient step on L(θ, φ) (e.g., using Adam).
  4. Set θ_old ← θ
End For
```
一个关键点是，PPO 会使用同一批数据进行多次（K个 epoch）的梯度更新，这提高了样本的利用率。

## 5. 优缺点 (Pros and Cons)

**优点:**
-   **实现简单**: 相比 TRPO，PPO 的实现要简单得多，它只需要标准的梯度下降优化器。
-   **性能强大**: 在许多基准测试中，PPO 的性能与 TRPO 相当，甚至更好。
-   **样本效率高**: 通过在同一批数据上进行多次更新，PPO 具有良好的样本效率。

**缺点:**
-   **对超参数敏感**: PPO 的性能可能对裁剪范围 $\epsilon$ 和其他超参数的选择比较敏感。
-   **理论保证较弱**: 与 TRPO 不同，PPO 没有严格的单调改进理论保证，尽管在实践中它通常表现得非常稳定。

## 6. 参考文献 (References)

1.  Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*.
