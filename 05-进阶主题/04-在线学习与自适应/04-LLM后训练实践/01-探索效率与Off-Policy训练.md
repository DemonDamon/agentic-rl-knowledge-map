# 1. 探索效率与 Off-Policy 训练

**Authors:** Damon Li

---

## 1.1. 探索效率的核心挑战

在大型语言模型 (LLM) 的强化学习 (RL) 后训练中，**探索效率**是决定训练成败的关键因素之一。这主要源于以下几个核心挑战：

1.  **高昂的采样成本**: LLM 的推理过程（即 rollout 或采样）是计算密集型操作。与传统 RL 任务相比，生成一个完整的轨迹（例如，一个多轮对话或一个解题步骤）需要消耗大量的计算资源和时间。

2.  **巨大的动作空间**: LLM 的动作空间是其整个词汇表，通常包含数万个 token。在每个生成步骤，模型都需要从这个巨大的离散空间中选择一个动作，这使得有效的探索变得极其困难。

3.  **环境交互延迟**: 在 Agent 类任务（如 Webshop, Mobile Agent）中，智能体需要与外部环境（真实的或仿真的）进行交互。这种交互通常伴随着高延迟，进一步降低了数据采样（rollout）的速度。

## 1.2. On-Policy vs. Off-Policy 训练

为了应对这些挑战，理解 On-Policy 和 Off-Policy 训练范式之间的权衡至关重要。

### 1.2.1. On-Policy 训练

-   **定义**: 在 On-Policy 方法中，用于更新策略的数据必须由当前策略本身生成。典型的例子是标准的 REINFORCE 和 A2C 算法。
-   **同步间隔 (Sync Interval)**: 在 LLM 训练中，这对应于 `Sync=1` 的情况，即每进行一次 rollout（探索一个 step），就立即用这批新数据训练模型一个 step。
-   **优点**: 理论简单，训练过程相对稳定，因为训练数据和当前策略的分布完全匹配。
-   **缺点**: **数据利用率极低**。由于 rollout 和训练步骤是串行的，GPU 在 rollout 期间处于空闲状态，在训练期间采样器又处于空闲状态，导致硬件利用率通常低于 50%。这在 LLM 的高昂训练成本下是难以接受的。

### 1.2.2. Off-Policy 训练

-   **定义**: Off-Policy 方法允许使用由旧策略（或行为策略）生成的数据来更新当前策略（目标策略）。典型的例子是 Q-Learning, DDPG, 以及带有重要性采样的 PPO/GRPO。
-   **同步间隔**: 对应于 `Sync > 1` 的情况。智能体可以连续进行多个 rollout steps，将数据收集到经验回放池 (Replay Buffer) 中，然后训练器从池中采样数据进行多次训练更新。
-   **优点**: **数据利用率高**。通过解耦采样和训练，可以实现异步执行，显著提高硬件利用率和训练吞吐量。
-   **缺点**: **训练不稳定性**。由于训练数据的分布（来自旧策略）与当前策略的分布不匹配，直接应用梯度更新会导致方差过大，甚至训练崩溃。因此，必须引入修正机制。

## 1.3. 重要性采样 (Importance Sampling)

**重要性采样**是解决 Off-Policy 分布不匹配问题的核心数学工具。

### 1.3.1. 理论基础

假设我们希望计算一个在目标分布 $p(x)$ 下的函数 $f(x)$ 的期望 $\mathbb{E}_{x \sim p}[f(x)]$，但我们只有从行为分布 $q(x)$ 中采样的样本 $x_i$。我们可以通过以下方式进行修正：

$$
\mathbb{E}_{x \sim p}[f(x)] = \int p(x) f(x) dx = \int q(x) \frac{p(x)}{q(x)} f(x) dx = \mathbb{E}_{x \sim q}\left[ \frac{p(x)}{q(x)} f(x) \right]
$$

其中，比率 $w(x) = \frac{p(x)}{q(x)}$ 被称为**重要性权重**。

### 1.3.2. 在策略梯度中的应用 (PPO/GRPO)

在策略梯度中，目标函数是 $J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}[R(\tau)]$。当使用由旧策略 $\pi_{\theta_{old}}$ 生成的数据时，我们需要对策略梯度进行修正。对于一个轨迹 $\tau = (s_0, a_0, \dots)$，其重要性权重为：

$$
w(\tau) = \frac{P(\tau | \pi_\theta)}{P(\tau | \pi_{\theta_{old}})} = \frac{\prod_t \pi_\theta(a_t|s_t)}{\prod_t \pi_{\theta_{old}}(a_t|s_t)} = \prod_t \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}
$$

令 $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$，则 PPO 的目标函数（Clipped Objective）为：

$$
L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min(r_t(\theta) A_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) A_t) \right]
$$

-   **$A_t$**: 在 $t$ 时刻的优势函数估计。
-   **$	ext{clip}(...)$**: 将重要性比率限制在一个小区间 $[1-\epsilon, 1+\epsilon]$ 内，通常 $\epsilon=0.2$。

**裁剪 (Clipping) 的作用**: 这是 PPO 稳定性的关键。当 $r_t(\theta)$ 过大或过小时，意味着新旧策略的偏差过大，直接使用会导致梯度更新步长过大，从而引发不稳定。通过裁剪，PPO 限制了单次更新的幅度，防止策略发生剧烈变化，从而在 Off-Policy 训练中保持了相对的稳定性。

### 1.3.3. 实践中的挑战

-   **Logprob 精度问题**: 在分布式训练中，用于 rollout 的推理引擎（如 vLLM, SGLang）和用于训练的框架（如 PyTorch + Huggingface）可能存在浮点精度差异或实现上的细微 bug。这会导致计算出的 $\pi_{\theta_{old}}(a_t|s_t)$ 不完全准确，使得即使在 `Sync=1` 的情况下，$r_t(\theta)$ 也不严格等于 1，导致不必要的裁剪和性能损失。
    -   **解决方案**: 一个实践技巧是在训练前，使用训练框架重新计算一遍 rollout 数据的 logprob，以保证一致性。

-   **上下文依赖问题**: 在通过“反思”等方式合成新数据时，生成的轨迹是在额外上下文（如反思文本）的条件下产生的。其 logprob $\pi(a|s, \text{context})$ 与无此上下文的 logprob $\pi(a|s)$ 是不一致的。直接将这些合成数据用于重要性采样在理论上是不严谨的。
    -   **解决方案**: 可以采用**上下文蒸馏 (Context Distillation)** 等技术，将额外上下文的信息“蒸馏”到不依赖该上下文的模型中，或者在实践中忽略这种不一致性（有时也能 work，但存在风险）。

## 1.4. 异步训练与优先级经验回放

为了最大化探索效率，可以采用完全异步的训练模式，即采样器和训练器完全解耦，独立运行。然而，这会加剧 Off-Policy 带来的数据陈旧性问题。

-   **优先级经验回放 (Prioritized Experience Replay, PER)**: 一种有效的缓解策略是使用优先级回放池。其核心思想是，并非所有经验都是同等重要的。例如，TD-error 较大的样本包含更多的学习信号。在 LLM 的 RL 训练中，可以设计不同的优先级标准：
    -   **新近度**: 给予新采集的样本更高的优先级，以保证训练器能及时看到最新的数据，减少策略偏差。
    -   **奖励/优势**: 给予奖励或优势值较高的样本更高的优先级，以聚焦于学习正向行为。
    -   **样本复杂度/稀有度**: 给予更复杂或更稀有的样本更高的优先级，以促进对困难和长尾场景的学习。

通过将 `Sync > 1` 的 Off-Policy 训练与优先级回放相结合，可以在探索效率和训练稳定性之间取得更好的平衡。

---

### 参考文献

1.  Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal Policy Optimization Algorithms. *arXiv preprint arXiv:1707.06347*.
2.  Schaul, T., Quan, J., Antonoglou, I., & Silver, D. (2015). Prioritized Experience Replay. *arXiv preprint arXiv:1511.05952*.
