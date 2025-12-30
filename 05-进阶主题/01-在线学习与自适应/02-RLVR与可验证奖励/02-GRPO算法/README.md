# GRPO 算法

**Authors:** Damon Li

## 1. GRPO 数学推导

GRPO (Group Relative Policy Optimization) 通过**组相对优势估计**替代 PPO 中的 Critic 网络。

### 优势函数估计

对于每个 prompt，采样 $G$ 个 completions，形成一个"组"。对于第 $i$ 个 completion：

$$A_i = \frac{r_i - \mu_{\text{group}}}{\sigma_{\text{group}} + \epsilon}$$

其中 $\mu_{\text{group}}$ 和 $\sigma_{\text{group}}$ 分别是组内奖励的均值和标准差。

### 损失函数

GRPO 使用与 PPO 相似的 Clipped Surrogate Objective：

$$L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min(r_t(\theta) A_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) A_t) \right]$$

## 2. 与 PPO 的对比分析

| 维度 | PPO | GRPO |
|:---|:---|:---|
| Critic 网络 | 需要 | 不需要 |
| 优势估计 | GAE (基于值函数) | 组相对 (基于奖励统计) |
| 内存消耗 | 高 | 低 |
| 适用场景 | RLHF | RLVR |

## 3. 收敛性证明

GRPO 的收敛性依赖于组相对优势估计是值函数的无偏估计（当 $G \to \infty$）。标准化（除以 $\sigma_{\text{group}}$）提供了自适应的梯度缩放，减少了奖励尺度对训练的影响。
