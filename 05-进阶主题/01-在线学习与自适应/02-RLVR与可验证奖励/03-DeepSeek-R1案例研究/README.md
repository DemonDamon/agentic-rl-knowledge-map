# DeepSeek-R1 案例研究

**Authors:** Damon Li

## 1. R1-Zero 纯 RL 训练

DeepSeek-R1-Zero 是一个纯粹通过 RL (GRPO + RLVR) 训练的模型，没有经过 SFT 阶段。它展示了 RL 能够从头开始引导模型学习复杂的推理能力。

## 2. R1 多阶段训练流程

1. **SFT Cold Start**: 使用少量高质量 CoT 数据进行 SFT，为 RL 提供一个好的起点。
2. **Reasoning-Oriented RL**: 使用 GRPO + RLVR 进行大规模推理能力训练。
3. **Rejection Sampling**: 使用训练好的模型生成大量数据，然后通过拒绝采样筛选出高质量的 SFT 数据。
4. **RLVR & RLHF**: 结合可验证奖励和人类偏好奖励进行最终对齐。

## 3. 性能分析与基准测试

DeepSeek-R1 在多个数学和代码推理基准上达到了与 OpenAI o1-preview 相媲美的性能，证明了 RLVR + GRPO 的有效性。
