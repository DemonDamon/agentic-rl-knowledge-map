# 2. 策略熵动态学：熵变与协方差的数学关联

**Authors:** Damon Li

---

在强化学习训练过程中，策略参数 $\theta$ 不断更新，导致策略 $\pi_\theta$ 发生变化，其熵也随之改变。理解策略熵的变化规律，尤其是它为何在 LLM 的 RL 训练中倾向于单调递减，是解决“熵崩溃”问题的关键。本节将深入推导策略熵变化与特定协方差项之间的数学关系。

## 2.1. Softmax 策略的熵变引理

我们首先分析对于一个采用 Softmax 函数的策略，当其参数发生微小变化时，策略熵会如何变化。

### 2.1.1. Softmax 策略定义

一个典型的 Softmax 策略将模型的 logits 输出 $z_\theta(a|s)$ 转换为动作概率：

$$
\pi_\theta(a|s) = \frac{e^{z_\theta(a|s)}}{\sum_{a' \in \mathcal{A}} e^{z_\theta(a'|s)}}
$$

### 2.1.2. 熵变的一阶泰勒近似

假设策略参数从 $\theta$ 更新为 $\theta' = \theta + \Delta\theta$。我们希望分析策略熵的变化量 $\Delta H = H(\pi_{\theta'}) - H(\pi_\theta)$。我们可以将 $H(\pi_{\theta'})$ 在 $\theta$ 处进行一阶泰勒展开：

$$
H(\pi_{\theta'}) \approx H(\pi_\theta) + (\theta' - \theta)^T \nabla_\theta H(\pi_\theta)
$$

因此，$\Delta H \approx \Delta\theta^T \nabla_\theta H(\pi_\theta)$。我们现在需要计算熵的梯度 $\nabla_\theta H(\pi_\theta)$。

为简化符号，我们省略对状态 $s$ 的依赖，即 $\pi(a) = \pi(a|s)$。

$$
\nabla_\theta H(\pi) = \nabla_\theta \left( -\sum_a \pi(a) \log \pi(a) \right)
= -\sum_a \left( \nabla_\theta \pi(a) \log \pi(a) + \pi(a) \frac{1}{\pi(a)} \nabla_\theta \pi(a) \right)
= -\sum_a \nabla_\theta \pi(a) (\log \pi(a) + 1)
$$

利用 $\sum_a \pi(a) = 1$，我们有 $\sum_a \nabla_\theta \pi(a) = \nabla_\theta \sum_a \pi(a) = 0$。因此，上式可以简化为：

$$
\nabla_\theta H(\pi) = -\sum_a (\nabla_\theta \pi(a)) \log \pi(a)
$$

现在，我们引入 Softmax 策略的梯度性质。我们知道 $\nabla_\theta \log \pi(a) = \nabla_\theta z(a) - \mathbb{E}_{a' \sim \pi}[\nabla_\theta z(a')]$。因此，$\nabla_\theta \pi(a) = \pi(a) \nabla_\theta \log \pi(a) = \pi(a) (\nabla_\theta z(a) - \mathbb{E}_{a'}[\nabla_\theta z(a')])$。

代入熵梯度公式：

$$
\nabla_\theta H(\pi) = -\sum_a \pi(a) (\nabla_\theta z(a) - \mathbb{E}_{a'}[\nabla_\theta z(a')]) \log \pi(a)
$$

这正是协方差的定义形式。回忆一下，两个随机变量 $X, Y$ 的协方差为 $\text{Cov}(X, Y) = \mathbb{E}[XY] - \mathbb{E}[X]\mathbb{E}[Y]$。如果我们在动作分布 $a \sim \pi$ 下计算期望，令 $X = \log \pi(A)$，$Y = \nabla_\theta z(A)$，那么：

$$
\mathbb{E}_{a \sim \pi}[(\log \pi(a)) \nabla_\theta z(a)] - \mathbb{E}_{a \sim \pi}[\log \pi(a)] \mathbb{E}_{a \sim \pi}[\nabla_\theta z(a)]
= \sum_a \pi(a) (\log \pi(a)) (\nabla_\theta z(a)) - (\sum_a \pi(a) \log \pi(a)) (\sum_a \pi(a) \nabla_\theta z(a))
$$

我们的熵梯度公式可以重写为：

$$
\nabla_\theta H(\pi) = -\left( \sum_a \pi(a) (\log \pi(a)) (\nabla_\theta z(a)) - (\sum_a \pi(a) \log \pi(a)) (\mathbb{E}_{a'}[\nabla_\theta z(a')]) \right)
$$

这与协方差的定义非常接近。事实上，论文 [1] 证明了一个更直接的近似关系。当参数变化很小时，$\Delta z(a) = z_{\theta'}(a) - z_\theta(a) \approx \Delta\theta^T \nabla_\theta z(a)$。可以证明熵变近似为：

> **引理 (熵变近似)**: 对于 Softmax 策略，当参数更新 $\theta \to \theta'$ 足够小时，策略熵的变化量 $\Delta H$ 可以近似为：
> $$
> \Delta H \approx -\text{Cov}_{a \sim \pi_\theta} (\log \pi_\theta(a|s), \Delta z(a|s))
> $$
> 其中 $\Delta z(a|s) = z_{\theta'}(a|s) - z_\theta(a|s)$ 是 logits 的变化量，协方差在当前策略 $\pi_\theta$ 的动作分布下计算。

**直观解释**: 这个引理是理解熵动态学的基石。它表明，熵的变化方向和幅度，取决于“当前动作的对数概率”与“该动作 logits 的变化量”之间的相关性。

-   **正协方差**: 如果高概率的动作（$\log \pi$ 较大）其 logits 也倾向于增加（$\Delta z > 0$），而低概率动作的 logits 倾向于减小，那么协方差为正。这会导致分布变得更“尖锐”，确定性增强，因此熵下降（$\Delta H < 0$）。
-   **负协方差**: 如果高概率动作的 logits 减小，或低概率动作的 logits 增加，协方差为负。这会使分布变得更“平坦”，不确定性增加，因此熵增加（$\Delta H > 0$）。

## 2.2. 策略梯度下的 Logits 变化

现在，我们需要确定在标准的策略梯度 (PG) 算法下，logits 的变化量 $\Delta z(a)$ 是由什么决定的。

### 2.2.1. 策略梯度定理

策略梯度算法旨在最大化期望回报 $J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}[G_t]$。根据策略梯度定理，目标函数的梯度为：

$$
\nabla_\theta J(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} [A^\pi(s, a) \nabla_\theta \log \pi_\theta(a|s)]
$$

其中 $A^\pi(s, a) = Q^\pi(s, a) - V^\pi(s)$ 是优势函数。

### 2.2.2. 参数更新与 Logits 变化

在最简单的梯度上升更新中，参数更新规则为 $\theta' = \theta + \eta \nabla_\theta J(\theta)$，其中 $\eta$ 是学习率。因此，logits 的变化量为：

$$
\Delta z(a|s) = z_{\theta'}(a|s) - z_\theta(a|s) \approx (\theta' - \theta)^T \nabla_\theta z_\theta(a|s) = \eta (\nabla_\theta J(\theta))^T \nabla_\theta z_\theta(a|s)
$$

在自然策略梯度 (NPG) 中，更新规则变为 $\theta' = \theta + \eta F^{-1} \nabla_\theta J(\theta)$，其中 $F$ 是费雪信息矩阵 $F = \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta \nabla_\theta \log \pi_\theta^T]$。对于 Softmax 策略，可以证明 $F^{-1} \nabla_\theta \log \pi_\theta(a|s)$ 近似地与优势函数 $A(s,a)$ 相关。更直接地，在 NPG 的函数空间视角下，策略更新等价于将新的 Q 函数投影到策略空间，其 logits 更新近似满足：

$$
\Delta z(a|s) \propto A^\pi(s, a)
$$

这意味着，一个动作的 logits 会朝着其优势函数值的方向进行调整。优势为正的动作，其 logits 会增加；优势为负的动作，其 logits 会减小。这正是强化学习“试错”机制的体现。

## 2.3. 熵变的最终驱动力：协方差的根源

将第 2.2 节的结论（$\Delta z \propto A$）代入第 2.1 节的熵变引理，我们得到了论文中最核心的理论发现：

> **定理 (熵变驱动力)**: 在基于（自然）策略梯度的强化学习中，策略熵的变化量 $\Delta H$ 由动作的对数概率与其优势函数之间的协方差驱动：
> $$
> \Delta H \propto -\text{Cov}_{a \sim \pi_\theta} (\log \pi_\theta(a|s), A^\pi(s, a))
> $$

**证明概要**: 
1. 从熵变引理出发: $\Delta H \approx -\text{Cov}(\log \pi, \Delta z)$。
2. 在 NPG 等算法下，logits 的变化方向与优势函数一致: $\Delta z \propto A$。
3. 将 (2) 代入 (1)，并利用协方差的性质 $\text{Cov}(X, cY) = c \cdot \text{Cov}(X, Y)$ (其中 c 是标量)，即可得到该定理。

### 2.3.1. “熵崩溃”的理论解释

这个定理完美地解释了为何在 LLM 的 RL 训练中会发生熵崩溃：

1.  **LLM 的强先验**: 预训练好的 LLM 已经具备了强大的语言模型能力。对于给定的上文（状态 $s$），它本身就能生成语法正确、语义连贯的 token（动作 $a$）。这些高概率的 token（即 $\log \pi(a|s)$ 较大）往往就是“好”的动作。
2.  **正反馈循环**: 在推理任务中，“好”的动作通常能导向最终的正确答案，从而获得较高的回报，即其优势函数值 $A(s, a)$ 也较大。
3.  **持续的正协方差**: 因此，在训练的大部分时间里，$\log \pi(a|s)$ 和 $A(s, a)$ 呈现出强烈的正相关性。一个 token 的生成概率越高，它带来的优势也越大。这导致了 $\text{Cov}(\log \pi, A)$ 持续为正。
4.  **熵的单调递减**: 由于协方差项持续为正，根据定理，$\Delta H$ 持续为负。策略熵因此单调递减，直到最终崩溃。

这种“强者恒强”的正反馈循环，是导致模型迅速变得“过度自信”、丧失探索能力、最终陷入性能瓶颈的根本理论原因。

---

### 参考文献

1.  Cui, G., Zhang, Y., Chen, J., Yuan, L., Wang, Z., Zuo, Y., ... & Zhou, B. (2025). The Entropy Mechanism of Reinforcement Learning for Reasoning Language Models. *arXiv preprint arXiv:2505.22617*.
2.  Kakade, S. M. (2002). A natural policy gradient. *Advances in neural information processing systems, 15*.
