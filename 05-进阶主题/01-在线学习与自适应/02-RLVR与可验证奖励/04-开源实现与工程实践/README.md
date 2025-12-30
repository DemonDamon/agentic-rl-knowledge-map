# 开源实现与工程实践

**Authors:** Damon Li

## 1. Hugging Face TRL

Hugging Face 的 TRL (Transformer Reinforcement Learning) 库提供了 `GRPOTrainer`，可以方便地进行 GRPO 训练。

**URL**: https://huggingface.co/docs/trl/main/en/grpo_trainer

## 2. VERL 框架

VERL (Volcano Engine Reinforcement Learning) 是一个专门为 LLM RL 训练设计的框架，支持分布式训练和高效的 GRPO 实现。

**URL**: https://verl.readthedocs.io/en/latest/algo/grpo.html

## 3. 训练技巧与超参数调优

- **组大小 (Group Size)**: 通常设置为 4-8，太小会导致优势估计不准，太大则增加计算成本。
- **学习率 (Learning Rate)**: 建议使用较小的学习率，如 1e-6 到 1e-5。
- **KL 惩罚系数 (β)**: 用于控制策略更新的幅度，防止偏离太远。
- **奖励归一化**: GRPO 内置了奖励归一化，对奖励尺度不敏感。
