# OpenManus-RL 深度调研：Agentic RL 的工程实践与算法创新

> **Author**: Damon Li
> **Category**: Agentic RL / RLVR / Engineering Practice
> **Status**: Deep Research Report

## 1. 项目概述与定位

**OpenManus-RL** 是由 Ulab-UIUC 与 MetaGPT 联合发起的开源项目，旨在探索基于强化学习（RL）的 LLM Agent 微调新范式。该项目深受 DeepSeek-R1、QwQ-32B 等推理模型成功的启发，致力于在 Agent 场景（如 GAIA, WebShop, AlfWorld）中实现高效的 RL 训练。

### 1.1 核心贡献
*   **集成 verl 框架**：深度集成字节跳动开源的 `verl` 框架，提供高性能的分布式 RL 训练能力。
*   **GiGPO 算法创新**：提出 Generalized Interaction Group Policy Optimization (GiGPO)，针对多轮交互场景优化奖励分配。
*   **多模态支持**：原生支持视觉观察（Vision-Language Models），适用于 OSWorld 等复杂 GUI 任务。
*   **标准化数据集**：开源了包含 5 万余条轨迹的 Agent SFT 数据集，涵盖 ReAct 框架下的多轮交互。

---

## 2. GiGPO 算法：多轮交互下的优势估计

在 Agentic RL 中，传统的 GRPO（Group Relative Policy Optimization）主要针对单轮推理（如数学竞赛题）。OpenManus-RL 提出的 **GiGPO** 将其扩展到了多轮交互场景。

### 2.1 数学定义

在多轮交互中，一个轨迹 $\tau$ 由一系列状态、动作和奖励组成：$\tau = (s_0, a_0, r_0, s_1, a_1, r_1, \dots, s_T, a_T, r_T)$。

GiGPO 的核心思想是在两个维度上进行奖励归一化：
1.  **Episode-level Group**: 针对同一个 Prompt 的不同采样轨迹进行对比。
2.  **Step-level Group**: 针对相同中间状态（Anchor Observation）下的不同动作分支进行对比。

### 2.2 优势函数推导

GiGPO 的优势函数 $A_{GiGPO}$ 定义为 Episode 优势与 Step 优势的加权和：

$$A_{GiGPO} = A_{episode} + \omega \cdot A_{step}$$

其中：
*   **Episode Advantage** ($A_{episode}$):
    $$A_{episode, i} = \frac{R_i - \text{mean}(\{R_j\}_{j \in G_{prompt}})}{\text{std}(\{R_j\}_{j \in G_{prompt}}) + \epsilon}$$
    这里 $R_i = \sum_{t=0}^T \gamma^t r_{i,t}$ 是第 $i$ 条轨迹的总回报。

*   **Step Advantage** ($A_{step}$):
    针对具有相同观察 $s_t$ 的动作 $a_t$，计算其相对优势：
    $$A_{step, t} = \frac{r_t + \gamma V(s_{t+1}) - \text{mean}(\text{Q-values at } s_t)}{\text{std}(\text{Q-values at } s_t) + \epsilon}$$

### 2.3 算法优势分析

| 特性 | GRPO (Original) | GiGPO (OpenManus-RL) |
| :--- | :--- | :--- |
| **适用场景** | 单轮推理 (CoT) | 多轮交互 (Agent) |
| **基准线 (Baseline)** | 组内平均奖励 | 组内平均 + 状态锚点平均 |
| **方差控制** | 依赖组大小 $G$ | 结合状态锚点，进一步降低动作方差 |
| **环境反馈** | 仅最终 Outcome | 包含中间 Step Rewards |

---

## 3. 系统架构与工程实现

OpenManus-RL 的架构设计体现了 Agentic RL 的典型工程模式：

### 3.1 模块化设计
*   **`TrajectoryCollector`**: 负责与环境交互并收集轨迹。支持多模态输入处理，通过 `tokenizer.apply_chat_template` 保持与模型训练的一致性。
*   **`EnvironmentManager`**: 封装了 AlfWorld 和 Webshop 等环境。引入了 `SummarizedMemory`，利用 LLM 对长历史进行摘要，解决 Context Window 限制。
*   **`verl` 集成**: 利用 `verl` 的 `DataProto` 进行高效的数据传递，支持 PPO、DPO 和 GiGPO 等多种算法。

### 3.2 奖励模型 (Reward Model)
项目支持多种奖励信号：
1.  **Outcome-based**: 任务是否成功的硬指标（如 Webshop 的订单成功）。
2.  **Format-based**: 强制模型遵循 ReAct 格式（`Think: ... Act: ...`）。
3.  **Step-level**: 环境提供的中间奖励，通过 GiGPO 进行归一化。

---

## 4. 在知识图谱中的归类建议

基于上述分析，OpenManus-RL 应归类于 **进阶主题 > 在线学习与自适应 > RLVR与可验证奖励** 模块下，作为 **开源实现与工程实践** 的核心案例。

### 4.1 关联路径
*   **理论关联**：与 `01-RLVR基础理论` 中的低方差奖励推导相呼应。
*   **算法关联**：作为 `02-GRPO算法` 的多轮交互扩展版本。
*   **实践关联**：为 `04-开源实现与工程实践` 提供具体的代码参考和 Benchmark 结果。

---

## 5. 参考文献

1.  Zhu, K., et al. (2025). *OpenManus-RL: Scaling Reinforcement Learning for LLM Agents*. GitHub Repository. [https://github.com/OpenManus/OpenManus-RL](https://github.com/OpenManus/OpenManus-RL)
2.  Bytedance. (2024). *verl: Volcano Engine Reinforcement Learning for LLMs*. [https://github.com/volcengine/verl](https://github.com/volcengine/verl)
3.  DeepSeek-AI. (2025). *DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning*. arXiv preprint.
4.  Shao, Z., et al. (2024). *DeepSeek-V3 Technical Report*.
