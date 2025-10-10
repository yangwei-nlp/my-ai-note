# **前言**

**准备好迎接搜索 3.0 时代了吗？**

随着这几年 AI 技术的革新，“搜索应用” 成为了 AI 应用层的第一个共识。

从海外的 OpenAI、微软 Bing Copilot、Perplexity AI，再到国内的豆包、Kimi，都是这一共识下的代表产品。

**技术上，从传统的关键词检索，到 RAG，大家已经不满足于只是生成对应的简单回答**

而是期待大语言模型能够更好地应用于**企业级场景**，产生更大的价值。

不久前，OpenAI 推出了最新的深度内容生成神器 “DeepResearch”，用户只需一个 "特斯拉的合理市值是多少" 的提问，

DeepResearch 就能生成一份包括企业财务、业务增长分析，再到最后的市值推演的专业分析报告。

**而这，也指明了搜索 AGI 的发展方向。**

在此背景下，如何基于 “DeepResearch” 理念，对其做定制化改造，成为了近一个月来的市场热门话题。

当然，Zilliz 也是其中非常幸运的一员。**不久前我们推出了 DeepSearcher 开源项目 ，不到半个月，在 GitHub 收获的 star 数量就已经超过 3100！**

感谢全球开发者和各位读者们的火热支持！

![img](https://oscimg.oschina.net/oscnet/c7f4b311-ab04-4295-b149-e6a26574235c.png)

尝鲜链接：https://github.com/zilliztech/deep-searcher

建立在 DeepResearch 大模型 + 超级搜索 + 研究助理的三合一的基础上，

DeepSearcher 还通过 Milvus 向量数据库引入本地数据，并支持用户自由更换包括 DeepSeek-R1 在内的底层模型，为用户带来了更符合企业级场景的全新 RAG 范式。



## **01** 



## **从 RAG 到 Agentic RAG，更聪明的推理，到底为 RAG 带来了什么？**

如果要给 DeepSearcher 一个明确的定性，那它应该属于一个非常典型的 Agentic RAG 架构。

所谓 Agentic RAG，是一种融合智能代理（Agent）能力的 RAG 架构。通过动态规划、多步骤推理和自主决策机制，Agentic RAG 可以在复杂任务中实现闭环的检索 - 加工 - 验证 - 优化。

其崛起，主要受到大模型推理能力提升的影响。

![img](https://oscimg.oschina.net/oscnet/3fe9c874-3f0a-4662-997a-39dd9af45855.png)

相比传统 RAG，Agentic RAG 可以将我们的问题，通过大模型的推理能力，拆分成多个子问题，并在多轮查询中，不断的迭代与优化，直到得到满意结果。

相应的，Agentic RAG 相比传统 RAG 有着三大优势：

（1）被动响应变为主动响应；

（2）单次的关键词检索，升级为多轮的动态调整检索，并拥有自我修正能力；

（3）适用场景，从最基础的简单事实问答，升级为复杂推理、复杂报告生成等开放域任务。

而基于以上能力的升级，DeepSearcher 能够像人类专家一样，面对提问，不仅给出答案，更能给出推理过程、执行细节在内的一整套完整方案。

在这一背景下，**长期来看，Agentic RAG 必定会取代传统 RAG。**一方面，传统 RAG 对需求的响应还停留在非常基础的阶段，另一方面，现实中，**我们大部分的需求表达背后，都是有隐含逻辑的，并不能被一步检索到位**，必须通过推理 - 反思 - 迭代 - 优化来对其进行拆解与反馈。



## **02** 



## **DeepSearcher 的架构设计**

一个通往搜索 AGI 的 Agentic RAG 应该如何设计？

从架构上看，DeepSearcher 主要分为两大模块。

![img](https://oscimg.oschina.net/oscnet/f623f5c4-954f-496c-a574-d98b927a55ad.png)

一个是数据接入模块，通过 Milvus 向量数据库来接入各种第三方的私有知识。这也是 DeepSearcher 相比 OpenAI 的原本 DeepResearch 做出的一大重大升级 —— 更适合拥有独家数据的企业级场景。

另外一部分是在线推理查询模块。这个模块包括了各种 Agent 策略以及 RAG 的实现部分，负责给用户提供准确有深度的回答。

这部分引入了动态循环迭代机制**：**每次对向量数据库中内容完成数据查询后，系统都会启动一个反馈（reflection）流程，然后在每一轮迭代结束时，智能体（Agent）会对查询到的知识进行评估，判断其是否足以解答初始提出的问题。若发现仍存在知识缺口，便会触发下一轮迭代查询；若判定已有足够知识来作答，系统就会生成最终报告 。

**这种不断 “追问”“反思” 的能力，其实也是 Agentic RAG 相比传统 RAG 的最大进步，甚至也是人类的文学、历史、哲学、科学千年来不断进步的根源所在。**



## **03** 



## **Deep Searcher 的效果到底如何，适合哪些场景？**

我们可以看到，DeepSearcher 在完成安装并导入本地文件后，针对我们的 query 命令，会对所有导入的内容进行深度搜索和分析，在搜索过程中，会打印每一步的搜索过程和思考过程，并对结果进行最后的整合。

![img](https://oscimg.oschina.net/oscnet/de5187fa-88ad-4152-91f4-f45bfa045c52.gif)

相应的，这种相对 **“重思考”“重算力投入”** 的模式，在落地中，更适合 “综述类提问” 以及 “含复杂推理类提问” 的场景。

**（1）综述类提问**

综述类提问的典型代表是写报告，写文档，写总结，它一般会给出一个简短的主题，但要求尽可能多的给出详尽汇报输出。

我们以 “How has The Simpsons changed over time?”(《辛普森一家》动画剧集随着时间的推移发生了什么变化？) 为例，在 Deep Searcher 第一轮，生成的初始子查询如下文所示：

```

Break down the original query into new sub queries: [  'How has the cultural impact and societal relevance of The Simpsons evolved from its debut to the present?',  'What changes in character development, humor, and storytelling styles have occurred across different seasons of The Simpsons?',   'How has the animation style and production technology of The Simpsons changed over time?',  'How have audience demographics, reception, and ratings of The Simpsons shifted throughout its run?']
在经过检索信息，反馈迭代后，又生成下一轮的子查询问题：

New search queries for next iteration: [  "How have changes in The Simpsons' voice cast and production team influenced the show's evolution over different seasons?",  "What role has The Simpsons' satire and social commentary played in its adaptation to contemporary issues across decades?",  'How has The Simpsons addressed and incorporated shifts in media consumption, such as streaming services, into its distribution and content strategies?']
每一轮的子问题都回到知识库里面检索，生成最后完整的报告：

**Report: The Evolution of _The Simpsons_ (1989–Present)**
**1. Cultural Impact and Societal Relevance**  _The Simpsons_ debuted as a subversive critique of American middle-class life, gaining notoriety for its bold satire in the 1990s. Initially a countercultural phenomenon, it challenged norms with episodes tackling religion, politics, and consumerism. Over time, its cultural dominance waned as competitors like _South Park_ and _Family Guy_ pushed boundaries further. By the 2010s, the show transitioned from trendsetter to nostalgic institution, balancing legacy appeal with attempts to address modern issues like climate change and LGBTQ+ rights, albeit with less societal resonance.
**2. Character Development and Storytelling Shifts**  Early seasons featured nuanced character arcs (e.g., Lisa’s activism, Marge’s resilience), but later seasons saw "Flanderization" (exaggerating traits, e.g., Homer’s stupidity, Ned Flanders’ piety). Humor evolved from witty, character-driven satire to reliance on pop culture references and meta-humor. Serialized storytelling in early episodes gave way to episodic, gag-focused plots, often sacrificing emotional depth for absurdity.
[...]
**12. Merchandising and Global Reach**  The 1990s merchandise boom (action figures, _Simpsons_-themed cereals) faded, but the franchise persists via collaborations (e.g., _Fortnite_ skins, Lego sets). International adaptations include localized dubbing and culturally tailored episodes (e.g., Japanese _Itchy & Scratchy_ variants).
**Conclusion**  _The Simpsons_ evolved from a radical satire to a television institution, navigating shifts in technology, politics, and audience expectations. While its golden-age brilliance remains unmatched, its adaptability—through streaming, updated humor, and global outreach—secures its place as a cultural touchstone. The show’s longevity reflects both nostalgia and a pragmatic embrace of change, even as it grapples with the challenges of relevance in a fragmented media landscape.
```

（报告过长，这里截取片段展示，完整内容参考视频）

<iframe class="video_iframe rich_pages" data-vidtype="2" data-mpvid="wxv_3883754507762892803" data-cover="http%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_jpg%2FMqgA8Ylgeh4wRYzJPkrxOP2wMciaLmG7m1Zwqaw0A14iayLiaMDniaWjbEMUCCGK3jb5ibbjMC1mFaSibt07kzkIbvHg%2F0%3Fwx_fmt%3Djpeg" allowfullscreen="" frameborder="0" data-ratio="1.6222222222222222" data-w="1752" data-src="https://mp.weixin.qq.com/mp/readtemplate?t=pages/video_player_tmpl&amp;action=mpvideo&amp;auto=0&amp;vid=wxv_3883754507762892803" style="box-sizing: inherit; border-radius: 4px;"></iframe>

**（2）含复杂推理类提问**

含复杂推理类提问，通常在提问中包含着隐含着复杂的逻辑推理，比如问谁的七大姑八大姨家的小孩上学的班级的班主任名字。这类问题往往存在多个实体关系或者逻辑之间的连接，需要多次反馈迭代查询。

我们以问题：

“Which film has the director who is older, God'S Gift To Women or Aldri annet enn bråk.”（哪部电影的导演年龄更大，是《God'S Gift To Women》还是 《Aldri annet enn bråk》）为例。

对人类来说，这个问题可能很基础。

但对传统 RAG 来说，这显然这是个复杂的问题，因为我们的知识库，并不会存储这个问题的直接答案。

因此，我们需要先检索到两部电影的导演的信息，然后找到他们的出生日期，最后再比较哪个年纪大。

Deep Searcher 的搜索过程如下：

系统针对拆解后的子问题：

```

["Who is the director of God'S Gift To Women?", 'Who is the director of Aldri annet enn bråk?', 'What are the ages of the respective directors?', 'Which director is older?']
```

然后第一次迭代找到

```
The director of "God's Gift To Women" is Michael Curtiz, the director of "Aldri annet enn bråk" is Edith Carlmar
```

定位到两部电影的导演的名字。

第二轮的子查询包括：

```

["Find the birthdate of Michael Curtiz, the director of God's Gift To Women", 'Find the birthdate of Edith Carlmar, the director of Aldri annet enn bråk']
然后搜索到两位导演的出生年月，最后得到正确的回答。
The director of "God's Gift To Women" is Michael Curtiz, who was born on December 24, 1886 and passed away on April 11, 1962. He was a Hungarian-born American film director known for his prolific career in Hollywood. On the other hand, the director of "Aldri annet enn bråk" is Edith Carlmar, who was born on November 15, 1911 and passed away on May 17, 2003. Therefore, Michael Curtiz is older than Edith Carlmar.

```

而反观普通 RAG 的回答效果：

```

1. **Aldri annet enn bråk** (1954) is directed by **Edith Carlmar**, who was born on November 15, 1911. 
2. The related chunks did not provide specific details about the director of "God's Gift to Women." However, if we look at external sources for "God's Gift to Women," the film was directed by **L. M. (Lyman) Steinberg**, who was born on December 19, 1905.
By comparing their birth dates:- Edith Carlmar: November 15, 1911- L. M. Steinberg: December 19, 1905
**Conclusion**: L. M. Steinberg, the director of "God's Gift to Women," is older than Edith Carlmar, the director of "Aldri annet enn bråk."
```

由于一次性检索的内容不足，检索到的电影《God's Gift to Women》的导演无法推断，此时 LLM 幻觉猜测是另外一名导演，这直接导致错误的回答。



## **04** 



## **Deep Searcher VS 普通 RAG 定量对比**

在 GitHub 的 Deep Searcher 官方代码库里，我们已经提供了定量测试的代码。

在本文中，我们以 2WikiMultiHopQA 这个常见的数据集测试。（由于测试需要消耗大量 API token，这里只测试前 50 条数据，和全量数据测试相比会有一些抖动误差，但大致可以反映和说明问题。）

**（1）和普通 RAG 的召回率效果对比**

以 Max Iterations（最大反馈迭代）为横轴，以 Recall（召回率）为纵轴，下图是 Deep Searcher 的召回率和普通 RAG 召回率的对比。

![img](https://oscimg.oschina.net/oscnet/834344cd-47b7-4bb6-9f11-49bfc28047b7.png)

可以看到，随着 Max Iterations（最大反馈迭代）次数变多，Deep Searcher 召回效果越来越好。

但也可以看到，**随着迭代次数慢慢增加，边际收益越来越少**，说明反馈次数增加后，可能会达到一定的上限，继续反馈可能不太能得到更好的效果（因此，我们默认迭代次数为 3，您可以根据自身的需求进行调整）。

**（2）token 的消耗量**

我们以迭代次数为横轴，50 次总的 token 消耗为纵轴，绘制出下图：

![img](https://oscimg.oschina.net/oscnet/58bef625-4c96-48f3-856e-1c38474f5be0.png)

很明显可以看到，随着迭代次数的提高，我们可以看到，Deep Searcher 的 token 的消耗是线性地提升。如果按照 4 次来算，大约 0.3M 的 token 消耗，如果粗略按照 OpenAI 的 gpt-4o-mini 单价 0.60$/1M output token 来算，平均每次查询大约是 0.18 / 50 = 0.0036 美元的费用消耗。

如果换做推理模型，这个费用应该会数倍增加，包括推理模型本身的单价更贵，以及推理模型的输出 token 量会更多。

**（3）模型间的对比**

![img](https://oscimg.oschina.net/oscnet/99f6f922-6647-47ec-b5aa-2ebc17ce552a.png)

相比 OpenAI 官方推出的 DeepResearch，我们推出的 Deep Searcher 的另一大特点是可以自由切换模型。

在这里，我们也对各种不同的推理模型、非推理模型（如 (gpt-4o-mini)）进行了测试。

可以看到，Claude 3.7 Sonnet 的效果最好，不过也领先的也不多。当然，因为我们的测试样本量不多，所以每次测试的结果可能会有一些偏差。

但总体上看，**推理模型是比非推理模型强的。**

另外在我们的测试中，**更弱更小的的非推理模型有时由于指令跟随能力太弱，无法完成整个 agent 查询流程，所以也无法完成整个测试**，这个也是不少开发者在部署类似产品时经常会遇到的一个问题。



## **05** 



## **Deep Searcher VS Graph RAG**

Deep Searcher 的本质是 Agentic RAG，和 Graph RAG 有着很明显的区别。

Graph RAG 主要聚焦于对存在连接关系的文档展开查询，在处理多跳类问题上表现出色。

例如，当导入一部长篇小说时，它能够精准抽取各个人物之间错综复杂的关系。其运作方式是**在文档导入环节，就对实体间关系进行抽取。因此，这一过程会大量消耗大模型的 token 资源 。**

而在**查询阶段，不论是否是查询图中某些节点的信息，都会进行图结构的搜索，这使得这一框架不太灵活。**

反观 Agentic RAG，它的资源消耗模式与 Graph RAG 恰好相反。在数据导入阶段，Agentic RAG 无需执行额外特殊操作，而在回答用户提问时，才会产生较多大模型的 token 消耗。

![img](https://oscimg.oschina.net/oscnet/48222a70-471f-499e-8ca9-8c6b7336b543.png)

从查询模式的灵活性角度对比，Graph RAG 存在明显短板，它的查询模式较为固定，通常仅适用于单一关系的查询，难以根据多样化的需求进行灵活调整。

而 Agentic RAG 则展现出强大的灵活性。Deep Searcher 在不论是类似探究 “A、B 和 C 之间存在何种关系” 这样复杂的逻辑问题，亦或是撰写各类专业报告等任务，都能在查询阶段，依据问题的特性灵活调整路由策略，进行反复的反馈迭代思考，从而给出最为精准、全面的回答。

未来，随着大模型成本持续降低，推理性能稳步提升，像 Deep Searcher 这样的 Agentic RAG 凭借其突出的灵活性与适应性，极有可能在未来成为主流技术，并在实际应用场景中实现深度落地。

但这并不意味着全盘否定 Graph RAG 类产品，由于其在调整方面存在一定难度，后续或许更多地作为一个基础组件，融入到更为复杂的技术体系中，与其他技术协同发挥作用。



## **06**



## **Deep Searcher VS Deep Research**

与 OpenAI 的 Deep Research 不同，Deep Searcher 着重聚焦于对私有数据的深度检索与理解。借助向量数据库，Deep Searcher 能够接入多样化的数据源，整合不同类型的数据，并将其统一存储于向量数据库的知识库中。凭借向量数据库强大的语义搜索功能，Deep Searcher 可实现对海量离线数据的语义搜索。

另一方面，Deep Searcher 是一个完全开源的产品。毋庸置疑，OpenAI 的 Deep Research 在内容生成质量上，依然在业内处于领先地位，但对企业用户来说，每个账户除了需要每月需支付 200 美元之外，闭源的服务也就意味着其内部流程对用户而言是未知的。而 Deep Searcher，所有内部流程都是开源的，用户能够清晰了解其中的细节，并基于开源代码进行定制，或者将其部署到生产环境中。



## **07** 



## **一些思考与经验分享**

在整个项目的研发以及后续迭代过程中，我们有一些经验与心得分享： 

**推理模型很好，但绝非万能**。我们通过实验发现，推理模型做 agent 虽然效果好，但有时也会对一些简单的指令判断，做一大堆分析与反复思考确认，不仅消耗过多 token，回答速度还慢。这也印证了诸如 OpenAI 的大模型厂商，后续不再区别推理和非推理模型背后的逻辑，模型服务应该自动根据需求判断是否推理，以节省不必要的 token 开销。

**Agentic RAG 的爆发尽在眼前**。从需求侧看，深度内容生成是一个刚需场景；技术侧来看，提升 RAG 效果也是刚需。长期来看，影响 Agentic  RAG 爆发的唯一阻碍就是成本，但是随着 DeepSeek R1 这样的低成本高质量 LLM 出现，以及摩尔定律推动的芯片降本，推理服务的成本也会变得越来越低。

**Agentic  RAG 也存在隐形的 scaling law  “墙”**。实验中，面对一些复杂问题，我们一度尝试通过增加 token 数量来优化效果。但最终发现，随着 token 数量的增加，边际收益递减的现象逐渐显现，到了一定阶段，即便投入更多 token 进行反思和迭代，效果也难以得到进一步提升。

**传统 RAG 已死**：传统 RAG 的优势在于一次检索的低成本。但其对需求的响应还停留在非常基础语义理解 - 检索的阶段；但现实中，我们大部分的需求表达背后，都是有隐含逻辑的，并不能被一步检索到位（比如，如何在一年赚一个亿），必须通过推理 - 反思 - 迭代 - 优化来对其进行拆解与反馈，而以 Deep Searcher 为代表的 Agentic  RAG 技术方向，将是无可置疑的大势所趋。

希望这些思考能够帮助到更多想要探索 RAG 与大模型最新落地范式的开发者们，**如对以上案例感兴趣，或想对 Milvus 与 DeepSearcher 做进一步了解，欢迎扫描文末二维码交流进步。**