# PPO - Proximal Policy Optimization

## 简介

PPO（近端策略优化）是一种强化学习算法，在 RLHF（基于人类反馈的强化学习）中被广泛用于优化语言模型以符合人类偏好。

## RLHF 三阶段流程

```
阶段1：SFT（监督微调）
    ↓
阶段2：训练奖励模型（Reward Model）
    ↓
阶段3：PPO 强化学习优化
```

## PPO 核心公式

### 目标函数

```
L^{CLIP}(θ) = E_t [min(r_t(θ) * A_t, clip(r_t(θ), 1-ε, 1+ε) * A_t)]

其中：
r_t(θ) = π_θ(a_t|s_t) / π_old(a_t|s_t)   ← 新旧策略比率
A_t    = 优势函数估计（Advantage）
ε      = 裁剪参数（通常 0.1 或 0.2）
```

### KL 惩罚

为防止模型偏离太远，加入 KL 散度约束：

```
L_total = L^{CLIP} - β * KL[π_θ || π_ref]
```

## 训练架构

```
┌─────────────────────────────────────────────────────┐
│                     PPO Trainer                      │
├─────────────────────────────────────────────────────┤
│  Actor (策略模型 π_θ)  ←  需要训练的模型            │
│  Critic (价值模型 V)    ←  预测状态价值              │
│  Reward Model           ←  人类偏好打分（冻结）      │
│  Reference Model        ←  SFT 模型（冻结）          │
└─────────────────────────────────────────────────────┘
```

## 训练流程

1. **采样阶段**
   - 使用当前策略生成回答
   - 计算奖励模型的分数
   - 计算 KL 惩罚

2. **更新阶段**
   - 计算优势值（GAE - Generalized Advantage Estimation）
   - 多次 mini-batch 更新策略
   - 应用 PPO-Clip 约束

## GAE (Generalized Advantage Estimation)

```
A_t = δ_t + (γλ) * δ_{t+1} + (γλ)^2 * δ_{t+2} + ...

其中：
δ_t = r_t + γ * V(s_{t+1}) - V(s_t)
γ = 折扣因子
λ = GAE 参数
```

## 超参数设置

| 参数 | 说明 | 典型值 |
|------|------|--------|
| `learning_rate` | 学习率 | 1e-6 ~ 1e-7 |
| `ppo_epochs` | 每次采样的更新轮数 | 4 |
| `mini_batch_size` | 小批量大小 | 1-4 |
| `clip_eps` | PPO 裁剪系数 | 0.2 |
| `kl_coef` | KL 惩罚系数 | 0.01-0.1 |
| `gamma` | 折扣因子 | 1.0 |
| `lam` | GAE 参数 | 0.95 |

## 代码示例

```python
from trl import PPOConfig, PPOTrainer

# 配置
config = PPOConfig(
    model_name="model_path",
    learning_rate=1e-6,
    batch_size=1,
    mini_batch_size=1,
    ppo_epochs=4,
)

# 初始化
transformer_model = AutoModelForCausalLM.from_pretrained(...)
reward_model = AutoModelForSequenceClassification.from_pretrained(...)
tokenizer = AutoTokenizer.from_pretrained(...)

ppo_trainer = PPOTrainer(
    config=config,
    model=transformer_model,
    tokenizer=tokenizer,
)

# 训练循环
for query_tensor in dataloader:
    # 生成回答
    response_tensors = ppo_trainer.generate(query_tensor)
    
    # 计算奖励
    rewards = reward_model(query_tensor, response_tensors)
    
    # PPO 更新
    stats = ppo_trainer.step(query_tensor, response_tensors, rewards)
```

## 常见问题

### 1. Reward Hacking
- **现象**：模型学会钻奖励模型的漏洞，输出高分但低质量的内容
- **解决**：
  - 使用 KL 约束
  - 定期更新奖励模型
  - 多轮迭代训练

### 2. 训练不稳定
- **现象**：损失发散，生成质量波动
- **解决**：
  - 降低学习率
  - 增加 KL 惩罚
  - 减小 PPO 裁剪范围

### 3. 模式崩溃
- **现象**：模型输出趋于同质化
- **解决**：
  - 增加采样温度
  - 使用 diverse 的 prompt
  - 正则可项

## PPO vs GRPO vs DPO

| 方法 | 需要 Critic | 需要 RM | 需要采样 | 适用场景 |
|------|-------------|---------|----------|----------|
| PPO | ✅ | ✅ | ✅ | 通用 RLHF |
| GRPO | ❌ | ❌ | ✅ | 有明确答案的任务 |
| DPO | ❌ | ❌ | ❌ | 偏好数据丰富 |

## 参考资料

- [Proximal Policy Optimization Algorithms](https://arxiv.org/abs/1707.06347)
- [TRL PPO Trainer](https://huggingface.co/docs/trl/ppo_trainer)
- [InstructGPT 论文](https://arxiv.org/abs/2203.02155)
