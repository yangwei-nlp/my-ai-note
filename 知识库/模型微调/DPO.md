# DPO - Direct Preference Optimization

## 简介

DPO（直接偏好优化）是一种不依赖强化学习的偏好优化方法，它通过直接优化策略模型来最大化人类偏好数据的似然，而不需要显式地训练奖励模型。

## 核心思想

传统 RLHF 流程：
1. SFT 训练 → 2. 训练奖励模型 → 3. 使用 PPO 优化

DPO 简化为：
1. SFT 训练 → 2. **直接**使用偏好数据优化策略

## 数学原理

DPO 的目标函数：

```
L_DPO(π_θ; π_ref) = -E[(x, y_w, y_l) ~ D] [log σ(β * log(π_θ(y_w|x) / π_ref(y_w|x)) - β * log(π_θ(y_l|x) / π_ref(y_l|x)))]
```

其中：
- `π_θ`：需要优化的策略模型
- `π_ref`：SFT 参考模型（通常冻结）
- `y_w`：人类偏好的回答（win）
- `y_l`：人类不喜欢的回答（lose）
- `β`：温度系数，控制偏离参考模型的程度

## 数据格式

```json
{
  "prompt": "解释量子计算是什么",
  "chosen": "量子计算是一种利用量子力学原理（如叠加和纠缠）进行信息处理的计算方式...",
  "rejected": "量子计算是一种计算机，它用量子来做计算。"
}
```

## 优势

1. **无需奖励模型**：简化了训练流程
2. **训练稳定**：避免了 RL 的不稳定性
3. **计算效率高**：单阶段训练，无需采样
4. **性能相当**：在很多任务上效果与 RLHF 相当甚至更优

## 训练参数

```python
from trl import DPOTrainer

training_args = TrainingArguments(
    learning_rate=5e-7,      # DPO 需要很小的学习率
    beta=0.1,                 # KL 散度控制系数
    # ... 其他参数
)
```

## DPO 变体

### IPO (Identity Preference Optimization)
- 解决 DPO 的过拟合问题
- 使用更优的目标函数

### KTO (Kahneman-Tversky Optimization)
- 只需要二元偏好标注（好/坏）
- 不需要成对比较数据

### RPO (Robust Preference Optimization)
- 对噪声数据更鲁棒
- 更稳定的训练过程

## 参考资料

- [DPO 论文](https://arxiv.org/abs/2305.18290)
- [TRL 库 DPO Trainer](https://huggingface.co/docs/trl/dpo_trainer)