# 1、数据篇

## 1.1、组织方式

1、Alpaca

> Instruction prompt
>
> Input
>
> Output
>
> History

2、ShareGPT

这种数据格式支持的角色种类更多（不像Alpaca只有user和assistant），ShareGPT除了user和assistant还有Function Calling、Observation等

> System prompt
>
> User message
>
> Assistant message
>
> History



## 1.2、提示词

1、有无系统提示词：System prompt / None

2、有无对话历史：History / None



## 1.3、数据内容

1、纯文本

一般用来pretrain

2、角色扮演对话数据

* 角色数量

  * 单一角色

  * 多个角色

* 对话轮数

  * 1轮，无上下文/历史

  * 多轮

3、通用指令数据

通用数据通常用来防止模型知识遗忘

4、偏好数据集Pair

Chosen、Reject答案



# 2、基模

1、Instruct模型

**专为指令遵循场景训练，如果任务不涉及多轮对话，一般一轮即可就用Instruct模型再加上Alpaca格式的数据来进行指令微调即可。**

23年，较多模型区分Instruct和Chat，现在基本上不区分，都叫Instruct模型

2、Chat模型

**专为对话场景训练，比如多轮问答场景、聊天机器人场景等都会用到Chat模型。**

3、Character模型

在Chat模型上微调而来





# 3、训练

## 3.1、参数量

1、LoRA

低秩拟合矩阵会丢失部分信息，拟合数据的能力不是最好，但胜在高效

2、冻结

3、全参微调

Base模型上全参微调一个角色扮演通用模型，具备通用扮演能力



## 3.2、范式

1、预训练

2、SFT

3、偏好对齐（如DPO）



# 4、评价

## 4.1、成对评估

https://github.com/boson-ai/RPBench-Auto

评判模型来评判model1和model2的输出谁好谁坏



## 4.2、维度打分

[CHARACTERBENCH](https://es3eflgtuw.feishu.cn/wiki/AR5twOf8aiwvirkCHLJcY9LIned)

LLM在若干维度对QA进行打分（1\~5）



## 4.3、其他

CharacterEval





# 5、训练小结

## 版本总结

1、版本0.0.1和0.0.2：

* 使用的CharacterGLM的开源数据集，基模为Qwen

* 差异在于提示词有无

2、版本0.0.4：

* 使用的Character100(2k多条数据)，基模为Qwen，无对话历史

3、版本0.0.5和0.0.6：

* 使用CharacterGLM开源数据集，拟人版基模[Human-Like-Qwen](https://www.modelscope.cn/collections/Human-Like-nirenyingda-38b077cf6d0a44)

* 差异在于有无训练

4、版本0.0.7：

* 自备角色对话数据(用于RAG)，拟人版基模Index-1.9B-Character

5、版本0.0.8：

* 拟人版基模CharacterGLM

* 和内置人物对话





## 技巧总结

1、system prompt、对话历史都是必需的

* system prompt中放入角色卡信息，越多样人设拟合越好；豆包角色扮演LLM就是这个方案。

2、全参SFT往往要好于LoRA，通用角色扮演LLM基本上都是全参SFT

* 如需自定义角色可以在此基础LoRA

* 全参SFT需要混合通用数据

3、数据质量高于数据数量



# 问题

一，训练

1、全参微调用来训练通用角色模型？LoRA训练自定义角色？

2、系统提示词是否必需？如何构造？

单一角色卡构造提示词还是复杂角色卡构造提示词？

3、在训练单角色（每个角色可能性格不一样，比如幽默，高冷，热情）LoRA时是用SFT还是DPO调风格

4、如何处理角色的知识？选择什么训练范式？sft or dpo？LoRA or 全参？

5、一条数据包含一轮QA or 一条数据包含50-100轮QA 对模型有什么影响？



二，数据

1、对话历史一般构造多少？

2、构造单条数据时如何在对话中反映角色卡个性？

3、从100条数据开始慢慢训练，观察效果，那么如何去定量评估这个效果？或者如何定性评估这个效果？（bad case如何构造？）

4、合成数据能达到的上限是多少？

> ![](images/image.png)
>
> [链接](https://zhuanlan.zhihu.com/p/719135803)什么是超拟人对话风格呢？就像上图case中模型最后两行的回复一样，或者说整个截图对话内容在没有先入为主的情况下真的可以做到以假乱真了（不得不说他们这个做得真好）。可以明显发现整个对话中模型的部分非常贴近真人，**这种输出单靠合成数据这个路线一定是做不到的**，所以他们的模型一定加了大量的真人数据来进行训练。
>
> 个人小厂来讲，很难去做标注或打磨数据体系，那就只能去合成。什么方案？



# 知识储备

## 强化学习改善角色扮演

https://ai.baidu.com/ai-doc/WENXINWORKSHOP/qm28sgpvu

