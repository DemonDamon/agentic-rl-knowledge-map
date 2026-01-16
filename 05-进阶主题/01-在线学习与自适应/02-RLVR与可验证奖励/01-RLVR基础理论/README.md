# RLVR 基础理论

> **Author**: Damon Li
> **Category**: Agentic RL / RLVR / Theory
> **Status**: Formal Definition & Mathematical Derivation

## 1. 可验证奖励的数学性质与形式化定义

RLVR 的核心在于其奖励函数 $R(s, a)$ 具有**客观性**和**确定性**，这在数学上表现为**低方差**和**高信噪比**。

### 1.1 奖励函数的形式化定义

在 RLVR 中，奖励函数 $R(s, a)$ 由一个预先存在的、确定性的外部工具（Verifier）定义 [1]。

$$
R(s, a) = \text{Verifier}(\text{Execute}(s, a))
$$

其中：
- $s$：当前状态（如问题描述和已生成的推理步骤）。
- $a$：采取的动作（如生成的下一个词元或代码块）。
- $\text{Execute}(s, a)$：执行动作后的结果，通常是最终的输出（如代码执行结果、数学证明的结论）。
- $\text{Verifier}(\cdot)$：验证器，返回一个标量奖励 $r \in \mathcal{R}$。在最简单的情况下，$\mathcal{R} = \{0, 1\}$（二元奖励），表示正确或错误。

### 1.2 低方差奖励的优势

与 RLHF 中奖励模型 $\hat{R}(s, a)$ 的**随机性**和**高方差**相比，RLVR 的奖励 $R(s, a)$ 具有极低的方差 $\text{Var}[R(s, a)] \approx 0$。

在策略梯度算法中，策略 $\pi_\theta$ 的梯度为：

$$
\nabla J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[ \sum_{t=0}^{T} \nabla \log \pi_\theta(a_t|s_t) A^\pi(s_t, a_t) \right]
$$

低方差的奖励信号使得**优势函数 $A^\pi(s_t, a_t)$ 的估计更加准确**，从而显著降低了策略梯度估计的方差，提高了训练的**数值稳定性**和**收敛速度** [2]。

---

## 2. 验证器 (Verifier) 的角色与边界

验证器是 RLVR 框架中的**外部确定性预言机 (External Deterministic Oracle)**，它不参与策略的训练，也不随策略的更新而变化。

### 2.1 验证器的核心作用

验证器的作用是提供**事实层面的真值 (Factual Ground Truth)**，将复杂的推理任务转化为可量化的奖励信号。

*   **验证工程 (Verification Engineering)**：验证器的设计和构建是一个独立于 RL 训练的工程领域 [3]。它要求验证器具备：
    1.  **完备性 (Completeness)**：能够识别所有正确的解。
    2.  **可靠性 (Soundness)**：不会错误地将错误的解标记为正确。

### 2.2 RLVR 与验证器构建的区分

*   **RLVR 不构建验证器**：RLVR 假设验证器已存在。其重点在于**如何利用验证器的输出**来优化 LLM 的推理策略。
*   **RLVR 激励推理**：研究表明，RLVR 的奖励机制能够**隐式地激励 LLM 产生正确的推理过程**，而不仅仅是正确的最终答案 [4]。这是因为只有正确的、逻辑连贯的推理路径才能持续地通过验证器的检查。

---

## 3. 奖励鲁棒性与噪声问题

尽管 RLVR 的奖励是可验证的，但在实际应用中，验证器本身可能存在局限性，导致奖励信号出现**噪声**或**稀疏性**。

### 3.1 噪声奖励的数学挑战

当验证器不完美时，奖励函数变为 $R'(s, a) = R(s, a) + \epsilon$，其中 $\epsilon$ 是噪声。

*   **噪声奖励下的策略优化**：噪声奖励会增加策略梯度估计的方差，降低训练效率。
*   **稀疏奖励**：在长序列推理任务中，只有最终结果才能被验证，导致中间步骤的奖励为零（稀疏性）。

### 3.2 解决方案

1.  **奖励平滑 (Reward Smoothing)**：使用 Monte Carlo 或 Temporal Difference 方法对稀疏奖励进行回溯和分配。
2.  **不完美奖励下的 RL**：针对带有噪声的可验证奖励，研究者提出了特定的 RL 算法来提高鲁棒性 [5]。例如，通过**不确定性感知优势塑造 (Uncertainty-aware Advantage Shaping)** 来指导探索 [6]。

---

## 4. 低方差奖励的数学优势：策略梯度方差分析

RLVR 的核心优势之一在于其**低方差**和**有界性**的奖励信号，这在数学上显著降低了策略梯度估计的方差，从而提高了训练的**数值稳定性**和**收敛速度**。

### 4.1 策略梯度估计的方差

策略梯度方法（如 REINFORCE）使用单条轨迹 $\tau$ 上的样本来估计梯度 $\hat{g}$：

$$
\hat{g} = \left( \sum_{t=0}^{T} \nabla \log \pi_\theta(a_t|s_t) \right) G(\tau)
$$

其中 $G(\tau)$ 是总回报。估计量 $\hat{g}$ 的方差 $\text{Var}[\hat{g}]$ 是衡量训练稳定性的关键指标。

为了简化分析，我们考虑一个单步 MDP，此时 $\hat{g} = \nabla \log \pi_\theta(a|s) R(s, a)$。我们关注 $\hat{g}$ 的二阶矩 $\mathbb{E}[\hat{g}^2]$，因为 $\text{Var}[\hat{g}] = \mathbb{E}[\hat{g}^2] - (\mathbb{E}[\hat{g}])^2$。

$$
\mathbb{E}[\hat{g}^2] = \mathbb{E}_{s \sim \rho^\pi, a \sim \pi_\theta} \left[ (\nabla \log \pi_\theta(a|s))^2 R(s, a)^2 \right]
$$

### 4.2 奖励方差与策略梯度方差的关系

我们观察到 $\mathbb{E}[\hat{g}^2]$ 与奖励的平方 $R(s, a)^2$ 直接相关。奖励的方差 $\text{Var}[R]$ 越大，其二阶矩 $\mathbb{E}[R^2]$ 往往越大，从而导致 $\mathbb{E}[\hat{g}^2]$ 增大，最终增大策略梯度估计的方差 $\text{Var}[\hat{g}]$。

**RLVR 的优势在于其奖励 $R_{RLVR}$ 具有天然的有界性：**

1.  **RLVR 奖励**：$R_{RLVR} \in \{0, 1\}$（二元奖励）。
    *   奖励的二阶矩 $M_2 = \mathbb{E}[R_{RLVR}^2] = 1^2 \cdot P(R=1) + 0^2 \cdot P(R=0) = P(R=1) \le 1$。
    *   **结论**：RLVR 的奖励二阶矩 $M_2$ 被严格限制在 $[0, 1]$ 范围内。

2.  **RLHF 奖励**：$R_{RLHF}$ 是奖励模型输出的连续值，通常没有严格的上限，可以非常大。
    *   奖励的二阶矩 $M_2 = \mathbb{E}[R_{RLHF}^2] = \text{Var}[R_{RLHF}] + (\mathbb{E}[R_{RLHF}])^2$。
    *   由于奖励模型的不确定性和人类偏好的主观性，$\text{Var}[R_{RLHF}]$ 往往很大，导致 $M_2$ 远大于 1，从而**显著增大了策略梯度估计的方差**。

**数学结论**：
RLVR 的可验证奖励通过将奖励限制在 $\{0, 1\}$ 集合中，有效地将 $\mathbb{E}[R^2]$ 限制在一个极小的范围内，从而**从根本上降低了策略梯度估计的方差**。

$$
\text{Var}[\hat{g}] \propto \mathbb{E}[R^2] \downarrow \implies \text{Var}[\hat{g}] \downarrow
$$

### 4.3 实际意义

*   **数值稳定性**：方差减小，避免了训练过程中的剧烈震荡和梯度爆炸。
*   **收敛速度**：方差更小的梯度估计能够更快地收敛到最优策略。
*   **样本效率**：在相同的训练效果下，RLVR 可以使用更小的批量大小或更少的环境交互次数。

---

## 参考文献

1. Wolfe, C. (2025). *Reinforcement Learning from Verifiable Rewards*. Label Studio Blog.
2. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction*. MIT Press.
3. Peng, H. (2025). *Verification Engineering for Reinforcement Learning in LLMs*. ACL Anthology.
4. Wen, X., Liu, Z., Zheng, S., et al. (2025). *Reinforcement Learning with Verifiable Rewards Implicitly Incentivizes Correct Reasoning in Base LLMs*. arXiv:2506.14245.
5. Cai, X. Q. (2025). *Reinforcement Learning with Verifiable yet Noisy Rewards*. arXiv:2510.00915.
6. Unlocking exploration in rlvr: Uncertainty-aware advantage shaping for deeper reasoning. arXiv:2510.10649.
