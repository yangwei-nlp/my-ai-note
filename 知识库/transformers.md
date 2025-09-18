短期目标：

1、学完后会用peft包来微调模型

2、prompt tuning vs prefix tuning vs adapter tuning

学习方式：

通过vllm项目

https://www.bilibili.com/video/BV1xY3wzZEFJ

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