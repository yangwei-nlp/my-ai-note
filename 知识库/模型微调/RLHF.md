# RLHF - 基于人类反馈的强化学习

## 简介

RLHF（Reinforcement Learning from Human Feedback）是一种训练大语言模型的方法，通过人类偏好数据来优化模型，使其输出更符合人类期望。

## 整体流程

```
阶段1: SFT（监督微调）
阶段2: 奖励模型训练
阶段3: 强化学习优化 (PPO/GRPO/DPO)
```

## 三个阶段详解

### 阶段1：监督微调 (SFT)

**目标**：让模型学会遵循指令和对话格式

**数据形式**：
- 高质量的指令-回答对
- 人工编写的对话数据

**训练方式**：
```python
# 最大化回答的似然概率
L_SFT = -Σ log P(y|x)
```

### 阶段2：奖励模型训练 (Reward Model)

**目标**：学习人类的偏好判断

**数据形式**（成对比较）：
```json
{
  "prompt": "解释光合作用",
  "chosen": "详细、准确、易懂的回答...",
  "rejected": "简短、模糊或不准确的回答..."
}
```

**损失函数**（Bradley-Terry 模型）：
```
L_RM = -log σ(r_θ(x, y_w) - r_θ(x, y_l))

其中：
y_w = 人类偏好的回答（win）
y_l = 人类不喜欢的回答（lose）
```

**奖励模型架构**：
- 基于 Transformer 的回归模型
- 输出标量奖励分数
- 通常从 SFT 模型初始化，修改最后一层

### 阶段3：强化学习优化

**目标**：使用奖励模型作为信号优化策略

**可选算法**：

| 算法 | 特点 | 适用性 |
|------|------|--------|
| **PPO** | 稳定、成熟，需要 Critic 网络 | 通用 RLHF |
| **GRPO** | 无需 Critic，内存高效 | 有明确答案的任务 |
| **DPO** | 无需强化学习，直接优化 | 简化流程 |
| **IPO** | DPO 的改进版，更鲁棒 | 噪声数据处理 |

## 关键概念

### 偏好数据收集

**方式1：人工标注**
- 标注员对比两个回答，选择更好的
- 成本高，质量高

**方式2：AI 辅助标注**
- 使用 GPT-4 等模型生成偏好
- 成本低，可能存在偏差

### Reward Hacking 问题

**现象**：模型找到奖励模型的漏洞，获得高分但输出质量差

**缓解策略**：
1. **KL 散度约束**：限制策略偏离参考模型
2. **多轮迭代**：定期更新奖励模型
3. **人类监控**：持续质量检查

## 开源工具

### 训练框架
- **TRL** (Transformer Reinforcement Learning)
- **LLaMA-Factory**
- **DeepSpeed-Chat**
- **OpenRLHF**

### 数据集
- **Anthropic HH-RLHF**
- **OpenAssistant Conversations**
- **SHP (Stanford Human Preferences)**
- **UltraFeedback**

## 参考资料

- [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155) (InstructGPT)
- [Learning to Summarize from Human Feedback](https://arxiv.org/abs/2009.01325)
- [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073)
- [TRL 文档](https://huggingface.co/docs/trl/index)
