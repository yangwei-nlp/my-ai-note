## Claude 3.7 Sonnet

* 时间：2025年2月25日

* 参数：3640 亿参数，128K上下文

* 优点：

  * Claude 3.7 Sonnet 在指令遵循、一般推理、多模态能力和代理编码方面表现出色，扩展思维在数学和科学方面提供了显著的提升。

* 特点：

  * **Hybrid Reasoning混合推理模型**，新的“**扩展推理模式**”允许Claude在回答问题之前进行更深入的思考，从而增强了其应对复杂任务的能力。

  * 在提示词的设计过程中融入了更多“AI个性”设定，**使得Claude听起来更加人性化**，能够贴近用户需求；比如在与用户的交互过程中，**Claude表现出了更强的主动性。它不仅能够按用户的指令作答，还能够主动提出相关话题，甚至引导对话的走向**。这种能力，相较于Claude 3.5的被动回应显得格外突出。

    * 其对话题引导的能力，使得Manus Agent主要使用Claude3.7作为底层基模

* 附录：

  * https://www.anthropic.com/news/claude-3-7-sonnet





# 查看机器的IP位置

* 命令行

```shell
curl -s -4 --location --request GET 'https://www.cloudflare.com/cdn-cgi/trace' |grep loc
```

* 网页版

https://www.ipaddress.my/
