# OpenAI的Codex上线

时间：2025-5-17

使用：ChatGPT Pro、Team 和 Enterprise 用户可使用 Codex，Plus 和 Edu 用户也很快可以上线使用。



**OpenAI的在线微调分为：**

1、2023年的SFT，即LoRA

简单讲，它就是把“你自己租 GPU 跑 LoRA”这件事搬到了 OpenAI 的云端

2、2024 年底 OpenAI 发布的“新在线微调”其实是强化学习范式

• 预热阶段：先用极少量（甚至 12 条）带答案的数据做一轮 SFT，让模型学会输出思维链（Chain-of-Thought）。

• 强化学习阶段：不再用“正确答案”做交叉熵，而是让模型对同一问题反复采样多条推理路径；由用户提供的“评分函数/Grader”给出 0/1 或连续奖励，然后用 PPO 在线更新策略。

