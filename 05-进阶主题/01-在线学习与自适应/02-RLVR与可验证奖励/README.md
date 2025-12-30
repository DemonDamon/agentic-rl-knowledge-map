# RLVR 与可验证奖励

**Authors:** Damon Li

## 1. 概述

**RLVR (Reinforcement Learning with Verifiable Rewards)** 是一种强化学习范式，它使用客观、可自动验证的外部信号作为奖励，来驱动策略优化。与依赖主观人类反馈的 RLHF 不同，RLVR 的核心是“奖励客观可验”，常见于代码生成、数学推理等结果导向型任务。

## 2. 核心概念

- **可验证奖励**: 来自编译器、单元测试、数学证明器等自动化工具的二元（0/1）或标量奖励。
- **Agentic RL 融合**: 当 RLVR 用于多步推理、工具调用时，与 Agentic RL 高度重合。
- **GRPO 算法**: RLVR 的主流优化算法，通过组相对优势估计替代 Critic 网络。

## 3. 目录结构

- [**01-RLVR基础理论**](./01-RLVR基础理论/README.md)
- [**02-GRPO算法**](./02-GRPO算法/README.md)
- [**03-DeepSeek-R1案例研究**](./03-DeepSeek-R1案例研究/README.md)
- [**04-开源实现与工程实践**](./04-开源实现与工程实践/README.md)

## 4. 参考文献

1. Wen, X., Liu, Z., Zheng, S., et al. (2025). *Reinforcement Learning with Verifiable Rewards Implicitly Incentivizes Correct Reasoning in Base LLMs*. arXiv:2506.14245.
2. Shao, Z., Wang, P., Zhu, Q., et al. (2024). *DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models*. arXiv:2402.03300.
