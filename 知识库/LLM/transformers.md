短期目标：

1、学完后会用peft包来微调模型

2、prompt tuning vs prefix tuning vs adapter tuning

学习方式：

通过vllm项目

https://www.bilibili.com/video/BV1xY3wzZEFJ

# Transformer细节

## 生成采样

`model.generate()` 中常见的三个参数: top-k, top-p 和 temperature

采样策略：

### 1、贪心算法每次只选择最好的，

### 2、而 [Beam Search](https://zhuanlan.zhihu.com/p/721450556) 会在多个候选中进行选择，

也即每次从所有token中选出num_beams个候选，在每一步搜索时，Beam Search 会生成所有可能的下一个词汇，**并从中选择得分最高的k个序列继续下一步**。所以，束宽越大，搜索空间越广，计算成本越高。**终止条件**：当所有候选序列都生成了结束标记（如 `<eos>`）或达到设定的最大长度 时，停止生成。**然后从所有num_beams个序列中选择一个得分最高的序列，作为采样路径。**

### 3、**[Top-K 采样](https://zhuanlan.zhihu.com/p/721653652)**: 限制候选词汇数量。

k=1的时候就是贪心算法。在每一步生成过程中，模型只考虑概率最高（Top）的 K 个词汇，然后从这 K 个词汇中根据概率进行采样。K=1 就是贪心搜索。

**步骤**:

- **获取概率分布**: 模型为每个可能的下一个词汇生成一个概率分布。

- **筛选 Top-K**: 选择概率最高的 K 个词汇，忽略其余词汇。

- **重新归一化**: 将筛选后的 K 个词汇的概率重新归一化，使其总和为 1。

- **采样**: 根据重新归一化后的概率分布，从 Top-K 词汇中随机采样一个词汇作为下一个生成的词。

**使用方法**:

将 `top_p` 设置为 1（表示不使用 Top-P 采样）。比如`top_k=10`: 首先限制候选词汇为概率最高的 10 个。

```text
outputs = model.generate(
    inputs,
    max_length=50,
    do_sample=True,
    top_k=top_k,                # 设置 Top-K
    top_p=1.0,                  # 不使用 Top-P
    temperature=temperature,    # 控制生成的随机性
    no_repeat_ngram_size=2,     # 防止重复 n-gram
    eos_token_id=tokenizer.eos_token_id
)
```

### 4、**Top-P 采样（Nucleus Sampling）**: 根据累积概率选择候选词汇，动态调整词汇集。

- **获取概率分布**: 模型为每个可能的下一个词汇生成一个概率分布。

- **排序概率**: 将词汇按照概率从高到低排序。

- **累积概率**: 计算累积概率，直到达到预设的阈值 P。

- **筛选 Top-P**: 选择累积概率达到 P 的最小词汇集合。
- **重新归一化**: 将筛选后的词汇概率重新归一化。
- **采样**: 根据重新归一化后的概率分布，从 Top-P 词汇中随机采样一个词汇作为下一个生成的词。

**只使用 Top-P**:

将 `top_k` 设置为 0（表示不使用 Top-K 采样）。

```text
outputs = model.generate(
    inputs,
    max_length=50,
    do_sample=True,
    top_k=0,                    # 不使用 Top-K
    top_p=top_p,                # 设置 Top-P
    temperature=temperature,    # 控制生成的随机性
    no_repeat_ngram_size=2,     # 防止重复 n-gram
    eos_token_id=tokenizer.eos_token_id
)
```



这四种采样策略各有优劣，没有绝对的"最好"，关键是要根据具体应用场景来选择：

### 各策略特点对比

**贪心算法**

- 优点：速度最快，输出最确定性
- 缺点：容易陷入重复循环，缺乏多样性
- 适用：需要高度确定性输出的场景，如翻译、摘要

**Beam Search**

- 优点：平衡了质量和多样性，避免局部最优
- 缺点：计算成本高，仍可能产生重复
- 适用：需要高质量输出的任务，如机器翻译、文本生成

**Top-K采样**

- 优点：增加随机性和多样性，计算简单
- 缺点：K值难以确定，可能选择不合适的词汇
- 适用：创意写作、对话生成等需要多样性的场景

**Top-P采样**

- 优点：动态调整候选集，既保证质量又有多样性
- 缺点：计算稍复杂，参数调优需要经验
- 适用：大多数生成任务的默认选择

### 实践建议

1. **通用推荐**：Top-P采样（P=0.9）+ 适当温度调节，这是目前最平衡的方案
2. **组合使用**：Top-K + Top-P 结合，先用Top-K限制范围，再用Top-P精细调节
3. **任务导向**：
   - 事实性任务：贪心或小beam_size的Beam Search
   - 创造性任务：Top-P或Top-K采样
   - 对话系统：Top-P + 温度调节
4. **动态调整**：根据生成过程中的上下文动态调整参数

目前业界主流的做法是以**Top-P采样为主**，因为它能够自适应地平衡质量与多样性。



## 温度参数

低温度（T < 1）：使概率分布更“尖锐”，高概率的选项被进一步放大，低概率的选项被抑制。这通常导致输出更确定、更保守，倾向于选择最可能的词。
高温度（T > 1）：使概率分布更“平滑”，减小高概率和低概率选项之间的差距。这增加了输出的随机性，模型更可能生成多样化或富有创意的文本。
温度为1（T = 1）：保持原始概率分布不变，等价于直接使用模型输出的logits进行采样。







# Attention机制

1. **计算注意力分数**：QK^T 
   1. Q (Query): 形状为 [seq_len, d_model]
   2. K (Key): 形状为 [seq_len, d_model]
   3. QK^T 的结果形状为 [seq_len, seq_len]

QK^T：(i, j)代表token_i和token_j的点积分数

1. **应用softmax函数**： 

```Plain
Attention_weights = softmax(QK^T / √d_k)
```

关于Mask机制：

BERT这种模型利用的是Transformer中的Encoder架构，即双向注意力机制，而Qwen等当代大模型，利用的是Transformer中的Decoder架构，即单向注意力/因果注意力机制，确保模型在预测下一个token时只能看到当前位置之前的信息。

## 从BERT到Qwen：Transformer架构演进史

1. BERT (2018) - Encoder-Only 开山之作

**架构特点：**

- **纯Encoder架构**： 只使用Transformer的encoder部分
- **双向注意力**：可以同时看到左右上下文
- **Mask机制**： 
  - Padding Mask：掩盖填充token
  - MLM Mask：随机掩盖15%的token进行预训练
- **预训练任务**： 
  - Masked Language Model (MLM)
  - Next Sentence Prediction (NSP)

**参数规模：**

- BERT-Base: 110M参数，12层
- BERT-Large: 340M参数，24层

**适用场景：** 文本理解、分类、问答等判别任务

1. GPT-1/2/3 (2018-2020) - Decoder-Only 生成先锋

**架构特点：**

- **纯Decoder架构**：只使用Transformer的decoder部分
- **单向注意力**：只能看到当前位置之前的token
- **Mask机制**： 
  - Causal Mask：下三角掩码确保自回归特性
  - Padding Mask：处理变长序列
- **预训练任务**：Next Token Prediction

**参数规模演进：**

- GPT-1: 117M参数
- GPT-2: 1.5B参数
- GPT-3: 175B参数

**突破意义：** 证明了decoder-only架构在生成任务上的强大能力

1. T5 (2019) - Encoder-Decoder 统一框架

**架构特点：**

- **完整Encoder-Decoder架构**：保留原始Transformer设计
- **Text-to-Text统一范式**：所有NLP任务都转化为文本生成
- **Mask机制**： 
  - Encoder: Padding Mask
  - Decoder: Causal Mask + Padding Mask + Cross-Attention Mask
- **预训练任务**：Span Corruption（随机掩盖文本片段）

**参数规模：**

- T5-Small: 60M参数
- T5-Base: 220M参数
- T5-Large: 770M参数
- T5-11B: 11B参数

**创新点：** 统一的文本到文本框架，适合多种NLP任务

1. LLaMA (2023) - Decoder-Only 的现代优化

**架构特点：**

- **Decoder-Only架构**：继承GPT的设计思路
- **架构优化**： 
  - RMSNorm替代LayerNorm
  - SwiGLU激活函数
  - Rotary Position Embedding (RoPE)
  - Pre-normalization
- **Mask机制**：Causal Mask + Padding Mask

**参数规模：**

- LLaMA: 7B, 13B, 30B, 65B
- LLaMA 2: 7B, 13B, 70B
- LLaMA 3: 8B, 70B, 405B

**技术突破：** 在相对较小的参数下达到优异性能

1. Qwen系列 (2023-2025) - 多模态Decoder-Only

**架构特点：**

- **Decoder-Only架构**：基于LLaMA的设计理念
- **多模态扩展**： 
  - Qwen-VL: 集成视觉编码器
  - Qwen-Audio: 集成音频编码器
- **Mask机制**： 
  - 主要使用Causal Mask
  - Padding Mask处理变长序列
  - 多模态融合时的特殊attention mask

**参数规模演进：**

- Qwen1: 1.8B, 7B, 14B, 72B
- Qwen2: 0.5B, 1.5B, 7B, 57B, 72B
- Qwen2.5: 0.5B, 1.5B, 3B, 7B, 14B, 32B, 72B

**技术特色：**

- 多语言能力（中英文优化）
- 长上下文支持
- 多模态理解

## 架构演进趋势总结

### 第一阶段：三足鼎立 (2018-2020)

- **BERT**：Encoder-only，专攻理解
- **GPT**：Decoder-only，专攻生成
- **T5**：Encoder-decoder，统一框架

### 第二阶段：Decoder-Only主导 (2020-至今)

**为什么Decoder-Only胜出？**

1. **简化架构**：去除复杂的cross-attention
2. **更好的可扩展性**：适合大规模预训练
3. **统一的生成范式**：理解和生成任务都用同一套架构
4. **涌现能力**：大规模下展现出强大的zero-shot和few-shot能力

### 关键技术演进

1. **位置编码**：从固定编码到RoPE
2. **归一化**：从LayerNorm到RMSNorm
3. **激活函数**：从ReLU到SwiGLU
4. **注意力机制**：从标准attention到各种优化版本
5. **多模态融合**：从纯文本到多模态理解

### Mask使用模式的变化

- **早期**：复杂的多种mask组合
- **现代**：主要依赖Causal Mask，架构简化但效果更强

这个演进过程体现了AI领域"简单即美"的设计哲学：通过架构简化和规模扩大，实现了更强的模型能力。

# Attention缺陷

计算复杂度高，内存占用大

Online Softmax、flashattention

9-1目标：

1、attention机制、pagedattention、flashattention、kv-cache等

https://claude.ai/chat/3877f341-a08f-4e38-bc76-67c362d9d391