---

# 3.3.2.1 优势估计: GAE (Generalized Advantage Estimation)

---

## 1. 背景：优势函数在策略梯度中的作用 (Background: The Role of Advantage Function in Policy Gradients)

在策略梯度方法中，我们的目标是更新策略参数 $\theta$ 以增加高回报动作的概率。策略梯度的基本形式是：
$$ 
\nabla_\theta J(\theta) = \mathbb{E}_\tau [\sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t|s_t) \Psi_t] 
$$
其中，$\Psi_t$ 是一个用于评估在时间步 $t$ 的动作 $a_t$ 有多好的标量。对 $\Psi_t$ 的不同选择导致了不同的算法：

-   **REINFORCE**: 使用完整的蒙特卡罗回报 $G_t = \sum_{k=t}^{T} \gamma^{k-t} r_k$ 作为 $\Psi_t$。这是无偏的，但方差极高。
-   **Actor-Critic**: 引入一个基线 (baseline) $b(s_t)$（通常是值函数 $V(s_t)$）来降低方差。此时 $\Psi_t = G_t - V(s_t)$ 或 $\Psi_t = r_t + \gamma V(s_{t+1}) - V(s_t)$。后者被称为**优势函数 (Advantage Function)** $A(s, a) = Q(s, a) - V(s)$ 的单步估计。

使用优势函数 $A(s, a)$ 作为 $\Psi_t$ 是一个很好的选择，因为它衡量了采取动作 $a$ 相对于在该状态下遵循当前策略的平均表现要好多少。一个准确的优势函数估计对于稳定和高效的策略学习至关重要。

## 2. 优势函数的不同估计方法 (Different Estimators for the Advantage Function)

我们可以用不同步长的 TD 估计来计算优势函数：

-   **1-步优势 (TD(0) Advantage)**: 
    $\hat{A}_t^{(1)} = \delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$
    这个估计方差低，但由于依赖于不完美的 $V(s_{t+1})$，所以是有偏的。

-   **N-步优势**: 
    $\hat{A}_t^{(N)} = (\sum_{k=0}^{N-1} \gamma^k r_{t+k}) + \gamma^N V(s_{t+N}) - V(s_t)$
    这在偏差和方差之间提供了一个权衡。

-   **蒙特卡罗优势 (Full Return Advantage)**: 
    $\hat{A}_t^{(\infty)} = G_t - V(s_t) = (\sum_{k=0}^{\infty} \gamma^k r_{t+k}) - V(s_t)$
    这是无偏的，但方差非常高。

哪种估计是最好的呢？答案是“视情况而定”。一个好的估计方法应该能够在偏差和方差之间取得良好的平衡。

## 3. GAE 的思想 (The Idea of GAE)

**广义优势估计 (Generalized Advantage Estimation, GAE)** [1] 正是为了解决这个问题而提出的。GAE 的核心思想是，将所有不同步长的优势估计进行**指数加权平均**，从而在偏差和方差之间提供一个可调节的、平滑的权衡。

GAE 引入了两个参数：
-   **$\gamma \in [0, 1]$**: 折扣因子，用于对未来的奖励进行折现。
-   **$\lambda \in [0, 1]$**: GAE 的权衡参数。

GAE 的定义如下：
$$ 
\hat{A}_t^{GAE(\gamma, \lambda)} = \sum_{l=0}^{\infty} (\gamma\lambda)^l \delta_{t+l} 
$$
其中 $\delta_{t+l} = r_{t+l} + \gamma V(s_{t+l+1}) - V(s_{t+l})$ 是在时间步 $t+l$ 的单步 TD 残差。

**GAE 的两个极端情况:**

-   **当 $\lambda = 0$**: 
    $\hat{A}_t^{GAE(\gamma, 0)} = \delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$
    这退化为了方差最低、偏差最高的单步 TD 优势估计。

-   **当 $\lambda = 1$**: 
    $\hat{A}_t^{GAE(\gamma, 1)} = \sum_{l=0}^{\infty} \gamma^l \delta_{t+l} = (\sum_{k=t}^{\infty} \gamma^{k-t} r_k) - V(s_t)$
    这退化为了偏差最低（无偏）、方差最高的蒙特卡罗优势估计。（这个等式可以通过展开 $\delta$ 项并抵消 $V$ 项来证明）。

通过选择一个介于 0 和 1 之间的 $\lambda$ 值（例如，0.95 或 0.97），GAE 可以在偏差和方差之间取得一个很好的平衡，这在实践中通常能带来显著的性能提升和更稳定的学习过程。

## 4. 计算 GAE (Calculating GAE)

在实践中，我们从一个有限长度的轨迹 $(s_0, a_0, r_0, \dots, s_{T-1}, a_{T-1}, r_{T-1}, s_T)$ 中计算 GAE。计算通常从轨迹的末尾开始，向前递推。

假设我们有一个轨迹，长度为 $T$。在时间步 $t$ 的 GAE 优势 $\hat{A}_t$ 可以通过以下递推公式计算：
$$ 
\hat{A}_t = \delta_t + \gamma\lambda \hat{A}_{t+1} 
$$
其中 $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$。

**计算步骤:**

1.  **初始化**: 在轨迹的最后一步 $T-1$，我们有：
    $\delta_{T-1} = r_{T-1} + \gamma V(s_T) - V(s_{T-1})$
    $\hat{A}_{T-1} = \delta_{T-1}$ (因为 $\hat{A}_T = 0$)

2.  **向前递推**: 从 $t = T-2$ 到 $0$ 循环：
    $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$
    $\hat{A}_t = \delta_t + \gamma\lambda \hat{A}_{t+1}$

通过这种方式，我们可以在 $O(T)$ 的时间内高效地计算出整个轨迹中每一步的 GAE 值。

## 5. 在 PPO 中的应用 (Usage in PPO)

GAE 是 PPO 算法的一个标准组件。在 PPO 的每次迭代中：

1.  使用当前策略收集一批轨迹。
2.  使用评论家网络 $V_\phi$ 来计算每个时间步的 TD 残差 $\delta_t$。
3.  使用上述递推公式计算每个时间步的 GAE 优势 $\hat{A}_t$。
4.  这些计算出的 $\hat{A}_t$ 被用于构建 PPO 的 Clipped Surrogate Objective，以更新策略网络。
5.  同时，回报目标 $V_t^{targ} = \hat{A}_t + V_\phi(s_t)$ 被用于更新评论家网络。

GAE 提供了一个高质量的优势函数估计，这是 PPO 能够实现稳定和高效学习的关键因素之一。

## 6. 参考文献 (References)

1.  Schulman, J., Moritz, P., Levine, S., Jordan, M., & Abbeel, P. (2015). High-dimensional continuous control using generalized advantage estimation. *arXiv preprint arXiv:1506.02438*.
