时间：2025-8-1

简介：

**在论文中，Anthropic 为各种人物角色训练了向量，包括：**

* **The Apologizer** — who consistently takes blame
  **道歉者&#x20;**——总是承担责任

* **The Cynic** — who is skeptical of almost everything
  **愤世嫉俗者&#x20;**——对几乎所有事物都持怀疑态度

* **The Optimist** — who frames all situations positively
  **乐观主义者&#x20;**——积极地看待所有情况

* **The Pirate** — who speaks in “Arrr”s but maintains semantic coherence
  **海盗&#x20;**——说话带有“Arrr”的语气，但保持语义的连贯性

**它做到了将知识和表达分开的特点。**



**Are personality traits** ***really*** **linear in LLMs?**
**法学硕士 (LLM) 中的性格特征*真的*是线性的吗？**
This technique assumes a high degree of modularity in internal representations. That’s a fascinating hypothesis — but one that demands scrutiny.
这项技术假设内部表征具有高度的模块性。这是一个引人入胜的假设，但需要进一步验证。



思考：

**大概意思就是在训练时在模型架构上加入了一个向量，然后用不同身份的数据（邪恶、谄媚和幻觉等）来训练，训练完成后，每类数据得到一个向量权重，推理时将该训练好的向量加入模型架构上，推理时就能按照该风格的表达来说话。**



https://www.anthropic.com/research/persona-vectors

https://arxiv.org/abs/2507.21509

https://github.com/safety-research/persona\_vectors
