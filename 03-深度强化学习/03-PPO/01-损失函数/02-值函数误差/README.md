---

# 3.3.1.2 PPO 损失函数: 值函数误差 (Value Function Error)

---

## 1. 背景：Actor-Critic 架构中的值函数 (Background: The Value Function in Actor-Critic)

PPO 算法通常在 Actor-Critic (AC) 架构下实现。在这个架构中，除了“演员 (Actor)”（策略网络 $\pi_\theta$）之外，还有一个“评论家 (Critic)”（值函数网络 $V_\phi(s)$）。

评论家的作用至关重要：

1.  **估计状态价值**: 评论家学习一个对当前策略下状态价值的估计，即 $V_\phi(s) \approx V^{\pi_\theta}(s)$。
2.  **计算优势函数**: 这个值函数估计被用来计算优势函数 $\hat{A}(s, a)$。一个准确的优势函数估计对于指导策略更新的方向和幅度至关重要。例如，在使用 GAE (Generalized Advantage Estimation) 时，我们需要 $V_\phi(s)$ 来计算 TD 残差 $\delta_t = r_t + \gamma V_\phi(s_{t+1}) - V_\phi(s_t)$。

因此，为了让演员能够有效地学习，我们必须同时训练一个准确的评论家。这就需要一个专门为评论家设计的损失函数。

## 2. 值函数损失的定义 (Definition of the Value Function Loss)

评论家的训练目标是使其输出 $V_\phi(s_t)$ 尽可能地接近该状态的“真实”价值。在 PPO 中，这个“真实”价值的目标通常是**蒙特卡罗回报 (Monte Carlo Return)** 或 **N-步回报**。

在实践中，当使用 GAE 计算优势函数时，我们已经计算了每一步的回报目标 $V_t^{targ}$。这个回报目标 $V_t^{targ}$ 正好可以作为训练值函数的目标。

回忆一下，GAE 的优势估计 $\hat{A}_t$ 和回报目标 $V_t^{targ}$ 的关系是：
$$ 
\hat{A}_t = V_t^{targ} - V_\phi(s_t) 
$$

因此，训练评论家的目标就是让 $V_\phi(s_t)$ 逼近 $V_t^{targ}$。这自然地导向了一个**均方误差 (Mean Squared Error, MSE)** 损失函数。

**值函数损失 (Value Function Loss):**
$$ 
L_t^{VF}(\phi) = (V_\phi(s_t) - V_t^{targ})^2 
$$

-   **$V_\phi(s_t)$**: 评论家网络对状态 $s_t$ 的价值预测。
-   **$V_t^{targ}$**: 状态 $s_t$ 的目标价值，通常是从经验轨迹中计算出的 N-步回报或 GAE 回报。

这个损失函数直观地惩罚了评论家的预测与从环境中实际观察到的回报之间的差异。

## 3. 在 PPO 总体损失中的作用 (Role in the Overall PPO Loss)

值函数损失是 PPO 总体损失函数的三个组成部分之一。完整的损失函数是策略损失、值函数损失和熵奖励的加权和：
$$ 
L_t(\theta, \phi) = \mathbb{E}_t [ -\mathcal{L}_t^{CLIP}(\theta) + c_1 L_t^{VF}(\phi) - c_2 S[\pi_\theta](s_t) ] 
$$

-   **$c_1$**: 这是一个超参数，用于控制值函数损失相对于策略损失的权重。通常，$c_1$ 的取值在 0.5 左右。这个权重很重要，因为它平衡了演员和评论家的学习速度。如果 $c_1$ 太大，学习可能会被值函数的拟合主导；如果太小，不准确的优势函数估计会损害策略的学习。

## 4. 参数共享 (Parameter Sharing)

在许多 PPO 的实现中，为了提高计算效率，演员（策略网络）和评论家（值函数网络）会共享一部分底层的网络参数。例如，在一个处理视觉输入的任务中，两个网络可能会共享一个共同的卷积神经网络 (CNN) 来提取特征，然后分别连接到各自的输出头（一个用于策略，一个用于值函数）。

```mermaid
graph TD
    Input[Input State s] --> SharedLayers[Shared Layers (e.g., CNN)]
    
    SharedLayers --> PolicyHead[Policy Head]
    SharedLayers --> ValueHead[Value Head]
    
    PolicyHead --> ActionProbs[Action Probabilities]
    ValueHead --> StateValue[State Value V(s)]
```

当使用参数共享时，这个统一的网络会同时根据策略损失和值函数损失进行优化。这种多任务学习的设置通常能带来好处，因为学习状态价值的表示可以帮助策略更好地理解环境，反之亦然。

## 5. 裁剪值函数损失 (Clipped Value Function Loss)

类似于对策略目标进行裁剪，一些 PPO 的实现也会对值函数损失进行裁剪。其思想是，如果值函数的预测变化过大，也可能导致训练不稳定。裁剪后的值函数损失可以写作：

1.  $V_{loss}^{unclipped} = (V_\phi(s_t) - V_t^{targ})^2$
2.  $V_\phi^{clipped}(s_t) = V_{\phi_{old}}(s_t) + \text{clip}(V_\phi(s_t) - V_{\phi_{old}}(s_t), -\epsilon_v, +\epsilon_v)$
3.  $V_{loss}^{clipped} = (V_\phi^{clipped}(s_t) - V_t^{targ})^2$
4.  $L_t^{VF}(\phi) = \max(V_{loss}^{unclipped}, V_{loss}^{clipped})$

其中 $\epsilon_v$ 是一个类似于策略裁剪的超参数。这种做法可以防止值函数在单次更新中变化过大，进一步增强稳定性。这在 OpenAI 的 PPO 实现中被采用。

## 6. 参考文献 (References)

1.  Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*.
2.  Schulman, J., Moritz, P., Levine, S., Jordan, M., & Abbeel, P. (2015). High-dimensional continuous control using generalized advantage estimation. *arXiv preprint arXiv:1506.02438*.
