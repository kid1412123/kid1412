---
title: '(auxten)为什么我会感觉vibe coding让程序员越来越浮躁了？'
category: '/小书匠/收集/知乎问答/auxten/26827ea43acd624ce07f23df81df8aa6'
slug: 'https://www.zhihu.com/question/1951716962645288920/answer/2011798106161840829'
createDate: '2026-3-2 13:40:45'
grammar_mathjax: false
grammar_footnote: false
grammar_ins: false
emoji: 'a'
tags: 'LLM,程序员​,大语言模型,Cursor,vibe-coding'

---


[toc]


# 问题

提问者：**<a href="https://www.zhihu.com/people/niu-gao-da">扭高达</a>**
提问时间: 2025-9-17 18:39:45

管理层：无脑吹AI提效，大力鼓励程序员vibe coding

底层牛马：能完成需求就行，我也不在意他的实现，中小型公司也没有code review的习惯

# 回答

回答者： **<a href="https://www.zhihu.com/people/auxten">auxten</a>**
回答时间: 2026-3-2 13:40:45
点赞总数: 100
评论总数: 2
收藏总数: 129
喜欢总数：3

我在 ClickHouse 工作，公司内部非常鼓励用 AI 辅助开发——几乎所有主流 AI coding IDE（Cursor、Windsurf 等）都给员工开了不限量的版本。在这个环境下，过去几个月我用 Claude Code + Cursor 构建了一个 pandas 兼容的 API 层，底层跑的是 ClickHouse 的嵌入式引擎。项目需要对齐两套完全不同系统的 600+ 个方法，前后烧掉了大约 $20K 的 token 费用。这种大面积、高重复的对齐工作，理论上正是 AI 擅长的。

以下是我的一些自我总结：

 **1. 规则写进文件，不要只存脑子里** 

AI 没有跨 session 记忆，今天教会它的东西明天全忘。我们最终在 repo 里维护了一个规则文件（CLAUDE.md）——AI 反复犯的错、禁止的捷径、已经定下来的架构决策，全写在里面。

这个文件还意外成了团队协作的接口。不然每个人各自调教 AI，经验全是 local 的，谁也看不到。

不过有个提醒：每次你往 CLAUDE.md 加一条新规则，问自己一句——我是不是在给 iPhone 加按钮？乔布斯当年坚持 iPhone 只有一个按钮是有道理的。规则文件膨胀到 500 行，AI 就会像用户跳过用户协议一样开始无视它。保持精炼，定期裁剪。

 **2. 早期看推理过程，不要只看产出** 

项目初期，看 AI 怎么想的比让它快速出活更重要。当它的逻辑开始偏离你的预期，问自己两个问题：是我原来的想法就有问题？还是我没表达清楚？两种情况都经常发生，需要完全不同的应对方式。

 **3. 定期用零记忆的 AI 做独立审视** 

你和日常合作的 agent 会形成共同盲区。我们开始定期用一个全新的、没有任何项目上下文的 agent（claude.ai/code，注意不是 Claude Code CLI），让它以批判、理性的外部视角审视我们的工作。

两个关键词： _批判_ ——覆盖 AI 默认的讨好模式，明确要求它找问题； _理性_ ——要结构化推理，不要感觉，具体哪里有问题、为什么、证据是什么。以现在前沿模型的能力，设对 framing 就够了，不需要过度工程化 prompt。

反馈往往不舒服但很有价值：你已经习以为常的报错信息其实很 confusing，你觉得 OK 的 API 其实不一致，文档缺失对新人来说一目了然——而你的日常 agent 永远不会告诉你这些，因为它已经学会绕过去了，跟你一样。

 **4. 用目标系统本身作为测试 oracle** 

我们的目标是兼容一个已有 API，所以根本不需要发明测试用例。直接找真实代码（GitHub/Kaggle 上的 notebook），换一行 import，对比输出。如果你也在做任何形式的兼容层，这个思路能省巨大的工作量。

 **5. 规则比 prompt 重要** 

观察 AI 走捷径的模式，然后写成明确的禁令。我们最好的例子：测试因为行顺序不一致而失败？AI 最爱的操作是加个 .sort\_values() 让测试通过。这不是修 bug，是遮盖 bug。我们明确禁止了这个行为。真正无法兼容的 case 标记 XFAIL，绝不允许静默跳过。

通用模式：观察 AI 的偷懒路径，逐条封堵。

 **6. 多 Agent 流水线：文件系统 > 对话历史** 

我们用 Python 脚本编排多 agent 工作流，文件系统作为 agent 间的共享上下文——每个 agent 把工作记录写到 tracking 目录，下一个 agent 自己决定要读多少历史。比把对话历史塞进 prompt 灵活得多。

几个管用的模式：角色分离、结构化决策（APPROVE/REJECT/ESCALATE 输出为 JSON，让控制流确定性而非依赖自由文本解析）、失败时自动 git 回滚。

  

原文地址：[(auxten)为什么我会感觉vibe coding让程序员越来越浮躁了？](https://www.zhihu.com/question/1951716962645288920/answer/2011798106161840829) 


