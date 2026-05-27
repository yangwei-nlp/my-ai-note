# GRPO - Group Relative Policy Optimization

## 简介

GRPO（组相对策略优化）是 DeepSeek 团队提出的一种在线强化学习算法，用于提升大语言模型的推理能力，特别是在数学和代码推理任务上表现优异。

## 与 PPO 的区别

| 特性 | PPO | GRPO |
|------|-----|------|
| 基线估计 | 需要价值模型 Critic | 使用同一问题的多个采样结果计算相对优势 |
| 内存需求 | 需要额外存储价值模型 | 无需 Critic，内存效率高 |
| 采样方式 | 单次采样 | 每问题生成 G 个回答 |

## 核心算法

### 四个主要步骤

```
1. 生成补全 (Generate Completions)
   对同一个问题生成 G 个不同的回答

2. 计算优势 (Compute Advantage)
   A_i = (r_i - mean(r)) / std(r)
   其中 r_i 是第 i 个回答的奖励

3. 估计 KL 散度 (Estimate KL Divergence)
   D_KL = log(π_ref / π_θ)

4. 计算损失 (Compute Loss)
   L_GRPO = max(
       π_θ/π_old * A,
       clip(π_θ/π_old, 1-ε, 1+ε) * A
   ) - β * D_KL
```

## 奖励函数设计

GRPO 通常使用基于规则的奖励函数（Rule-based RM）：

### 格式奖励
```python
# 检查是否包含特定格式标签
format_reward = 1.0 if "<answer>" in response and "</answer>" in response else 0.0
```

### 答案正确性奖励
```python
# 数学问题：验证最终答案是否与标准答案匹配
accuracy_reward = 1.0 if extract_answer(response) == ground_truth else 0.0
```

### 过程奖励（可选）
```python
# 对中间推理步骤给予奖励
process_reward = calculate_step_correctness(response_steps)
```

## 训练流程

```python
from trl import GRPOConfig, GRPOTrainer

# 定义奖励函数
def reward_func(prompts, completions, **kwargs):
    rewards = []
    for prompt, completion in zip(prompts, completions):
        # 自定义奖励计算逻辑
        reward = compute_reward(prompt, completion)
        rewards.append(reward)
    return rewards

training_args = GRPOConfig(
    learning_rate=1e-6,
    per_device_train_batch_size=1,
    num_generations=8,  # 每组采样 G=8 个回答
    # ...
)

trainer = GRPOTrainer(
    model=model,
    reward_funcs=reward_func,
    args=training_args,
    train_dataset=dataset,
)
```

## 关键超参数

| 参数 | 说明 | 典型值 |
|------|------|--------|
| `num_generations` (G) | 每问题采样数 | 4-16 |
| `beta` | KL 惩罚系数 | 0.01-0.1 |
| `clip_eps` | PPO 裁剪阈值 | 0.2 |

## 应用场景

1. **数学推理**：通过验证答案正确性提供奖励
2. **代码生成**：通过执行代码测试正确性
3. **逻辑推理**：多步推理任务

## 优势

- 无需训练奖励模型，降低复杂度
- 使用相对优势，减少方差
- 特别适合有明确正确答案的任务

## 参考资料

- [DeepSeekMath 论文](https://arxiv.org/abs/2402.03300)
- [TRL GRPO Trainer](https://huggingface.co/docs/trl/grpo_trainer)
- [EasyR1 训练框架](https://github.com/hiyouga/EasyR1)