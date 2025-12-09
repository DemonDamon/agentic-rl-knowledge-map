---

# 3.1.1 经验回放 (Experience Replay)

---

## 1. 概述 (Overview)

经验回放 (Experience Replay) 是深度 Q 网络 (DQN) [1] 及其众多变体中的一项核心技术。它的引入主要是为了解决在使用非线性函数逼近器（如神经网络）时，强化学习训练不稳定的两个关键问题：

1.  **样本相关性 (Correlated Samples)**: 在标准的在线 RL 中，智能体按顺序经历一个轨迹，导致训练样本 $(s_t, a_t, r_{t+1}, s_{t+1})$ 之间具有很强的时间相关性。这种相关性违反了许多监督学习算法所依赖的“独立同分布 (i.i.d.)”假设，会导致网络在局部数据上过拟合，从而产生不稳定的训练。

2.  **非平稳数据分布 (Non-stationary Data Distribution)**: 随着策略的改进，智能体访问的状态和动作的分布会发生变化。对于一个正在学习的目标网络来说，这意味着它所要拟合的目标函数的数据分布在不断变化，这给训练带来了困难。

经验回放通过创建一个**回放缓冲区 (Replay Buffer)** 来解决这些问题。这个缓冲区是一个存储了大量近期经验元组 $(s_t, a_t, r_{t+1}, s_{t+1})$ 的队列。

**核心思想:**

在每一步训练时，算法不再使用刚刚产生的单个经验样本来更新网络，而是从回放缓冲区中**随机采样一个小批量 (mini-batch)** 的经验来进行训练。这个简单的机制带来了几个关键优势：

-   **打破相关性**: 通过随机采样，一个 mini-batch 中的样本来自于智能体在不同时期的不同经验片段，从而打破了样本之间的时间相关性，使得更新更加接近于监督学习中的 i.i.d. 假设。
-   **数据重用，提高样本效率**: 每一个经验样本都可能被多次用于训练，这极大地提高了数据的利用率，对于样本获取成本高的环境（如真实机器人）尤为重要。
-   **平滑数据分布**: 通过对大量历史数据进行平均，训练时的数据分布变得更加平稳，有助于稳定学习过程。

## 2. 算法流程 (Algorithm Flow)

经验回放机制的集成非常直接：

1.  **初始化**: 创建一个固定大小的回放缓冲区 $D$（例如，容量为 $N$）。
2.  **存储经验**: 在每个时间步 $t$，智能体与环境交互后，将产生的经验元组 $e_t = (s_t, a_t, r_{t+1}, s_{t+1})$ 存入缓冲区 $D$ 中。如果缓冲区已满，通常会移除最旧的经验。
3.  **采样与训练**: 在每一步或每几步，从缓冲区 $D$ 中随机均匀地采样一个 mini-batch 的经验 $e_i = (s_i, a_i, r_{i+1}, s_{i+1})$，其中 $i$ 是随机索引。
4.  **计算损失**: 对于采样出的每一个经验，计算其 TD 目标和损失。例如，在 DQN 中，损失为：
    $$ 
    L_i(\theta_i) = \left( \underbrace{r_{i+1} + \gamma \max_{a'} Q_{\theta^-}(s_{i+1}, a')}_{\text{TD Target}} - Q_\theta(s_i, a_i) \right)^2 
    $$
5.  **梯度更新**: 对整个 mini-batch 的总损失执行一次梯度下降步骤，以更新网络参数 $\theta$。

## 3. 伪代码 (Pseudocode)

以下是带有经验回放的 DQN 算法的伪代码片段，重点突出了经验回放的部分。

```
Algorithm: Deep Q-Network with Experience Replay

Initialize replay memory D to capacity N
Initialize action-value function Q with random weights θ
Initialize target action-value function Q_hat with weights θ⁻ = θ

For episode = 1, M do
  Initialize sequence s₁ and preprocessed sequence φ₁ = φ(s₁)
  For t = 1, T do
    // With probability ε select a random action a_t
    // otherwise select a_t = max_a Q*(φ(s_t), a; θ)
    
    Execute action a_t in emulator and observe reward r_t and image x_{t+1}
    Set s_{t+1} = s_t, a_t, x_{t+1} and preprocess φ_{t+1} = φ(s_{t+1})
    
    // Store transition in D
    Store transition (φ_t, a_t, r_t, φ_{t+1}) in D
    
    // Sample random minibatch of transitions from D
    Sample random minibatch of transitions (φ_j, a_j, r_j, φ_{j+1}) from D
    
    // Set y_j (TD Target)
    Set y_j = r_j if episode terminates at step j+1
    else y_j = r_j + γ max_{a'} Q_hat(φ_{j+1}, a'; θ⁻)
    
    // Perform a gradient descent step
    Perform a gradient descent step on (y_j - Q(φ_j, a_j; θ))² with respect to the network parameters θ
    
    // Every C steps reset Q_hat = Q
  End For
End For
```

## 4. 改进与变体 (Improvements and Variants)

标准的均匀采样经验回放虽然有效，但并非最优。一个重要的改进是**优先经验回放 (Prioritized Experience Replay, PER)** [2]。

PER 的核心思想是：并非所有的经验都是同等重要的。那些 TD 误差很大的经验（即网络预测与目标差异很大的样本）包含更多的“意外”和学习价值。因此，我们应该更频繁地从这些“令人惊讶”的经验中学习。

PER 不再进行均匀采样，而是根据每个样本的 TD 误差的绝对值 $|\delta|$ 来赋予其一个采样优先级 $p \propto |\delta|^\omega + \epsilon$。为了处理由此引入的采样偏差，PER 还使用了**重要性采样 (Importance Sampling)** 权重来修正梯度更新。

## 5. 参考文献 (References)

1.  Mnih, V., Kavukcuoglu, K., Silver, D., Rusu, A. A., Veness, J., Bellemare, M. G., ... & Hassabis, D. (2015). Human-level control through deep reinforcement learning. *Nature, 518*(7540), 529-533.
2.  Schaul, T., Quan, J., Antonoglou, I., & Silver, D. (2015). Prioritized experience replay. *arXiv preprint arXiv:1511.05952*.
3.  Lin, L. J. (1992). Self-improving reactive agents based on reinforcement learning, planning and teaching. *Machine learning, 8*(3-4), 293-321. (This is one of the earliest works to propose the idea of experience replay).
