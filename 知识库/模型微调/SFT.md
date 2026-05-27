# 监督微调 SFT (Supervised Fine-Tuning)

## 简介

监督微调（SFT）是大模型微调的第一阶段，通过使用高质量的人工标注数据对预训练模型进行微调，使模型学习遵循指令并生成符合人类偏好的回复。

## 核心特点

- **数据形式**：(instruction, input, output) 三元组格式
- **训练目标**：最大化输出序列的似然概率
- **损失函数**：交叉熵损失（Cross-Entropy Loss）

## 与预训练的区别

| 阶段 | 目标 | 数据 | 训练方式 |
|------|------|------|----------|
| 预训练 | 学习通用语言知识 | 大规模无标注语料 | 自回归语言建模 |
| SFT | 对齐指令遵循能力 | 高质量指令数据 | 监督学习 |

## 数据构造

### Alpaca 格式

```json
{
  "instruction": "将下面的句子翻译成中文",
  "input": "Hello, how are you?",
  "output": "你好，你怎么样？"
}
```

### ShareGPT 格式

```json
{
  "conversations": [
    {"from": "human", "value": "你好"},
    {"from": "gpt", "value": "你好！有什么我可以帮助你的吗？"}
  ]
}
```

## 训练技巧

1. **学习率选择**：通常使用较小的学习率（1e-5 ~ 5e-5）
2. **Epoch 数量**：1-3 个 epoch，避免过拟合
3. **批量大小**：根据显存选择合适的 batch size
4. **序列长度**：根据任务选择合适的 max_length

## 常用框架

- **LLaMA-Factory**：一站式微调框架
- **Transformers Trainer**：HuggingFace 官方 Trainer
- **DeepSpeed**：分布式训练加速

## 参考资料

- https://arxiv.org/abs/2203.02155 (InstructGPT)
- https://github.com/tatsu-lab/stanford_alpaca
