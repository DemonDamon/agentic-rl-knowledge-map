---

# 3.1.2 目标网络 (Target Network)

---

## 1. 问题：移动的目标 (The Problem: A Moving Target)

在 Q-Learning 和 DQN 中，我们使用时间差分 (TD) 学习来更新我们的 Q 值估计。更新的目标（TD 目标）依赖于对下一状态的 Q 值估计。在标准的 Q-Learning 中，这个更新规则是：
$$ 
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha [\underbrace{R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a)}_{\text{TD Target}} - Q(S_t, A_t)] 
$$

当我们将这个思想应用到深度学习，使用一个神经网络 $Q_\theta$ 来参数化 Q 函数时，问题就出现了。损失函数变为：
$$ 
L(\theta) = \mathbb{E} \left[ \left( \underbrace{R_{t+1} + \gamma \max_{a} Q_\theta(S_{t+1}, a)}_{\text{TD Target}} - Q_\theta(S_t, A_t) \right)^2 \right] 
$$

注意到，在这个损失函数中，我们用来计算 TD 目标的网络 $Q_\theta$ 和我们正在更新的网络 $Q_\theta$ 是**同一个网络**。这意味着，在每一步梯度下降更新后，参数 $\theta$ 都会发生变化。这导致我们用于计算损失的 TD 目标值本身也在**不断地移动**。

想象一下，你正在试图射击一个目标，但这个目标在你每次瞄准和射击后都会移动一下。这会让你的学习过程变得非常困难和不稳定。在神经网络的训练中，这种“追逐移动目标”的问题会导致训练的剧烈振荡，甚至发散。

## 2. 目标网络的思想 (The Idea of a Target Network)

为了解决这个问题，DQN 的作者引入了**目标网络 (Target Network)** 的概念 [1]。其核心思想非常简单：我们使用**两个**独立的神经网络。

1.  **在线网络 (Online Network) $Q_\theta$**: 这是我们主要训练和更新的网络。它被用来在每一步选择动作（例如，通过 ε-贪婪策略），并且它的参数 $\theta$ 在每次训练迭代中都会通过梯度下降进行更新。

2.  **目标网络 (Target Network) $Q_{\theta^-}$**: 这是一个与在线网络结构完全相同的网络，但它的参数 $\theta^-$ 是“冻结”的。这个网络专门用于**计算 TD 目标**。它的参数不会在每次训练迭代中更新。

通过引入目标网络，DQN 的损失函数被修改为：
$$ 
L(\theta) = \mathbb{E} \left[ \left( \underbrace{R_{t+1} + \gamma \max_{a} Q_{\theta^-}(S_{t+1}, a)}_{\text{TD Target}} - Q_\theta(S_t, A_t) \right)^2 \right] 
$$

现在，在一次 mini-batch 的训练中，TD 目标是由一个固定的、参数为 $\theta^-$ 的网络计算出来的。这个目标在整个 mini-batch 的梯度计算和更新过程中是**稳定不变的**。这就像射击一个静止的目标，大大降低了训练的难度，提高了稳定性。

## 3. 更新目标网络 (Updating the Target Network)

当然，目标网络不能永远保持不变，否则它很快就会与不断学习的在线网络脱节，提供一个非常差的价值估计。因此，目标网络的参数需要被定期更新。

有两种主要的更新方式：

1.  **硬更新 (Hard Update)**: 这是原始 DQN 论文中使用的方法。每隔固定的 $C$ 个训练步骤，将在线网络的权重**完全复制**到目标网络中。
    $$ 
    \theta^- \leftarrow \theta \quad (\text{every C steps}) 
    $$
    在两次更新之间，目标网络的参数保持完全不变。

2.  **软更新 (Soft Update)**: 这种方法由 DDPG [2] 推广，它在每个训练步骤后都对目标网络进行一次平滑的、渐进式的更新。这通常通过一个加权平均来实现：
    $$ 
    \theta^- \leftarrow \tau \theta + (1 - \tau) \theta^- 
    $$
    其中 $\tau$ 是一个很小的常数（例如，$\tau = 0.001$），被称为“软更新系数”或“混合因子”。这种方式使得目标网络的更新更加平滑，有助于进一步提高训练的稳定性。

## 4. 总结 (Summary)

目标网络是稳定深度 Q 学习的关键技术之一，它与经验回放一起构成了 DQN 成功的两大支柱。

-   **作用**: 通过固定用于计算 TD 目标的网络，解决了“移动目标”问题，从而稳定了训练过程。
-   **机制**: 使用一个独立的、参数被周期性或渐进式更新的目标网络来计算 TD 目标。
-   **影响**: 显著降低了训练的振荡，使得使用强大的非线性函数逼近器（如深度神经网络）进行稳定的 Q 学习成为可能。

这项技术已被广泛应用于各种基于值函数的深度强化学习算法中，如 Double DQN, Dueling DQN, DDPG, TD3, SAC 等。

## 5. 参考文献 (References)

1.  Mnih, V., Kavukcuoglu, K., Silver, D., Rusu, A. A., Veness, J., Bellemare, M. G., ... & Hassabis, D. (2015). Human-level control through deep reinforcement learning. *Nature, 518*(7540), 529-533.
2.  Lillicrap, T. P., Hunt, J. J., Pritzel, A., Heess, N., Erez, T., Tassa, Y., ... & Wierstra, D. (2015). Continuous control with deep reinforcement learning. *arXiv preprint arXiv:1509.02971*.
