---
title: "2025–2026 国内大厂 Agent 面经：从 RAG 到 Harness Engineering"
description: "Agent 面试文章与工程学习笔记合编，涵盖岗位画像、能力地图、工程实践和学习路线。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Agent", "面经", "转载"]
---

> **转载说明：** 本文作者为 Liuliming，原文见 [2025–2026 国内大厂 Agent 面经：从 RAG 到 Harness Engineering](https://keepkeen.github.io/blog/china-agent-interview-guide-2025-2026/)。此页面依据粘贴文本整理；原文中的部分链接未包含在粘贴内容中，查阅具体出处请访问原文。

检索截止：2026-08-30。本文覆盖公开的一手面经、官方招聘/JD、公司技术文章和两份结构化面经索引，并结合本项目对 Pi、OpenAI Codex、DeepSeek Harness、Claude Code 的源码分析。它是一份准备指南，不是任何公司的官方题库。



## 先说结论

2025 到 2026 的变化，不是 RAG、Transformer 和后端基础突然不考了，而是面试官开始把问题继续向模型外面的控制系统推进。


2025 年公开样本的共同底座主要是：Transformer/Attention、LoRA/SFT/DPO/PPO/GRPO，RAG 的切块、Embedding、混合检索与 Rerank，ReAct、Function Calling、LangChain/LangGraph，以及项目深挖、算法和数据库/缓存/消息队列。2025 年第四季度已经有少量深题进入 Claude Code/Cursor 上下文压缩、Reflection、工具失败归因和多 Agent；因此这里不是“2026 突然发明新范式”的断崖式切换。


2026 年这些内容仍然存在，变化在于下面这些主题开始跨公司、跨岗位反复出现，并被当前 JD 明确写成 Runtime/Harness 能力：


- Agent Loop 与 Harness/Orchestrator 的边界；
- Prompt Engineering、Context Engineering、Harness Engineering 的演进；
- Coding Agent 的上下文装配、压缩、Checkpoint、恢复和路径震荡；
- Skill、Tool、Function Calling、MCP、A2A 与 Subagent 的边界；
- 工具越权、Human-in-the-loop、Permission、Sandbox、凭证隔离与审计；
- 多 Agent 的共享状态、并发、取消、预算和文件冲突；
- Outcome、Trajectory、Tool、Safety、Latency、Token/Cost 的分层 Eval；
- Claude Code、Codex、DeepSeek Harness、OpenClaw、Hermes 等真实系统的架构或源码比较；
- AI Coding/Vibe Coding/Spec Coding 作为口头题，甚至直接进入现场编码或正式机考。

因此，2026 年的分水岭已经从“会不会用框架搭一个 Demo”，移到“能不能解释控制面、失败路径和验证证据”。但另一个同样重要的结论是：传统工程基础没有消失。京东、华为、美团、腾讯、蚂蚁、拼多多等样本仍在同一轮里考 Java/Go/Python、并发、MySQL/Redis/MQ、网络、操作系统和手撕算法。


还有一条比“厂味”更可靠的规律：岗位类型通常比公司名称更能预测题目。 同一家公司里的 Agent 应用开发、Agent 算法/RL、Agent Runtime/Harness，题目差异往往大于不同公司同类岗位之间的差异。



## 一、证据怎么筛


### 1. 证据等级


| 等级 | 来源 | 能证明什么 | 不能证明什么 |
| --- | --- | --- | --- |
| A | 公司官网、官方 JD、官方技术文章 | 当前岗位、产品和技术方向 | 不能单独证明某道题真的问过 |
| B | 第一人称面经，有岗位、轮次、日期/时长和自然的个人复盘 | 该候选人在该团队的大概率真实经历 | 不能外推为全公司统一题库 |
| C | 保留原帖线索的汇总、开源结构化样本、可信转载 | 发现重复主题、补充检索入口 | 不能与原帖重复计数，也不能替代一手来源 |
| D | 卖课/内推引流、SEO 拼接、“每题标准答案”、标题正文公司不一致、时间矛盾 | 只作为搜索线索 | 不进入公司画像和趋势判断 |

本文特意排除了多类看起来很“完整”、实际不可靠的材料：同一账号连续发布多家公司“被问麻了”且每题附标准答案；把 2025 内容重新包装为 2026；标题写拼多多、正文却写快手；含博彩/广告域名；以及无法回溯原始面经的题库文章。



### 2. 可用的结构化旁证

开源项目 smile-struggler/kaomian 披露的快照覆盖 2025-03-22 至 2026-04-18：抓取 767 帖，保留 248 帖，抽取 948 个问题并归并为 638 个 canonical question。它的高频项依次包含 Agent 评估、Agent 难点、场景流程设计、指标构建、多智能体协作、上下文超窗、工具参数幻觉、记忆、工具失败/超时与 Skill。


这组统计有价值，但不能当成无误的真题数据库：流水线使用 Qwen3-VL 做筛选、OCR、结构化抽取和语义归并，仓库又没有公开全部原始帖子。因此，本文只用它证明“多个独立样本反复触及哪些主题”，不据此计算公司的精确出题率。


另一份 agent-interview-hub 在 2026-08-13 生成了 51 条候选索引，含来源链接或小红书站内检索提示。本文把它当作导航，并回到可访问原帖交叉核对。



### 3. 这份面经的边界

公开样本明显偏校招、实习和一面，社招高级岗位、主管面与内部转岗样本更少。

牛客很多页面只显示月日。无法从帖子、评论和招聘届次共同确认年份的条目，不用于年度趋势核心判断。

同一公司不同 BG、业务线甚至面试官的差异非常大。下文的“公司倾向”都是有限样本归纳。

源码可以证明实现机制，不能直接推出成功率、模型质量、成本或 benchmark 排名。

Claude Code 的详细实现只以本项目验证过的 npm 2.1.88 source map 为证据；2.1.251 已是 wrapper + 原生二进制，不能把旧实现无条件外推到最新版。


## 二、先按岗位分型，再看公司


### 1. Agent 应用/全栈开发

典型任务是客服、知识问答、投研、数据分析、办公、营销、运维和企业数字员工。面试重点通常是：


- 为什么需要 Agent，而不是一次 LLM 调用或固定 Workflow；
- RAG 全链路、意图识别、Prompt/Context、Tool Calling；
- SSE/WebSocket、异步任务、数据库、缓存、MQ、限流与幂等；
- 项目是否真的上线、用户是谁、失败成本是什么、指标如何得到；
- 幻觉、超时、重试、人工接管、降级和线上排障。


### 2. Agent 算法/后训练

这类岗位不是“会调 API”即可。常见深挖包括：


- Agent trajectory 与普通 SFT 语料的差别；
- 工具调用、环境 Observation、最终回答的 loss mask；
- SFT、DPO、PPO、GRPO、RLVR 何时使用；
- reward/verifier、credit assignment、reward hacking；
- 数据合成、正负样本、长度偏差、训练—服务分布偏移；
- 规划、搜索、Memory、工具选择、长程任务与 test-time scaling。


### 3. Agent Runtime/Harness/Infra

2026 年增长最明显的一类。面试会把“模型能不能回答”转成“系统能不能稳定执行”：


- Agent Loop、Task/Turn/Step 状态机和 Early Stop；
- Context assembly、Compaction、Checkpoint/Resume、Event Log；
- Tool registry/router、动态发现、并发门、结果规范化；
- Approval、Permission、Sandbox、凭证、租户与网络/文件边界；
- Runtime、模型网关、流式、语义缓存、熔断与故障恢复；
- Trace/Metric/Log、回放、Eval、成本和容量规划。
- 京东当前官方 Agent 专家 JD 直接列出 Agent Runtime、上下文状态、工具/插件注册、Streaming、Semantic Cache、熔断、Eval/回放与全链路 Trace；百度官方云原生 Agent Infra 岗则出现 K8s、gVisor/Firecracker、Skill/MCP/Plugin、Harness 与 OpenTelemetry。这类岗位本质上仍是分布式系统和平台工程，只是负载变成了概率模型驱动的长链任务。



### 4. Coding/GUI/Browser/Computer-use Agent

常见问题会进一步落到环境和产物：


- 如何选择代码上下文，grep、AST/符号索引、向量检索分别做什么；
- 仓库规则、Prompt、Skill 与工具描述怎样进入 Context；
- 代码编辑、测试、Review、回滚和工作区隔离；
- 长任务如何续跑，多 Agent 如何避免同时修改同一文件；
- 浏览器/桌面 Agent 如何处理本地文件权限、截图/DOM、多模态与不可逆操作；
- AI 生成代码怎样验证，什么由人负责。


## 三、公司画像：公开样本里更常追什么


### 字节跳动

2025 年代表样本仍以 RAG 检索/排序、Prompt 评测、ReAct、LoRA/SFT、Attention、幻觉、项目指标和算法为主。2025-04-28 的四轮面经 很典型：Leader 会继续追为什么不用闭源 SOTA、为什么尚未上线、Precision/Recall 怎么算。


到 2026 年，字节是公开样本里最早、最密集直接追 Coding Agent 控制面的公司之一：


- 2026-08-13/16 的 Agent 实习两轮 问 ReAct/Plan、上下文溢出与保真压缩、路径震荡、Context 与长期 Memory、工具描述过多、Prompt Cache、多 Agent 通信和同时改文件；
- 2026-06-04 一面 问三级压缩、SDD/Spec、Harness 模块和评测集过拟合；
- 2026-07-01 中国交易与广告一面 直接比较候选人项目与 Claude Code、Codex、Hermes，继续追 Harness 取舍、Skill 加载、三层记忆、Context 拼装和 Checkpoint；
- 2026 年社招多轮记录 还出现 OpenClaw/Hermes 源码、A2A 死循环、Subagent 幻觉、共享状态、Memory/RAG 与实际用户收益。
- 但不要因此忽略基础：同一批面经仍有 Transformer、RL、Python、数据库、浏览器链路和算法。字节官方 Seed 招聘也把 Code Agent RL、Long-Horizon Task、Multi-Agent RL、Memory/Search 与模型/系统训练同时列出。Seed Early Career


准备重点： 项目失败轨迹、上下文与压缩、Skill/Tool、长任务、Eval、多 Agent 冲突；同时按岗位补齐 RL 或后端基础。



### 阿里巴巴、淘天、阿里云与阿里国际

阿里内部业务差异很大，按集团名准备会失焦。


阿里云/瓴羊更偏 Runtime 与企业工程。2026-08-27 Agent 算法一面 直接问 Agent Loop、Harness 安全/权限、长任务 Early Stop 和 DeepSeek Harness；2026-04-10 阿里云面试 同时考 Workflow/Agent、跨机通信、RPC，并让候选人现场用 AI Coding 完成功能后人工 Review。

淘天/淘宝闪购更重业务落地、研发提效和真实故障。2026-04-30 淘宝闪购 Agent 算法一面 追框架与部署、HITL、风险分级、熔断、沙箱、审计、跨周 Memory、反馈/RL、Tool Schema 和线上 AUC 下降；2026-08 的多份样本又出现 CR Agent、Meta-Harness、DeepSeek Harness、多 Agent 测试、MQ 幂等、OOM 和算法。

千问/夸克通常是模型与应用双栈。2026-08-25 千问应用开发 问文档切块、BM25+向量、多跳、Query Rewrite、ACL/PII、长对话摘要与 Harness 数据流；算法岗还会继续进入数据合成、后训练和模型选型。

阿里国际/Accio公开社招样本更强调云端环境、Search、Worktree、Sandbox、Harness Gate、Eval/Benchmark、Context 隔离/压缩和 Token ROI。

2025 的阿里云 AI 平台面经仍以 RAG 整体链路、Transformer、Embedding、MRR 与多路召回为主；2025 下半年的淘天/夸克开始出现工具冲突、超时降级、Memory、Prompt A/B 与 RL，但 2026 才把 Harness、权限和长任务明确推到台前。


准备重点： 先确认 BU。阿里云准备 Runtime/权限/可观测；淘天准备业务链路、AI Coding、并发与指标；千问/夸克准备模型—应用双栈；阿里国际准备搜索、云沙箱、长任务与成本。



### 蚂蚁集团

蚂蚁的辨识度是“可信执行 + 后端基本功”。2025 蚂蚁国际面经已在问 RAG vs Fine-tuning、Rerank/NDCG、Memory、Agent Eval、MCP/A2A；到 2026，安全与全自动链路被问得更具体：


- 2026-05-20 智能体/LLM 应用一面 问 Skill、Spring AI、Claude Code/Harness 源码、grep vs RAG、混合召回，再转入网络、MySQL、Spring 和 AI Coding；
- 2026-08-29 蚂蚁财富 Agent 开发 问 Markdown Memory/RAG、Agent Teams、全自动任务可靠性、FC 内部流程、Harness、Skill 懒加载、Temperature=0 的非确定性，同时考 ACID、数据库日志、Redis、MQ、限流和算法；
- 算法轨还会深入 DPO/PPO/GRPO、FlashAttention、蒸馏、数据和金融信用建模。
- 准备重点： 权限、审计、隐私、HITL、Prompt Injection、不可逆操作、资金/状态一致性、幂等/补偿/熔断；不能只会说“加个人工确认”。



### 腾讯

腾讯样本显示出“模型适配 + Agent 应用 + 传统大规模后端”的混合特征。


- 2026-04-04 AI Agent 应用开发一面 直接问为什么已有 Claude Code 还要自研 Coding Agent、产品差异、分层 Context、Summary 质量、Subagent 通信与防循环，再进入 GRPO/PPO、QLoRA、量化和 Python；
- 2026-04-10 两轮样本索引 包含 Qwen 工具调用适配、空调用、长短期 Memory、代码助手仓库依赖检索、Text2SQL、Rerank 变差的诊断、Query Rewrite 与多 Agent；
- 2026-05-21 一手样本 把 Prompt Injection/Agent 安全与 Redis/Kafka 容灾、亿级表变更放在同一轮；
- 2026-07-07 跨公司面经 还提醒了一个现实风险：候选人投的是 Agent，腾讯实际团队调整后可能几乎只考云基础设施和算法。
- 2025 的混元算法样本更偏 SFT 动机、数据标注、Qwen 推理模式、RoPE、量化和 baseline。2026 当前招聘则已经细分出 Harness、Eval Infra、Agent RL 框架、Sandbox 和轨迹观测方向。


准备重点： 先问清团队。应用岗准备 Qwen/工具/记忆/RAG；平台岗准备协议、仓库理解与后端；算法岗准备数据、微调/RL 和模型基础。



### 百度

百度是 2026 年公开样本中把“Prompt → Context → Harness”问得最直白的一家。


- 2026-07-02/03 文心一、二面 一面偏 Agent 后训练：SFT 何时足够、reward/verifier、reward hacking、PPO/GRPO 和 task eval；二面偏系统：Action、长文本、Context/Memory、冲突指令、Harness/Loop、Skill 为什么在系统层，以及 Claude Code/Codex/GLM；
- 2026-08-18 大模型研发一面 直接问 Prompt/Context/Harness Engineering、上下文压缩、Claude Code 和从零 RAG；
- 2026-08-21 二面 几乎改成开放式系统题：C 端 Agent 模块、主流产品比较、Claude Code 数据流、防重复 Tool Call、只有 LLM API 与 VS Code 时如何造 Agent；
- 官方 2027 AIDU 智能体算法岗 同时列出感知—决策—执行、多 Agent、长期记忆、规划/工具/反思/代码、RAG+Agent，以及成功率、延迟、成本和用户体验 Eval。
- 准备重点： 一定先区分应用、算法和 Infra。应用要全栈与 RAG，算法要 reward/verifier/trajectory，Infra 要 Harness、Sandbox、K8s 和可观测。



### 华为

华为公开样本的特点不是最追新名词，而是流程正式、基础扎实、项目必须说清楚。


- 2025 AI 工程师实习 已问 RAG 构建、分块、Query/知识歧义、Agent 模块/选型、边缘设备模型规模和算法；
- 2026-05-14 云软件 AI 技术应用 有双机位、三数之和、项目架构/个人贡献，主管面才问 Agent 宏观架构与从零设计；
- 2026 某 BG Agent 岗完整记录 同时验证 ReAct/RAG Agent、TCP/C/存储/拓扑排序、Computer Use、Skill/MCP，并建议研究主流 Agent 框架和 Claude Code 实现；
- 华为官方挑战课题 已出现“AI Agent 多模态智能通信”，华为云公开方向也包含 Agent 平台、Skill/CLI 和 Agentic Infra。
- 准备重点： 机试/手撕、C/C++/OS/网络、项目真实性、端云部署和业务价值优先；Harness/MCP/Skill 是加深项，不是基础差的替代品。



### 美团

美团最稳定的追问方式是：为什么这么选、替代方案是什么、如何验证、剩余坏例怎么发现。


- 2025 后端暑期样本已经把三个 Agent 的输入输出、RAG 消融、BM25、双路召回、Tool Calling、MCP 的 SSE/stdio、上下文过长与 MySQL/Redis/线程池放在同一轮；
- 2026-08-25 Agent 开发一面 是很有价值的样本：RAG vs 长上下文直接读文件、RAG vs Coding Agent 用 grep 搜仓库、无标题 Markdown 切分、短 Query/长 Chunk、如何发现剩余 1%–2% 幻觉、单/多 Agent、Plan-and-Execute/ReAct、Agent vs Harness、Context 裁剪、Prompt Cache、Milvus、协程/线程、Vibe/Spec Coding 和两道算法；
- 2026 校招企业页有多名候选人独立报告“大模型选择题 + 算法 + AI Coding”；
- 美团官方 CatPaw 当前产品强调本地/云端长任务、专家/Skill/Subagent、跨会话 Memory、多 Agent 独立环境、凭证隔离、权限、Sandbox 和全链路可观测。这不能当作真题，但能解释岗位为何围绕这些机制继续追问。
- 准备重点： 每个架构选择必须带 baseline、指标、失败样本和替代方案；RAG 不能只背流程，要能说明什么时候不该用 RAG。



### 京东

京东的公开证据最像“后端/分布式系统 + Agent Runtime”。


- 2025 样本已问用户意图多样化、工具选择准确率下降、RAG 召回低和真实应用体会；
- 2026-02-10 实际 Agent 开发一面 从告警 Agent 的输入输出、节假日/发布期准确性和非 AI fallback，问到 4000 页多模态/GB 文档、RAG 更新、LangChain 链路、MCP 工具选择和 Skill；
- 2026-03-13 京东零售后端 把 Redis+Lua、MQ、性能与 RAG 分片/幻觉、Function Calling/Skill 和模型选型混合考；
- 2026-06-24 官方 Agent 专家 JD 明确要求 Model Gateway、Agent Runtime、上下文状态、工具/插件注册、Streaming、Semantic Cache、熔断、Eval/回放和 Trace/Metric/Log；
- 另一条官方大模型智能体岗位 进一步把 Skills、Subagents、MCP/A2A/A2UI、Coding Agent、Sandbox、Memory、Context Engineering 与 Harness Engineering 写进职责和能力要求，说明这些不只是社区热词，而是当前平台岗位的正式能力项。
- 准备重点： Java/Go、RPC/MQ/网关、并发、缓存和故障恢复不能丢；Agent 题按“状态、权限、fallback、Eval、可观测”回答。



### 快手

快手样本一条是后训练/模型，一条是 Data Agent/应用，不能混在一起。


- 2025-09-15 大模型二面 深入 PPO/DPO/GRPO、GAE、reward/reward hacking、分层强化学习 Agent、FSDP/DeepSpeed、显存和手写 MHA；
- 2025 Data Agent 一面 仍以 LLM vs Agent、ReAct/LangChain、RAG 与模型微调为主；
- 2026 Data Agent 一面 已继续追父子索引、BM25/Rerank、Context vs Memory、Function Calling/Planning、Prompt Injection、工具安全、限流与 RAG Eval；另一份问题密度更高、但需要保守使用的同类记录 又出现三层 Memory、Skill 渐进披露和工具选择。这里更可靠的结论是题面从“会搭框架”向检索、记忆、安全和评估深化，而不是把所有问题外推成快手统一题库。
- 准备重点： 应用岗把 RAG/Memory/Security 与限流/缓存/数据库一起准备；算法岗另开一条 RL/训练/显存路线。



### 拼多多

拼多多 2026 校招公开出现独立 AI Agent 研发方向，但当前可高置信核验的一手样本不够多，不能像字节、阿里那样概括稳定“厂味”。


- 2026-08 提前批一面短帖 主要问 LLM API 失败如何感知和定位、限流之外的异常、Context/Memory 较弱、多用户×多角色记忆，以及算法；
- 企业面经页还有代码生成全链路、断点恢复、Codebase Memory、代码审查和长耗时 Agent 资源管理等记录，但部分条目无法稳定取得独立原帖，只作二级线索；
- 网上同时存在标题写拼多多、正文写快手，或“含完整答案/专栏付费”的内容，均未用于公司画像。
- 准备重点： 以 JD 和自己投递团队为准，优先准备 LLM/Tool 故障定位、会话与多角色 Memory、分布式基础和算法；Coding Agent/DeepSeek Harness 只在能够说明来源和实现时展开。



### 小米

小米的业务方向决定了端侧/OS/GUI Agent 值得重点准备，但当前可核验的 2026 技术一手面经仍较少，下面是“岗位方向 + 有限样本”的组合判断，不代表稳定题库。


- 2025 大模型应用样本偏 RAG 指标、低延迟、Prefill/Decode、KV Cache、Python/GIL；
- 2026 AI Agent 暑期一面 追“为什么代码生成要微调，不能只用 Prompt/Spec Coding”、缺陷修复 Agent、静态 Spec 与动态环境反馈、模型选型、自托管/API 和 Memory；
- 普通汽车软件岗已经开始问 AI Coding、Skill、如何维护 CLAUDE.md 和 Loop；
- 小米官方顶尖人才课题 明确列出“认知—记忆—问答链路及自动评估”“AI Agent 与操作系统深度集成”和端侧具身智能决策。
- 准备重点： 微调 vs Prompt/Spec、GUI/OS 权限与状态、端侧模型/推理、Memory、Sandbox/Trace，以及真实代码验证。



### 小红书

公开样本数量不如前几家稳定，但时间演进很有代表性：2025-10 的二面 已经问 Skill 召回 vs Function Call/RAG、混合检索，以及 Cursor/Claude Code 的 System Prompt、上下文压缩和工具选择；2026-08-06 Agentic 全栈一面 则继续进入写操作边界、Harness 层、编排/意图分流、Skill vs Tool、多模型一致性、十几万请求扩容、LangGraph vs 自研和 Java/MySQL。产品工程样本还会问多 Agent Context 共享、Skill 管理和业务指标。


准备重点： 内容/电商场景、检索与推荐数据、Agentic 全栈、状态/Checkpoint、Skill、模型一致性和服务扩缩。证据不足时不要套用“某厂固定风格”。



### 国内模型公司：阶跃星辰、智谱、月之暗面、MiniMax

模型公司公开样本更少，但岗位分化更鲜明。


阶跃星辰： 2026 企业面经页 中的 Agent 算法一面会追工具调用训练的 reward、数据配比、ReAct 的 Plan/Observation/Action 输入输出、TRL/verl 类和参数；另一场问 LoRA 初始化、DPO 数据、SFT/DPO、Workflow 与采样。它更像 Agent 后训练岗，而不是应用框架岗。

智谱： 2026 Agent 算法面经 追 DPO loss、负样本、长链 GUI 正负样本长度、GRPO、Tool Response 是否 loss mask、MCP 训练数据、bad case、参数量/KV Cache 和流式 Tool 输出处理。

月之暗面/Kimi： 可信公开样本较少。2026-07-23 Desktop AI-Native 研发 主要问本地文件权限、产物面板、多模态 Context、长任务通知/回看/续跑、结果编辑—确认—发布和 Agent 任务中心。网上多篇“被吊打+完整答案”的 Kimi 面经模板化明显，不能作为核心证据。

MiniMax： 2026 已有几份较自然的一手样本。AI 应用开发一面 追 PRD/design 如何防需求丢失、需求到代码定位、Task 粒度、多 Agent 状态传递/冲突，以及 Rules vs OpenClaw Memory；同一候选人的平台研发一面和二面 又进入 Codex 数据采集耐久性、MQ、Agent 自进化、用户反馈有效性、协议扩展和平台能力产品化。官方 Agent Team 技术文章 可用于校准长程任务、多 Agent 并行和质量门禁，但仍不是面试题证据。

如果目标是 DeepSeek、MiniMax、Kimi、智谱、阶跃一类模型公司，至少要另外准备模型/后训练/推理与研究复现；只读 Agent 框架源码并不够。



## 四、一场 Agent 面试通常怎样推进

公开样本不能推出统一轮次，但问题的推进方式相当稳定。



### 第一层：确认项目是不是你做的

面试官先问背景、用户、输入输出、整体数据流、个人代码边界和实际结果。随后故意绕开简历里的框架名：


- 用户一句话进入系统后，第一个写入状态或日志的组件在哪里？
- 模型为什么选择这个工具，Schema 谁定义，错误码如何返回？
- 你说准确率提升，基线、样本量、统计窗口和归因是什么？
- 为什么不用 Workflow、规则、小模型或一次 API 调用？
- 哪个线上 bad case 最难，最终确认根因的证据是什么？

这一层不是考架构词汇，而是检查“输入、状态、动作、观察、指标”能否连成可追踪的链。



### 第二层：沿失败路径追问

项目讲顺后，2026 年面试官经常把系统推到异常状态：


- 工具超时、返回空值或部分成功怎么办？
- 同一个 Action+参数重复五次，在哪里停止？
- 压缩后丢掉用户硬约束，如何发现和恢复？
- 两个 Agent 同时改一个文件或写同一行数据库怎么办？
- 高风险工具被 Prompt Injection 诱导调用怎么办？
- 进程在第七步崩溃，能否从 Checkpoint 恢复而不重复副作用？

优秀答案不是堆“重试、熔断、降级”四个词，而是说明：哪些错误可重试，哪些操作幂等，状态写在哪里，副作用如何确认，恢复从哪个边界继续，什么时候必须交给人。



### 第三层：要求你比较，而不是背定义

常见比较题包括：


- Agent vs Workflow；ReAct vs Plan-and-Execute；
- LangChain vs LangGraph vs 自研 Loop；
- RAG vs 长上下文直接读文件 vs grep/代码索引；
- Function Calling vs Tool vs MCP vs Skill；
- Context vs Session vs Memory；
- Permission vs Sandbox；
- 单 Agent vs Multi-Agent；
- Claude Code vs Codex vs 自研 Coding Agent；
- Prompt/Spec 工程 vs SFT/DPO/GRPO。
- 回答结构最好固定为：目标约束 → 两种机制 → 适用条件 → 失败模式 → 自己项目的选择与证据。



### 第四层：基础与现场题

Agent 岗的“基础”由岗位决定：


- 应用/后端：Java/Go/Python、协程/线程、MySQL、Redis、MQ、RPC、限流、SSE/WebSocket；
- 算法：Transformer、Attention、KV Cache、显存、LoRA/SFT/DPO/PPO/GRPO、reward；
- Infra：OS、容器/K8s、隔离、网络、调度、存储、可观测和分布式一致性；
- Coding/GUI：代码仓库、IDE/CI、工作区、浏览器/桌面权限和流式 UI。

手撕仍常见，难度从 Hot100 到 DP、图、并查集不等。2026 还出现了两种新形态：让候选人用 AI Coding 完成功能后 Review，以及正式笔试里的 AI Coding 题。面试官真正看的是需求拆解、上下文提供、验证、回退和代码责任，不是提示词写得多华丽。



## 五、结合四个源码仓库，回答 12 类高频硬题


### 1. Agent、Workflow、Orchestrator 和 Harness 有什么区别？

可以按控制范围从小到大回答：


- **Agent Loop** 是局部迭代：模型基于当前 Context 选择 Action，环境执行并返回 Observation，系统继续或停止。
- **Workflow** 是开发者预先定义的状态转移、分支和失败处理；模型可以是某个节点，但不是全部控制逻辑。
- **Orchestrator** 负责计划、路由、调度、依赖与并发，可能编排一个或多个 Agent/Workflow。
- **Harness** 是模型外完整运行与治理环境，除了 Loop/Orchestrator，还包括 Context、Tool Registry、Policy/Approval/Sandbox、Session/Compaction、Workspace/Subagent、Telemetry/Eval 和恢复。

本项目四仓正好给出四种控制权分配：


- **Pi** 的生产主链很薄，把大量控制交给 provider 与扩展；核心没有内置 Permission、OS Sandbox、MCP 或 Subagent Scheduler。
- **Codex** 把控制放在 app-server、Session/Task/turn、Tool Registry/Router、安全状态机、ThreadStore 和共享多 Agent 控制面。
- **DeepSeek Harness** 把控制放在 Cordis 插件组合与持久事件面，Loop 薄而 Session/Event 很重。
- **Claude Code 2.1.88** 把产品能力收进共享 async-generator query()，再叠加工具验证、Hook、Permission、可选 Sandbox、压缩与 AgentTool。

详见Agent Loop、模型调用与工具执行和功能总矩阵。这比把 Harness 解释成几份 Markdown 文件更准确。



### 2. Context、Working State、Session、Transcript 和 Memory 怎么分？

- **Context**：下一次模型调用真正能看到的 Token/消息表面。
- **Working State**：任务当前目标、计划、未完成步骤、工具状态、产物句柄等结构化状态。
- **Session/Thread**：一次任务或对话的生命周期、身份与可恢复边界。
- **Transcript/Event Log**：持久记录用户、模型、工具、审批和系统事件。
- **Long-term Memory**：经过写入门控、作用域、冲突与过期治理后，跨会话按需召回的事实或经验。

关键句是：磁盘上有完整 Transcript，不代表模型当前 Context 完整可重放。 压缩、过滤、角色映射、工具结果裁剪都可能改变模型表面。四仓分别采用会话树、ThreadStore、事件投影和 JSONL 父链，差异见上下文、会话、压缩与子代理。



### 3. 上下文快超窗时，怎样设计压缩？

先定义不可丢失集合，而不是先让模型“总结一下”：


- 固定保留系统/项目规则、权限边界、当前目标、用户硬约束和输出契约；
- 最近交互、未完成步骤、尚未确认的工具副作用保留原文；
- 已完成阶段压成结构化字段，例如 decisions/completed/open/artifacts/risks；
- 原始消息、日志和大文件留在外部事件/对象存储，摘要只保存可回读的 ID/句柄；
- 按 Token 软/硬水位、阶段完成或目标切换触发，而不是机械地“每五轮总结”；
- 用回放集比较压缩前后约束保留、下一步工具选择和终态成功率。

源码里，Codex 把 Compaction 当作 Loop 内状态转换；DeepSeek Harness 是持久事件不删、模型 Surface 被替换；Claude Code 2.1.88 有 microcompact、collapse/full compact 的分层路径；Pi 在会话分支上记录摘要。不要把任何一个旧版本的具体阈值说成行业通则。



### 4. Tool 调用链怎样做到生产级？

一个完整回答至少包括：


发现/目录 → Schema → 模型选择 → 参数验证 → Policy/Approval → Sandbox → 执行 → 结果规范化 → Observation → Trace/Eval


进一步说明：


- 工具名和描述解决“模型能否选对”；Schema 与验证器解决“参数是否合法”；
- Approval 决定是否授权，Sandbox 限制授权后实际触达范围；
- 只对幂等或有明确补偿的操作自动重试，指数退避不能替代副作用设计；
- 错误应作为结构化 Observation 返回，区分可重试、需 Replan、权限拒绝和终止；
- 无依赖只读工具可并行，有文件/数据库写依赖必须由执行器而不是模型文本保证顺序；
- Trace 要关联 session/turn/tool_call/attempt/policy/result，便于回放与 bad case 归因。
- Codex 的 Registry/Router 与读写并发门、Claude Code 的 validation→hooks→permission→call→post-hook 流水线，以及 DSH 的 ToolRuntime 都可作为具体例子。见Loop 与工具执行。



### 5. Function Calling、Tool、MCP 与 Skill 有什么区别？

Function Calling 是模型输出结构化调用意图的一种接口形式；它不负责真正执行、权限或传输。

Tool 是可执行能力及其输入输出契约，可以是本地函数、Shell、HTTP 服务或 MCP 端点。

MCP 是工具/资源/提示的发现与通信协议，解决客户端与外部 Server 的互操作，不等于业务流程。

Skill 通常是面向任务的可复用 playbook/能力包，包含说明、脚本、资源、示例、验收和失败处理；它不是统一网络协议。

工具很多时可采用渐进披露：Context 里常驻名称、短描述和触发条件，命中后再加载完整 Skill 和资源。但 Harness 仍要执行依赖校验、权限、版本、输出裁剪和观测，Skill 不能把未经审计的动作藏在 Prompt 里绕过安全。



### 6. Permission 和 Sandbox 为什么必须分开？

一句话：Permission 决定“是否允许”，Sandbox 决定“允许后最多能碰到什么”。


面试里可画两层状态机：先根据身份、工具、参数、资源和风险等级作 deny/ask/allow；允许后再用文件系统、网络、进程、凭证、容器/虚机边界限制实际能力。高风险操作还要有二次确认、审计和可逆/补偿设计。


四仓差异很适合举例：Codex 有显式 Approval/Sandbox 编排；DSH 用 Permission Preset、Approval Provider 与 Sandbox 组合；Claude Code 2.1.88 有 Permission Context 和可选 Sandbox Adapter；Pi 默认把边界交给宿主。详见权限、沙箱、信任与扩展。



### 7. 什么时候该用 Multi-Agent？

不要用“复杂任务就多 Agent”作答。只有以下收益大于协调成本时才拆：


- 子任务可并行且依赖清晰；
- 不同子任务需要隔离 Context、权限、工具或工作区；
- 专业角色能带来可测的质量增益；
- 单 Agent 的上下文污染或搜索空间已成为瓶颈。
- 设计时要回答：root/controller 是谁，共享状态 Schema，任务 Ownership，预算与取消传播，结果/错误协议，重试与幂等，工作区/Worktree，Merge/Review Gate，孤儿任务清理和 Trace。


还要指出“支持 Subagent”不是一个统一能力：Pi 的示例扩展、DSH 的 provider seam、Claude 的 AgentTool sidechain、Codex 的 root-tree scheduler 不能互换。见Subagent 架构对比。



### 8. Agent 死循环、路径震荡、偏航怎样处理？

按“检测—限制—恢复”回答：


- 检测：最近 N 步 Action+参数重复、状态哈希不变、计划在两个节点间往返、错误类型连续相同、里程碑长期不推进；
- 限制：Step/Time/Token/Cost Budget、工具级熔断、递归深度、并发与子 Agent 数；
- 恢复：保留目标和失败证据后 Replan，切换工具/模型，回到最近 Checkpoint；
- 副作用：工具幂等键、事务/补偿、已提交动作清单，避免恢复时重复付款、发信或写库；
- 人工：高风险、重复失败或证据不足时转 HITL，而不是无限 Reflection。


### 9. Agent Eval 应该怎样分层？

至少四层：



| 层 | 例子 |
| --- | --- |
| Outcome | 任务成功率、正确性、约束满足、人工验收 |
| Process | 规划/工具选择、参数正确率、无效轮次、恢复成功率、轨迹长度 |
| Safety | 越权、Prompt Injection、秘密泄露、不可逆副作用、误拒绝 |
| Efficiency | P50/P95 延迟、Token、工具次数、并发资源、单任务成本 |

评测时固定模型、Prompt、工具 Schema/版本、权限和环境快照；保存可回放的 Session/Event Log；按失败阶段分桶；LLM-as-Judge 必须用人工或可验证终态校准。离线回归只能支持发布门禁，不能替代线上 A/B、用户反馈和安全监控。



### 10. RAG、长上下文与 grep/代码索引怎样选？

- 大规模、持续更新、需要权限过滤和证据引用的异构知识，优先 RAG；
- 小而完整、结构强耦合的材料，可以直接放长上下文；
- 代码里的精确标识符、路径、错误字符串和当前工作区状态，grep/符号/AST 索引通常比纯向量更可靠；
- 模糊语义、跨文件概念和历史设计可用向量/图检索补充；
- 生产 Coding Agent 常把精确搜索、结构索引、语义检索和按需读文件组合，而不是三选一。
- 无论哪种方案，都要用任务数据验证：检索层看 Recall@K/MRR/NDCG，生成层看 Faithfulness/引用，端到端看任务成功率、延迟和成本。Rerank 变差时先检查候选召回、训练域、Query 长度、Chunk 粒度和截断，不要只调 TopK。



### 11. SFT、DPO、PPO/GRPO 在 Agent 中分别解决什么？

先确认问题是否真的需要训练。私有 Schema、固定格式和初始轨迹模仿可用 SFT；成对偏好可用 DPO 类目标；多步决策、可执行终态或过程 Reward 明确时再考虑 PPO/GRPO/RLVR。没有可靠 verifier/reward 和对照实验，RL 只会把错误放大。


Agent 轨迹数据要区分 System/User、Assistant 决策、Tool Call、环境 Tool Response 和最终答案。环境返回通常不是模型要预测的目标，需要明确 loss mask；训练还要处理长短样本偏差、无效调用、工具版本漂移、训练—服务 Schema 不一致和 Reward Hacking。



### 12. AI 生成的代码怎样验证？

可以给出五层门禁：


- 需求与约束：先写 acceptance criteria、禁止改动范围和接口契约；
- 静态检查：类型、Lint、依赖、秘密与安全扫描；
- 动态验证：单测、集成、回归、边界/故障注入；
- 差异审阅：逐文件解释为什么改、是否有无关重构、迁移/回滚方案；
- 责任边界：AI 可生成，候选人必须理解、Review、验证并对提交负责。
- 现场 AI Coding 时应先让 Agent 理解仓库和测试，再做最小可运行版本；每轮只修改可归因的一小组问题，保留提交/检查点，避免把越来越长的失败上下文继续喂回去。



## 六、项目深挖：一套不容易被问穿的讲法


### 2 分钟版本

用七句话讲清：


- 用户与任务： 谁在什么场景下要完成什么；
- 原始基线： 人工、规则、搜索或固定 Workflow 的痛点；
- 为什么 Agent： 哪个决策必须根据 Observation 动态变化；
- 核心链路： Context → Loop/Planner → Tool → State → Validation；
- 个人边界： 自己直接拥有的模块、代码和决策；
- 证据： 数据集、baseline、指标、样本数和观测窗口；
- 失败与取舍： 一个真实 bad case、根因、修复和仍未解决的限制。


### 10 分钟架构版本

建议按下面的执行顺序画图，而不是按技术栈罗列名词：


入口/身份 → Policy → Context Builder → Loop/Orchestrator → Tool Router → Validation/Approval/Sandbox → Executor → Observation → State/Event Log → Eval/Trace → 用户确认/交付


每一层都准备四个问题：输入输出是什么、状态在哪里、失败怎么处理、指标怎么验证。



### 指标怎样说才可信

不要临场编“从 60% 提升到 85%”。一个可信表述至少包含：


- 基线版本和比较变量；
- 数据来源、样本量、时间范围和难度分布；
- 主指标和护栏指标；
- 是否重复运行、是否有置信区间或人工复核；
- 改进究竟来自 Agent 机制，还是数据清洗、规则、模型升级或人工补丁；
- 当前证据的限制。
- 没有线上 A/B 就明确说离线回放；没有用户就说尚未验证真实使用价值。诚实的证据边界通常比漂亮但无法复现的数字更有说服力。



## 七、P0 必答题单

下面 36 题建议都能在白板上讲 3–5 分钟，并承受两轮追问。



### 架构与控制面

- Agent 与 Chatbot、一次 Function Call 的本质区别是什么？
- Agent、Workflow、Orchestrator、Harness 怎么分？
- ReAct 与 Plan-and-Execute 各自的失败模式是什么？
- Planning 应由模型还是框架负责？
- 为什么自研，不直接用 LangGraph、Claude Code 或公司平台？

画出一次用户请求从输入到交付的完整数据流。


### Context、Session 与 Memory

- Context、Session History、Working State、Long-term Memory 怎么分？
- Context 超窗时如何触发、压缩、验证和恢复？
- 哪些信息绝不能交给摘要模型决定是否保留？
- Memory 何时写入，如何做 Scope、Authority、TTL、去重与冲突？
- Transcript 完整保存，为什么仍不能保证精确重放？
- Prompt Cache 怎样受 Context 顺序和动态内容影响？


### Tool、Skill 与协议

- 模型如何从几百个工具中选到正确工具？
- Tool Schema、参数幻觉和错误码怎样设计？
- Function Calling、Tool、MCP、Skill 与 A2A 的边界是什么？
- Skill 渐进披露如何节省 Context，又有什么安全风险？
- 工具超时、空返回、部分成功、重复副作用分别怎样处理？
- 哪些 Tool Call 可以并行，谁来保证依赖顺序？


### 安全与可靠性

- Permission 与 Sandbox 为什么不能互相替代？
- 如何防 Prompt Injection 诱导越权或秘密泄露？
- 付款、发信、删库、执行 Shell 分别怎样分级与确认？
- Agent 死循环、路径震荡和任务偏航如何检测？
- 进程崩溃后如何恢复而不重复副作用？
- 高并发下如何做幂等、限流、熔断、降级和补偿？


### Multi-Agent 与 Coding Agent

- 什么场景单 Agent 更好，什么场景值得多 Agent？
- 多 Agent 如何共享状态、传播取消和控制 Token 预算？
- 两个 Agent 同时改文件/数据库怎样避免冲突？
- Claude Code、Codex 与你的 Coding Agent 控制面有何差异？
- 代码上下文为什么不能只靠向量库？
- AI 生成代码怎样 Review、测试、回滚和追责？


### RAG、Eval 与训练

- RAG、长上下文、grep/符号索引如何选？
- Chunk、Hybrid Retrieval、Rerank 和 Query Rewrite 怎样调？
- Agent Eval 为什么不能只看最终准确率或 LLM Judge？
- 怎样构建可回放的 bad case 集与发布回归门禁？
- SFT、DPO、PPO/GRPO 各自适合 Agent 的什么问题？
- Tool Response 是否参与 loss，reward hacking 如何发现？


## 八、按岗位追加的 P1 题


### 应用/全栈

- SSE 与 WebSocket 如何选，断线续传和取消怎样做？
- RAG 索引怎样不停服更新，权限过滤放在哪一层？
- Redis Semantic Cache 的 key、失效与污染怎样治理？
- 消息队列如何保证长任务至少一次消费下的业务幂等？
- Agent 输出怎样转成可验证的结构化业务对象？


### Harness/Infra

- Session/Task/Turn/Step 如何建模？
- Tool Registry 与动态插件如何版本化和热更新？
- 模型网关如何路由、限流、缓存与统计 Token？
- 容器、gVisor、Firecracker 和宿主进程隔离怎样取舍？
- Trace 如何跨模型调用、工具、子 Agent 与审批串起来？
- 长任务调度怎样处理租户公平性、取消和孤儿任务？


### 算法/后训练

- Agent trajectory 数据如何生成、筛选和去重？
- SFT 何时足够，切 RL 的判据是什么？
- DPO 正负样本长度差异会带来什么偏差？
- GRPO 的 reward、group 与 credit assignment 如何设计？
- Tool 使用的过程 reward 与终态 reward 如何组合？
- Train-serving 的 Prompt/Schema/工具版本偏移如何诊断？


### GUI/Browser/端侧

- DOM、截图、OCR、Accessibility Tree 如何组合？
- 本地文件和目录授权如何可见、可撤销、可审计？
- GUI 动作怎样检测页面已变化、动作是否真正生效？
- 端侧模型在延迟、内存、功耗和隐私之间如何选型？
- 长任务在本地关机、断网或 App 重启后怎样续跑？


## 九、14 天突击计划


### 第 1–3 天：把项目讲成状态机

- 画完整数据流和失败流；
- 写出个人代码边界、baseline、指标口径和三个 bad case；
- 为每个框架选型准备一个替代方案和拒绝理由；
- 把 2 分钟与 10 分钟版本各录两遍。


### 第 4–6 天：RAG、Context、Memory

- 用同一批文档比较 BM25、向量、混合+Rerank；
- 实现 Query Rewrite 和权限元数据过滤；
- 做一次压缩前后回放，记录约束丢失、Token、延迟和任务成功；
- 能明确说出 Context、Session、Transcript 和 Memory。


### 第 7–9 天：Loop、Tool、安全与恢复

- 自己写一个最小 Loop，不依赖黑盒 Agent API；
- 加 Schema 验证、结构化错误、Step/Time/Token Budget；
- 为写操作加幂等键、Approval 和最小 Sandbox；
- 注入一次工具超时、进程崩溃和 Prompt Injection，验证恢复/拒绝。


### 第 10–11 天：Eval 与可观测

- 建 30–50 条小型回放集，按 outcome/process/safety/efficiency 评分；
- 保存模型、Prompt、Tool、权限和环境版本；
- 把 bad case 分成检索、Context、规划、工具、权限、状态和验证七类；
- 不用单一 LLM Judge 代替全部评测。


### 第 12–14 天：公司与岗位定向

- 按 JD 判断自己在应用、算法、Harness、Coding/GUI 哪条线；
- 从上面的公司画像选 10 道定向题；
- 每天手撕 2–3 道算法，并复习岗位对应的数据库/OS/网络或 RL；
- 做两次 60 分钟模拟：20 分钟项目、20 分钟场景、15 分钟基础/代码、5 分钟反问。
- 如果有四周时间，把第 7–11 天的最小 Harness 做成可运行项目，并保留 Eval 报告、Trace 截图、故障注入和设计文档。这个证据比再堆一个 LangChain Demo 更有区分度。



## 十、反问面试官

反问的目标是确认岗位的真实控制面与失败成本：


- 这个团队更接近 Agent 应用、模型后训练，还是 Runtime/Harness？三者工作比例如何？
- 当前最大的失败来源是模型、Context、工具、状态、评测还是业务接入？
- Agent 的输出是建议，还是会直接触发不可逆动作？HITL 和责任边界是什么？
- 团队用哪些 outcome/process/safety/cost 指标做发布门禁？
- 是否保存可回放轨迹，线上 bad case 如何进入回归集？
- 自研能力与 Claude Code/Codex/开源框架的边界在哪里？
- 实习生/新人的第一个可独立交付模块通常是什么，如何判断完成？


## 十一、最后的准备优先级

如果时间只够抓五件事，顺序建议是：


- 把自己的项目、指标和坏例说到可复现；
- 保住算法与岗位对应的工程/模型基础；
- 真正理解 Context、Tool、State、Permission/Sandbox 和 Eval；
- 读一个 Coding Agent 的完整调用链，而不是只看产品介绍；
- 针对目标团队补 RAG、RL 或 Infra，不按“Agent 八股大全”平均用力。
- 2026 年面试官真正区分候选人的问题，往往不是“你知道多少新名词”，而是：模型做了一个概率决策之后，你怎样让真实系统仍然可控、可恢复、可验证、可追责。



## 主要来源索引

下面只列本文反复使用的核心来源；各公司章节中的链接提供更细的一手样本。



### 一手面经

字节：2026-08 Agent 两轮、2026-06 Harness/Context/Eval、2025 四轮大模型

阿里/淘天：2026 淘宝闪购 Agent 算法、2026 阿里云 AI Coding、2026 瓴羊 Harness

蚂蚁：2026 智能体与 LLM 应用、2026 蚂蚁财富 Agent、2025 蚂蚁国际

腾讯：2026 Coding Agent/Context/Subagent、2026 腾讯多样本索引

百度：2026 文心 Agent RL/Harness、2026 Prompt→Context→Harness

华为：2026 某 BG Agent 完整流程、2025 AI 工程师

美团：2026-08 Agent 开发一面、2025 后端/Agent

京东：2026 实际 Agent 开发一面、2025 一面

快手：2026 Data Agent/RAG/Tool Security、2025 Data Agent、2025 Agent RL

拼多多：2026 提前批 Agent 一面短帖

小米：2026 AI Agent 一面、2025 大模型应用

小红书：2026 Agentic 全栈、2025 Skill/Context

阶跃星辰：2026 企业面经页；智谱：2026 Agent 算法；MiniMax：2026 AI 应用开发、2026 平台研发


### 官方岗位与技术方向

百度 2027 AIDU 智能体算法工程师

京东大模型应用算法/开发专家（Agent）

京东大模型智能体岗位（Skills/Subagents/Harness）

字节 AI Agent Memory Infrastructure

字节 Seed Early Career

阿里 2027 校园招聘

美团 CatPaw Agent 平台技术文章

小米全球顶尖人才前沿课题

华为挑战课题专项招聘


### 本项目源码分析

- [Pi Agent 源码详解](https://keepkeen.github.io/blog/pi-agent-source-analysis/)
- [OpenAI Codex 源码详解](https://keepkeen.github.io/blog/openai-codex-source-analysis/)
- [DeepSeek Harness 源码详解](https://keepkeen.github.io/blog/deepseek-harness-source-analysis/)
- [Claude Code 源码详解](https://keepkeen.github.io/blog/claude-code-source-analysis/)
- [Agent Loop 与工具执行](https://keepkeen.github.io/blog/coding-agent-loop-tools/)
- [权限、沙箱、信任与扩展](https://keepkeen.github.io/blog/coding-agent-security-extensions/)
- [上下文、会话、压缩与子代理](https://keepkeen.github.io/blog/coding-agent-context-session-subagents/)
- [接口与可观测性](https://keepkeen.github.io/blog/coding-agent-interfaces-observability/)
本专题目录



---


## 补充：从「烤面」题库延伸的 Agent 练习

这部分由本站根据 [smile-struggler/kaomian](https://github.com/smile-struggler/kaomian) 公开题库的主题独立整理，题目是练习方向，不是对面试原话的逐条转录。该仓库是面试题库，截图中介绍 Pi、Codex 等系统源码的八篇文章则来自 Liuliming 的专题，二者并非同一个项目。

仓库 README 所示快照覆盖 2025 年 3 月 22 日至 2026 年 4 月 18 日：共抓取 767 篇帖子，筛选保留 248 篇，抽取 948 个问题，并合并为 638 个规范化问题。它通过模型进行分类、OCR、问题提取和语义归并；这些数字描述的是该数据集的处理结果，不能直接代表整个行业的面试频率，也不能单独证明某家公司在特定轮次问过某题。

| 练习方向 | 可以怎样展开回答 | 继续追问 |
| --- | --- | --- |
| 效果评估 | 先定义任务成功条件，再分别观察最终结果、过程、安全与耗时成本。 | 样本如何构造？人工标注与自动评测如何互相校验？ |
| Agent 工作流 | 从输入、规划、工具、状态和退出条件画出一次任务的执行链。 | 任务失败时如何定位是哪一步出错？ |
| 工具参数可靠性 | 用清晰的工具描述和参数约束帮助模型选工具；执行前做服务端校验。 | 参数缺失、越权或重复调用分别怎么处理？ |
| 上下文长度 | 先控制历史和工具结果的进入范围，再设计摘要、检索和关键状态保留。 | 摘要丢失约束时怎么发现？如何回退？ |
| 长短期记忆 | 将本次任务状态与跨会话保存的信息区分，明确写入、召回和过期规则。 | 如何防止错误记忆反复影响结果？ |
| 工具超时 | 区分未发出请求、下游未完成和下游已完成但响应丢失，再决定查询、重试或人工介入。 | 涉及写操作时如何避免重复提交？ |
| 多 Agent 协作 | 根据任务拆分收益决定是否并行，约定每个角色的输入、输出和共享状态。 | 同时修改同一份文件或互相等待时怎么办？ |
| 项目追问 | 按背景、个人职责、设计取舍、验证方式和失败案例来讲。 | 哪些结果是你直接完成的，哪些是团队共同完成的？ |

练习时可以先选一个熟悉的业务场景，把表中的问题串成一次完整执行：用户提出目标 → 系统读取必要数据 → 模型选择工具 → 程序校验和执行 → 验证结果 → 记录失败并改进。项目细节只写实际做过、能够解释和验证的部分。

题库原始入口：[总索引](https://github.com/smile-struggler/kaomian/blob/main/%E9%A2%98%E5%BA%93/00_%E9%A2%98%E5%BA%93%E6%80%BB%E7%B4%A2%E5%BC%95.md) · [Top 100 高频题](https://github.com/smile-struggler/kaomian/blob/main/%E9%A2%98%E5%BA%93/01_Top100_%E9%AB%98%E9%A2%91%E9%A2%98.md) · [Agent / RAG / Tool / Memory](https://github.com/smile-struggler/kaomian/blob/main/%E9%A2%98%E5%BA%93/03_Agent_RAG_Tool_Memory.md) · [项目追问题](https://github.com/smile-struggler/kaomian/blob/main/%E9%A2%98%E5%BA%93/06_%E9%A1%B9%E7%9B%AE%E6%8B%B7%E6%89%93%E9%A2%98.md)。


## 补充阅读：我们整理的 Agent 学习笔记

以下是本站此前的三篇学习材料，合并在同一篇文章中，便于继续查阅。与上方原文分别署名；各部分可能有重复或不同的证据边界。


### 公开样本与通用能力

> **检索截止：2026 年 11 月 15 日。** 这是一份面向 Agent 相关学习者的通用能力记录，涵盖应用、知识检索、运行时、模型与 Coding Agent。本文参考了 [Liuliming 的调研文章](https://keepkeen.github.io/blog/china-agent-interview-guide-2025-2026/)的组织思路；样本由本文重新检索、筛选和分析。公开面经是候选人回忆，不等于官方题库。

### 先说结论

1. **Agent 不是一个单一岗位。** 有的工作负责具体业务，有的建设工具和运行环境，有的聚焦模型训练、检索或自动评测。先辨认交付物，再决定学习范围。
2. **基础能力与 Agent 能力交叉出现。** 在本次可读样本中，有人被问模型、工具和上下文，也有人同时面对数据库、网络、编程或系统设计问题；但小样本不能说明各题的整体频率。
3. **能够描述一次真实执行很重要。** 不仅解释正常调用，还要交代数据来源、状态、权限、失败处理和如何判断结果正确。
4. **技术答案需要匹配具体系统。** 工具调用、MCP、记忆、检索、权限与评测没有放之四海皆准的实现。面试中要区分一般原理、产品文档、特定项目事实及待验证方案。
5. **证据比术语堆砌更可靠。** 知道一份资料来自谁、何时、什么岗位，能证明什么、不能证明什么，比把所有条目都叫“高频真题”更有用。

### 一、证据如何选择

#### 1. 本次范围与分级

这里使用四类材料，等级表达用途，不表示可以机械地用单条来源证明全部结论。

| 类型 | 内容 | 能用于什么 | 局限 |
| --- | --- | --- | --- |
| A：官方来源 | 岗位官网、产品文档、协议和原始研究 | 确认公开职责或对应技术机制 | 招聘信息会变化；技术文档不证明某道面试题真实出现 |
| B：候选人自述 | 明确以个人经历叙述的面试记录 | 分析一次面试的岗位、上下文和追问 | 是未经独立验证的个人回忆，可能不完整或有误 |
| C：二次资料 | 合集、结构化题库、第三方岗位快照 | 找线索、补充主题和来源 | 可能混入转述、去重错误或过期岗位；不能重复计为独立样本 |
| D：可疑或不可核查材料 | 时间矛盾、来源不清、明显营销或无法追溯的内容 | 最多作为继续检索的线索 | 不用于支撑公司画像和年代比较 |

本次直接阅读了下表列出的十条个人自述；其中仅四条页面明确标出年份或具体面试日，其余只记录页面能确认的信息。未把搜索到的所有结果都算作已核实样本，也没有进行代表性抽样。

#### 2. 一手候选人记录台账

| 编号与来源 | 岗位／时间线索 | 本次可观察主题 | 可用边界 |
| --- | --- | --- | --- |
| [B1 快手多轮记录](https://www.nowcoder.com/feed/main/detail/4344d0da296944f8b6bcdb0acdbe141e) | 2025-09-15 发帖，具体职位未写明 | RAG、MCP、Agent、模型微调和代码题 | 一位候选人的简短回忆；不能确定每个问题对应哪一轮 |
| [B2 字节 AI 应用一面](https://api-cdn.nowcoder.com/feed/main/detail/b34f6902e8544fe2953696ed52e49dba?sourceSSR=dynamic) | 文章自述面试日为 2026-08-27 | 执行循环、长任务取消、上下文、记忆和 Java 基础 | 一次候选人自述，不代表企业统一题单 |
| [B3 阿里云平台方向一面](https://www.nowcoder.com/feed/main/detail/e2d7d87cab1c41c6a70d9eef166ec1c1?sourceSSR=subject) | 秋招后端／Agent；具体面试日期未披露 | 网关、流式背压、资源治理与系统设计 | 日期不精确；岗位偏底层平台，不能当作应用岗普遍要求 |
| [B4 美团 Agent 开发一面](https://www.nowcoder.com/feed/main/detail/58159306df52463ab75d72daa80d66df?sourceSSR=enterprise) | 日常实习；页面显示 8 月 25 日发布，年份未明确 | RAG 与文件搜索、评测、Agent 与 Harness | 年份不推断；实习样本不能直接套用到社招 |
| [B5 百度 Agent 开发一面](https://www.nowcoder.com/feed/main/detail/fb9d83f0b6cd44b1ac3e3cafed6a1cf0) | 一面；页面显示 3 月 24 日，年份未明确 | 上下文、代码理解、生成测试质量、模型基础 | 年份和招聘类型未核实 |
| [B6 腾讯后台 Agent 实习一面](https://www.nowcoder.com/feed/main/detail/5a6705afdb45487e852c29bad27575d5) | 实习；页面显示 4 月 29 日，年份未明确 | 编程语言、限流、消息、数据库与模型接入 | 更偏后端的一次经历，不代表腾讯 Agent 岗整体 |
| [B7 贝壳 RAG 实习记录](https://www.nowcoder.com/feed/main/detail/b8c08e8aa1974639965a795a2d7f82d8) | 2025-11-20 发帖，面试日期未写明 | 项目细节、模型基础和算法题 | 是简短个人回忆；不能据此确定具体面试日或招聘趋势 |
| [B8 京东技术运维／Agent 实习](https://linux.do/t/topic/1591661) | 2026-02-10 发帖；具体面试日未标明 | 运维场景、RAG 文件处理与业务效果 | 一次实习面试自述，岗位名称与实际工作需结合原文理解 |
| [B9 蚂蚁智能体与大模型应用](https://ac.nowcoder.com/discuss/1646420?type=0) | 帖子记录 5 月 20 日面试；年份未明确 | Skill、代码搜索、RAG 与后端基础 | 年份未独立确认；只可归纳该候选人回忆 |
| [B10 阿里云 Agent 算法](https://www.nowcoder.com/feed/main/detail/3a61d09fc2fb436f9bd5c1ed99b43330?sourceSSR=users) | 页面显示 8 月 27 日；年份未明确 | Agent Loop、安全、权限和长任务 | 页面可读但年份不明确；不代表所有团队要求 |

由于年份可确认的材料少，**不能据此量化 2025 到 2026 的趋势，也不能排名哪个企业“最常考”某题**。个别网站搜索摘要可见但原文无法读取时，本文不把它提升为可核实的一手记录。

#### 3. 岗位与技术旁证

- [字节 Seed 官方人才页面](https://seed.bytedance.com/zh/seedearlycareer)列有模型、机器学习系统、搜索、记忆和多模态等方向，反映其公开的研究与人才方向；它不是面试题来源。
- [AWS AgentCore 官方岗位](https://www.amazon.jobs/en/jobs/10591991/software-development-engineer-agentcore-aws-agentic-ai)描述运行时、会话隔离、工具编排、观测和失败处理等职责；这是海外岗位，不直接代表国内招聘市场。
- 国内岗位另参考[知识库岗位第三方快照](https://jobs.ultraai.site/jobs/bytedance/7644505471581849909)和[跨平台 Agent 岗位第三方快照](https://jobs.ultraai.site/jobs/bytedance/7683834512367388933)。岗位可能变化或下线，申请前需重新核实官方页面。
- [开源面经题库](https://github.com/smile-struggler/kaomian)自报的快照覆盖 2025-03 至 2026-04，披露了筛选、OCR 和模型归并过程。本文将其作为找主题的线索，不把它的结果合并为本次一手样本计数，也不从中推导精确公司考频。

### 二、先按交付物划分岗位

#### 应用与业务 Agent

交付的是用户能使用的功能：例如问答、分析、流程辅助或业务系统操作。重点包括需求边界、工具和数据接入、交互体验、权限、效果与运营。一个可用的应用不一定要求候选人亲自训练模型。

#### 知识检索与数据应用

交付的是可访问、可更新、可追溯的信息能力。重点包括文档处理、索引构建、检索排序、权限过滤、版本更新与评测。检索只是系统一环，数据是否可信和授权范围同样重要。

#### Runtime、Harness 与基础设施

交付的是支撑任务执行的公共能力，例如运行状态、任务调度、工具执行、隔离、流式传输和恢复。不同框架对术语的划分不同，不应把某一产品的模块边界当作唯一标准。AWS 岗位提供了一个可对照的公开实例，而非所有实现的定义。

#### Coding、浏览器和多模态 Agent

系统需要理解和操作具体环境：读取代码或页面、调用工具、验证产物、处理界面变化与用户确认。评估不能只看生成文本，还要看文件改动、测试、执行结果及安全边界。

#### 模型与训练方向

关注模型能力、数据、训练、推理和实验设计。候选人可能需要解释微调、强化学习、奖励和验证等，但具体深度取决于岗位；理解概念与有训练项目经验应分开表述。

岗位分类是分析工具，并非互斥标签：一个团队也可能同时覆盖应用和基础设施。

### 三、公司样本怎么看

下表只概括本次**可直接阅读的单条样本**，方便定位来源；它不是企业排名或题目概率。

| 企业或团队线索 | 单条记录中出现的内容 | 准备时可借鉴 | 不应推断 |
| --- | --- | --- | --- |
| 字节，AI 应用一面 B2 | 执行循环、任务取消、上下文与 Java 基础 | 练习说明执行与取消边界，并保留语言基础 | 所有字节岗位都这样面 |
| 阿里云平台方向 B3 | 大模型网关、流式背压、资源与系统设计 | 分清应用层和平台层，准备扩展性与稳定性 | 阿里所有 Agent 团队都做网关 |
| 美团，日常实习 B4 | 项目、RAG 与文件检索、结果验证 | 能解释检索选型、评测和真实项目取舍 | 这套题可代表美团社招 |
| 百度，Agent 一面 B5 | 上下文、代码理解、生成测试与模型基础 | 练习测试有效性及上下文选择 | 依据页面推断确切面试年份 |
| 腾讯，后台实习 B6 | 编程语言、数据库、限流和模型接入 | 重视后端基础与业务场景 | 所有腾讯 Agent 岗技术比重相同 |
| 快手，多轮 B1 | RAG、MCP、Agent、微调和代码题 | 复习模型应用和基本工程问题 | 代表完整面试流程或 2025 年市场全貌 |

### 四、面试通常怎样向下追问

这里归纳的是**阅读和准备框架**，不是所有面试都会采用的固定顺序。

1. **目标与背景**：用户要完成什么，为什么需要这个系统？
2. **执行链路**：输入如何变成上下文，何时调用模型和工具，结果如何返回？
3. **个人边界**：哪些模块是自己完成的，哪些来自平台或其他团队？
4. **失败场景**：输入缺失、接口超时、模型选错工具、进程中断如何处理？
5. **效果与验证**：测试样本、基线、指标口径、线上观测与真实失败案例是什么？
6. **方案取舍**：为什么选择这种实现，其他方案的成本与限制是什么？

回答时如果某项能力来自现有框架，可以解释自己负责的接入和配置；没有做过的底层实现，明确说不了解，再讲合理的验证思路。

### 五、Agent 核心概念与边界

#### 1. Agent、Workflow、运行时怎么区分？

Workflow 通常强调开发者预设的步骤与条件；Agent 通常强调模型根据上下文和可用能力参与下一步决策；运行时负责管理实际执行。它们不是互斥的：一个工作流里可以有模型决策节点，Agent 内部也可能调用确定性流程。

**追问：** 哪些步骤必须固定？决策出错时能否停止？模型能否直接触发外部副作用？

#### 2. Tool、Function Calling、MCP、Skill 怎么区分？

工具是可执行能力；Function Calling 是模型提出结构化调用意图的一种机制；MCP 是连接应用与外部能力的一种协议。Skill 在不同平台中可能指组织任务所需说明和资源的方式，需结合具体产品理解。

模型给出参数后，实际调用通常仍由应用负责，格式正确也不等于权限正确。[Function Calling](https://developers.openai.com/api/docs/guides/function-calling)；[MCP 架构](https://modelcontextprotocol.io/docs/learn/architecture)。

#### 3. 上下文、会话、记忆和状态有何不同？

上下文是某次模型调用实际看到的信息；会话用于组织交互；记忆通常指更长期可复用的信息；状态记录任务或业务进展。名称在不同系统中可能重叠，应先说明保存内容、生命周期和读取者。不能把模型上下文当作业务数据库或权限系统。

#### 4. RAG、长上下文与精确搜索如何选择？

资料规模、更新频率、权限、问题类型和可核查性都会影响方案。小而完整的材料可能适合直接提供；大量资料可能需要索引检索；精确代码标识符或文件路径可用文本或符号搜索。应在具体任务上对比正确性、延迟和成本，避免假定单一方案永远最优。[Retrieval 文档](https://developers.openai.com/api/docs/guides/retrieval)

#### 5. 多 Agent 什么时候值得使用？

当子任务可独立分工、需要不同工具或上下文、并能产生可衡量收益时，可以考虑。额外开销包括调度、通信、状态冲突、错误归因、权限和预算。先定义任务边界与验收方式，再和单 Agent 或固定流程比较。

### 六、故障、权限与安全：从正常路径走向可控系统

下面是通用设计问题，实际项目要以系统实现和业务约束为准。

| 场景 | 首先区分什么 | 可考虑的措施 |
| --- | --- | --- |
| 参数缺失 | 能否从可信上下文补齐；错误猜测的代价 | 参数校验、向用户澄清、限制模型自造身份 |
| 空结果与超时 | 查询成功为空，还是结果未知 | 统一状态语义，保留错误类型与来源 |
| 写操作超时 | 请求是否可能已生效 | 幂等键、查询执行状态、业务补偿或人工确认 |
| 重复调用 | 是合法分页还是无进展循环 | 调用预算、重复检测、错误分类和安全停止 |
| 任务中断 | 哪些状态已提交，哪些副作用已发生 | 持久化检查点、恢复前核对执行状态 |
| 不可信内容 | 外部文档是否试图影响工具或权限 | 限制权限、隔离数据与指令、对高风险操作设置确认 |

提示词注入不能只靠一句“忽略恶意指令”解决。受控工具、最小权限、审计、校验与人工确认可以组合使用；但任何单项措施都有边界。[Agent 安全指南](https://developers.openai.com/api/docs/guides/agent-builder-safety)

### 七、评测：不要只看最终回答

先界定任务和成功标准，再确定样本与比较方式。一种实用的拆分如下：

| 维度 | 示例问题 |
| --- | --- |
| 结果 | 最终事实、格式和用户约束是否满足？ |
| 过程 | 是否调用正确工具，参数和顺序是否合理？ |
| 安全 | 是否越权、泄露数据或产生未经授权的副作用？ |
| 效率 | 用时、模型调用量、工具调用数和资源消耗如何？ |
| 恢复 | 工具失败或会话中断后能否得到正确状态？ |

先有可复现的测试集：正常、缺参、无权限、空数据、工具超时、错误资料等都要覆盖。比较前后版本时固定模型、工具、数据和评估口径，并对自动评分抽样人工校准。轨迹评测可以帮助定位过程问题，但不能单独证明真实用户体验。[Agent Evals](https://developers.openai.com/api/docs/guides/agent-evals)；[Trace Grading](https://developers.openai.com/api/docs/guides/trace-grading)。

### 八、十个通用练习题

这些问题是根据能力主题编写的**练习题**，不是逐字复制的企业真题。

1. 设计一个需要查询外部数据的 Agent，画出模型、工具、业务服务和用户之间的边界。
2. 什么时候选择固定工作流，什么时候让模型选择下一步？分别有哪些失败模式？
3. 工具返回大量数据时，如何降低上下文负担，同时保留可追溯信息？
4. 用户说“按上次的条件继续”，如何确定本轮参数和授权范围？
5. 查询无结果、接口超时和没有权限，最终回答应该怎样区分？
6. 一个有副作用的任务执行到一半中断，如何避免恢复时重复执行？
7. 外部资料中混入诱导工具调用的文本，系统有哪些不同层次的防护？
8. 检索结果看起来相关但回答仍错误，如何分阶段定位？
9. 如何用结果、轨迹、安全和成本共同评估一次系统修改？
10. 多个执行者同时操作同一资源时，如何划分任务、发现冲突和决定谁来合并？

### 深入专题：六类问题怎样答到工程层

以下答案是综合公开技术资料形成的**设计与学习示例**，不是对某个候选人项目的事实描述。每节都包含一个场景、分析方法、取舍和面试时可以如何简述。

#### 专题一：一次工具调用，为什么看起来成功却答错了？

**场景：** 用户问“上海区域上周的销量”，模型调用查询工具，接口返回 HTTP 200，最终回答却使用了全国数据。只看接口状态会误以为整个过程成功。

**怎样拆开定位：**

1. 看用户请求：是否明确了区域、时间和指标？“上周”按哪个时区、自然周还是最近七天？
2. 看模型参数：区域是否正确映射，是否遗漏筛选字段？
3. 看权限与服务端执行：用户身份由哪里传入，工具服务是否真的按授权范围和查询条件过滤？
4. 看返回结果：服务返回的是原始明细、聚合结果还是缓存？是否带有查询范围和数据时间？
5. 看最终回答：模型是否忠实使用结果，是否把空字段或近似数据解释成确定事实？

这五步中任何一步都可能出错。因而状态码只能说明某层调用返回了响应，不能单独证明参数、业务数据或最终答案正确。[OpenAI Function Calling](https://developers.openai.com/api/docs/guides/function-calling) 描述了模型提出调用、应用执行并返回结果的基本过程；具体业务权限和数据语义仍由应用设计。

**设计取舍：** 工具入参可以加类型、枚举和必填规则，但 schema 无法判断用户是否有权访问某个区域。工具返回中可以附带范围、单位、数据时间和状态，代价是响应更长；高价值字段可保留，超大明细则提供分页或引用。

**面试简答示例：**“我会把问题拆成参数、权限、工具执行、返回数据和回答生成五层。先用关联标识还原请求，再检查区域与时间条件是否从输入一路传到了数据层。接口成功只代表工具调用完成，不代表结果符合用户意图。”

**继续追问：** 如果用户没有说区域，是继承上轮条件、从个人偏好读取，还是反问？要说明优先级、过期策略和身份变更时怎样处理，而不能默认任何历史条件都有效。

#### 专题二：工具超时之后，为什么不能一律重试？

**场景：**Agent 在办理业务时调用“提交申请”，等待 5 秒后超时。此时有两种完全不同的可能：请求没有到达下游，或者下游已经提交成功但响应在途中丢失。

**处理思路：**

- 对无副作用的查询，先限制重试次数和总时长，考虑退避、抖动以及下游是否允许重试。
- 对提交、支付、发信等写入，先记录业务请求标识，再通过下游查询执行状态；仅在下游提供相应保证时，使用同一个幂等键重试。
- 若结果仍不确定，不应告诉用户“肯定失败”；可以标记为处理中或未知，按业务方案补偿、重查或转人工。
- 日志记录应能关联用户请求、工具调用和业务动作，同时避免暴露敏感参数。

**关键边界：** 幂等不是在本地生成一个随机数就自动成立，还需要服务端定义键的作用范围、保存期限，以及并发请求如何处理。不同服务的保证可能不同，需要逐项验证。

**面试简答示例：**“我先判断有没有副作用。查询类可在预算内重试；写操作超时后结果可能未知，优先按业务标识查询状态，并依靠下游支持的幂等机制避免重复生效；状态无法确认时不盲目重试。”

#### 专题三：上下文快满了，怎么做才不只是“让模型总结”？

**场景：** 一次任务进行了多轮查询，历史消息和工具结果越来越长，而任务目标、权限约束和某个关键编号还要在后续使用。

可以先将信息分层：

| 信息 | 处理思路 | 主要风险 |
| --- | --- | --- |
| 用户当前目标、不可违背的约束 | 明确保留，并在关键动作前再次验证 | 压缩时丢失约束 |
| 业务标识、执行状态、已完成动作 | 尽量保存在结构化状态或可信业务系统 | 仅靠摘要可能丢失或写错 |
| 较早的对话和推理过程 | 选取必要部分或压缩摘要 | 摘要错误可能持续污染后续决策 |
| 大型工具输出 | 服务端筛选、聚合或分页，按需读取 | 过度裁剪可能丢掉关键证据 |
| 原始文档与证据 | 保留可重新访问的引用与版本 | 来源更新、权限变化或引用失效 |

**验证方式：** 构造压缩前后等价任务，检查关键条件、事实和授权范围是否保留；对关键动作再用确定性校验。上下文并非越长越好，但具体质量变化应在目标模型和任务上测试，而不是当成绝对规律。[Anthropic 上下文工程文章](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents)讨论了选择高价值上下文、工具与历史信息的设计。

**面试简答示例：**“我不会直接把整段历史交给模型概括。先把目标、约束和业务状态从普通对话里区分出来；结构化事实交给可信状态保存，大结果尽量源头筛选，普通历史才考虑摘要，并用测试样本检查压缩是否丢失关键信息。”

#### 专题四：RAG、长上下文、关键词搜索，面试官到底在问什么？

[美团日常实习面经](https://www.nowcoder.com/feed/main/detail/58159306df52463ab75d72daa80d66df?sourceSSR=enterprise)中，候选人回忆了 RAG 与让 Agent 搜索文件的比较。它说明某次面试中出现过这类取舍，不代表公司的统一标准。

以“查找某个系统规则的最新定义”为例：

1. **文件较少且能完整阅读：** 可以直接提供相关文档，但要考虑上下文容量和版本。
2. **用户给了精确规则名或错误码：** 关键词／符号搜索往往易于解释；但同义表达或模糊问题可能漏召回。
3. **资料多、表述多样：** 可以使用语义检索，也可以与关键词检索组合；仍需处理权限、更新、分块及重排。
4. **涉及精确计算或实时状态：** 应调用可信服务或确定性程序，而不是让模型凭文档猜测。

**怎样判断是否选对：** 建立包含精确查询、同义查询、过期资料和无权限资料的样本，分别看是否找到了正确来源、回答是否忠实以及端到端耗时。单纯比较“模型回答看起来好不好”不足以定位检索链路。[OpenAI Retrieval](https://developers.openai.com/api/docs/guides/retrieval)

**面试简答示例：**“我会先看问题需要精确匹配还是语义匹配，再看资料规模、时效和权限。实际可以组合多种检索方式，并用同一批任务样本比较召回和最终回答，而不是预设某种方案一定最好。”

#### 专题五：怎样从坏案例走到可复现的评测？

**场景：** 用户反馈“商品推荐不对”。这是一个结果描述，不能直接认定是模型幻觉，也可能是输入条件、数据、检索、工具或规则问题。

一个排查闭环可以这样设计：

1. **还原上下文：** 记录脱敏后的用户问题、当时系统版本、输入条件、工具调用及返回状态。
2. **明确正确答案从何而来：** 可信数据、业务规则或经授权的人工标注；没有标准答案时不要硬造标签。
3. **按环节分类：** 路由错误、参数错误、数据缺失、检索错误、生成偏差、权限问题等。允许一条案例有多个原因。
4. **转成可复现样本：** 固定必要数据版本、权限和模拟工具结果，保留预期行为与禁止行为。
5. **修改后比较：** 观察该案例是否改善，以及相邻案例是否退化；把有效案例加入回归集。

对于开放性回答，模型评分可作为辅助，但应设计评分标准并抽样人工核对。**回答正确但发生越权调用**，不能简单算作完整成功。官方文档也区分了最终结果与轨迹检查，参见 [Agent Evals](https://developers.openai.com/api/docs/guides/agent-evals) 和 [Trace Grading](https://developers.openai.com/api/docs/guides/trace-grading)。

**面试简答示例：**“我先还原一次执行轨迹，确认问题来自数据、工具还是生成；然后把样本和预期行为固定下来，再比较改动前后结果和回归情况。没有明确标准答案的部分，需要人工判断并说明口径。”

#### 专题六：长任务能恢复，不等于把对话历史存起来

**场景：**Agent 完成了三步查询，第四步写入业务系统前进程退出。重启后，如果只恢复聊天记录，系统可能不知道哪一步已经完成、外部写入是否发生，进而重复执行。

恢复设计至少要回答：

- **存什么：** 任务身份、目标、步骤状态、输入引用、工具结果引用、错误与版本等；不一定所有原始内容都应长期保存。
- **何时存：** 在有意义的执行边界记录状态；状态更新与外部副作用跨系统时，通常不能简单假定两者天然原子。
- **怎样恢复：** 重新验证权限与环境，检查外部动作是否已经执行，再决定重试、跳过或转人工。
- **怎样验证：** 在调用前、调用中和调用后分别模拟中断，检查数据最终状态及重复副作用。

这不是宣称某个平台已经具备上述全部能力。Anthropic 对长任务的公开讨论也指出，仅做上下文压缩不足以解决跨会话任务连续性；其文章中的具体做法应在对应环境和版本下理解。[长任务 Agent 的 Harness](https://www.anthropic.com/research/effective-harnesses-for-long-running-agents)

**面试简答示例：**“我会把会话内容和任务执行状态分开保存，并围绕外部副作用设计恢复边界。恢复前确认哪些动作已生效，再通过幂等和状态查询决定下一步；同时用故障注入验证，而不是只演示正常续跑。”


### 深入专题二：从完整任务看 Agent 工程

上一组专题侧重单个问题。本组围绕一个贯穿案例，将业务交付、安全、可靠性和评测连成一条线。**案例与数据均为虚构，仅用于解释设计方法。**

假设要构建一个“经营分析助手”：用户可以提问、查询获授权的数据，并在确认后生成一份报告。分析与报告草稿属于低风险步骤；向外发送报告是有副作用的动作，需要更严格的验证。下面讨论的是可选设计，不表示所有 Agent 框架都已经实现。

#### 1. 从自然语言到任务：哪些事情可以交给模型？

用户说“看一下昨天的销售情况”，系统首先需要知道“昨天”属于哪个时区、要看什么范围以及访问者有权看什么。模型可以帮助识别意图或建议下一步，但不能因为用户在文本里写了某个身份，就相信他拥有相应权限。

一种拆分方式是：入口取得服务端可信身份；应用结合用户输入和会话状态解析时间、区域和指标；必要信息不全时向用户提问；参数通过验证后才调用查询服务。模型负责理解语言和提供候选参数，业务服务负责授权和数据边界，确定性代码负责日期换算及数值计算。

这不意味着所有任务都要设计复杂的自主循环。如果查询流程固定，模型只需将自然语言转换为受限的参数，再进入普通业务流程即可。若任务需要根据中间结果决定下一步，比如先判断是否有足够数据再选择后续查询，可以考虑让模型在受限工具集合里决策。Anthropic 对工作流与 Agent 的讨论也强调应从任务需要出发，而非先决定使用最复杂的架构：[Building effective agents](https://www.anthropic.com/research/building-effective-agents)。

**常见追问：用户说“按上次的范围”，取到历史值后就能执行吗？**

不能直接假设。需要判断历史值属于哪个用户、哪个会话、是否过期，当前身份和授权是否变化。更稳妥的做法是在最终执行前重新确认数据权限；当历史值含糊或将触发高风险操作时，展示解析结果并让用户确认。会话记忆提供便利，不应承担授权职责。

#### 2. 把一次执行拆成可观察的状态

一个常见做法，是把任务、对话轮次、模型请求、工具调用和外部业务操作区分开。它们可以通过关联标识串起来，但各自有不同的生命周期：用户对话可能继续，某次工具调用可能失败，而外部业务操作可能已经成功。

| 对象 | 可能记录的内容 | 特别需要说明 |
| --- | --- | --- |
| 任务 | 目标、状态、创建人、截止条件 | 是整个任务完成，还是仅当前轮回答完成 |
| 模型调用 | 版本、输入引用、输出类型、耗时 | 不一定需要长期保存全部敏感内容 |
| 工具调用 | 调用标识、参数摘要、状态、结果引用 | 空结果、失败、超时和拒绝应可区分 |
| 外部操作 | 业务请求号、确认状态、执行结果 | 可能与任务记录跨服务，不能假定天然原子 |
| 用户确认 | 确认对象、版本、时间和身份 | 修改了草稿后，旧确认是否仍有效 |

这些是建议记录的字段类别，不是一套强制 schema。模型、框架和业务的具体协议可能不同。重要的是能够回答：**失败发生在哪一步、下一步能否安全重试、用户看到的状态是否真实。**

**容易忽略的细节：** 前端流式连接断开，不必然意味着后端任务已停止；后端任务标记取消，也不代表已发出的外部请求一定撤回。需要区分“停止后续工作”“尝试取消正在执行的调用”和“确认外部副作用是否发生”。

#### 3. Tool Schema 既是接口契约，也是模型使用说明

普通服务接口只面向程序员，而 Agent 工具的名称、描述和参数定义还要让模型知道何时使用。若有“查询订单”“搜索订单”“列出订单”三个工具，但描述和输入范围高度重叠，模型可能难以选择。

设计时可以从四个方面检查：

1. **职责是否清晰：** 一个工具做什么，不做什么；名称不要靠内部缩写猜测。
2. **参数是否自解释：** 时间格式、单位、取值范围、默认行为和必填项是否明确。
3. **返回是否可使用：** 区分成功、空结果、参数错误、权限拒绝和系统失败，避免模型根据含糊文本猜测。
4. **权限是否独立验证：** 工具描述和结构化参数可以提高使用正确性，但不能取代服务端鉴权。

参数校验可以限制字段类型与格式，但并不能证明查询范围合理；例如 `regionId` 是合法整数，不代表当前用户可以访问该区域。工具结果则应按需要返回足够的摘要和来源，避免无条件返回大量记录。Anthropic 的[工具设计经验](https://www.anthropic.com/research/writing-tools-for-agents)也建议重视工具职责、描述与响应内容，并通过评测验证改动。

**可做的评测：** 准备边界重叠的用户问题，逐条记录应选的工具、合理参数、允许的等价路径和禁止行为。修改工具描述前后用同一批数据比较；若某些任务本来就可由多个工具完成，不要强行要求唯一调用路径。

#### 4. Function Calling 与 MCP：标准化的是哪一段？

可以先从直接 API 场景理解：应用提供工具定义；模型给出工具调用请求；应用执行工具并回传结果。在这个过程中，模型接口、应用内部封装和真实业务服务各自承担不同职责。[OpenAI Function Calling](https://developers.openai.com/api/docs/guides/function-calling)给出了该产品的典型交互过程。

MCP 则提供一种协议，用于客户端与服务端之间发现和交互工具等能力。它可以减少不同集成方式的适配成本，但**它不替业务系统定义正确的财务规则，也不自动保证每位用户有权限执行工具**。服务端仍需按照自身架构验证令牌、身份和资源范围，应用端也应管理工具暴露范围。

遇到“有了 MCP 还要不要接口？”时，可以回答：MCP 服务本身可以封装已有 API、数据库或内部服务；底层能力并不会因为多了一层协议而消失。是否使用 MCP，取决于跨应用复用、现有生态、团队维护成本和性能要求，而不是协议越多越先进。[MCP 架构](https://modelcontextprotocol.io/docs/learn/architecture)

**安全追问：** 如果 MCP 服务可以访问内部网络或接受远程地址，还要结合具体部署考虑凭证范围、服务身份、令牌处理与请求目标校验。相关风险和适用条件以对应版本的 [MCP 安全实践](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices)为准。

#### 5. 记忆不是“把所有聊天记录存下来”

把全部历史塞进数据库，解决的是“保存”，不自动解决“什么时候取、取什么、取错怎么办”。设计记忆前，先区分：

- **当前任务状态：** 本次正在处理什么、哪些步骤完成、用户确认了什么。
- **近期对话：** 为了处理指代和追问，需要保留的少量相关上下文。
- **长期偏好或事实：** 用户长期有效且允许保存的信息；需要来源、更新时间和失效机制。
- **权威业务数据：** 余额、库存、权限和订单状态等应以业务系统为准，不应只依赖模型记忆。

例如，过去用户习惯看某区域数据，不表示今天也有该区域权限；用户一句“以后不再使用这个偏好”也应能覆盖或删除旧记录。记忆更新需要解决冲突、过期和错误来源，读取时要避免跨用户或跨租户混用。

**评测建议：** 准备条件继承、用户主动更改、偏好失效、身份变化和相互矛盾的记忆样本；检查系统是否选择了合适来源、必要时是否再次确认，以及有没有泄露其他用户信息。

#### 6. RAG 不准：应该从哪里开始排查？

假设系统引用了一篇过期规则并生成了错误回答。“回答错了”只是终点，至少有以下可能：

1. 新版文档没有进入知识库，或者权限不允许访问。
2. 新版进入了系统，但切分时把关键条件与标题拆开。
3. 正确片段被索引，但召回没有覆盖，或排序被旧版内容压过。
4. 正确片段已经进入上下文，但生成时引用错误或没有遵守条件。
5. 正确资料本身没有覆盖用户问题，却仍然生成确定结论。

因此应先保留可核对的原始问题、文档版本、召回列表及最终引用，再逐层判断。对于检索组件可设计覆盖目标文档是否进入候选的指标；对于最终回答则需要检查事实、引用、完整性和无法回答时的处理。**检索命中率高不等于回答一定正确**，回答正确也不说明引用路径一定可靠。

**追问：旧文档已经被删除，为什么还被检索到？**

需要查删除事件是否到达、索引是否更新、缓存是否失效以及用户权限过滤是否同步。具体一致性取决于系统实现；不能在没有证据时断言它是某个组件的故障。

#### 7. 长任务、检查点与副作用：如何设计可恢复边界？

以生成报告并发送为例，一个任务可能经历“收集数据、生成草稿、用户确认、发送、查询发送结果”。如果系统在发送请求后崩溃，只从“待发送”状态重新执行，可能造成重复发送。

应该分别考虑两类状态：一类是内部进度，例如草稿版本、已读取数据和下一步；另一类是外部结果，例如消息服务是否已接受请求。检查点可以帮助记录进度，但不能自动保证外部业务动作与本地记录在同一个事务中。

一种设计是为确认动作和业务请求分配稳定标识，并记录目标、草稿版本及允许的操作；恢复时先核对确认是否仍有效，再查询外部操作状态。若外部服务提供幂等接口，按其契约重用请求标识；若不提供，则需要明确不确定状态和人工处理边界。

**如何验证：** 在发送前、请求已发出但未收到响应、下游已成功但本地未记账等位置分别模拟故障，核对最终业务效果。Anthropic 关于[跨会话长任务](https://www.anthropic.com/research/effective-harnesses-for-long-running-agents)的讨论展示了保存进度与环境信息的一种方案；它不意味着所有外部副作用都能靠同一种检查点处理。

#### 8. 多 Agent 的难点是协作契约

如果将任务拆给“资料收集”“分析”和“审核”三个执行者，需要先定义每个子任务的输入、输出、完成标准和负责人。每个执行者独立上下文可以减少无关信息，但也可能让它缺失关键约束；共享状态便于协作，却会带来并发写入和覆盖风险。

可讨论的控制点包括：是否允许并行、共享状态谁能修改、如何表示冲突、子任务失败是否阻塞整体、取消是否传递、预算如何分配，以及最终结果由谁验收。让多个 Agent 相互评价也不保证正确：他们可能共享同样的错误前提。对于可验证的部分，仍应尽量依靠真实工具结果、测试和人工检查。

**什么时候不值得拆？** 如果任务很短、依赖关系紧密、结果无法并行验证，拆分可能只增加调用次数和等待时间。应该比较单执行者与多执行者的正确率、延迟、成本及维护复杂度，而不是预设角色越多越好。

#### 9. 评测不仅是一个“准确率”

一个 Agent 的质量至少涉及输入、过程、结果与成本。如果只看“最终回答对不对”，可能忽略它是否调用了不该调用的工具；如果只看工具参数，也可能漏掉最后的解释错误。

评测集应覆盖真实任务与边界：正常输入、缺少参数、含糊指代、权限不足、空数据、外部超时、冲突资料和恶意输入。对每个样本写清期望结果、允许的路径、禁止的行为和评审口径。一条问题可能有多种合理路径，不能把工具调用顺序不完全相同都算错。

模型评分适合辅助处理开放答案，但需要检查评分标准是否清晰、不同评分轮次是否稳定，并抽样人工复核。人工标签也可能出现分歧，需要记录争议而不是只保留一个数字。结果报告应包含样本规模、模型和工具版本、对照方式与失败分类；没有可比基线时不能声称提升百分之多少。

#### 10. 一题多问：为什么同一主题在不同岗位问法不同？

以“上下文管理”为例：应用开发可能更关注如何继承用户条件和减少不相关数据；Runtime 可能关注消息协议、压缩时机、恢复和成本；Coding Agent 可能关注代码定位、仓库规则和长任务状态；算法方向则可能进一步涉及模型能力、训练数据和评测。

准备时先画出自己能解释的系统，再把每个问题放回相应层次。知道术语并不等于必须声称自己开发过框架；反过来，也不必因为没有训练模型就忽略数据质量与评测基础。

#### 11. 停止按钮按下后，到底停止了什么？

流式 Agent 往往涉及浏览器连接、应用任务、模型请求、工具调用和外部服务。用户关掉页面或点击停止，首先发生的可能只是前端不再接收消息；后台是否停止，需要有明确的任务取消机制。

一种设计是：客户端发送带任务标识的取消请求，服务端先校验用户是否有权取消，再将任务置为“取消请求已接收”，通知能够响应取消的执行单元；随后记录实际停止结果。已发出的 RPC 或已提交的外部操作不一定能撤回，所以最后要区分“停止了后续步骤”和“之前所有动作都没有发生”。

如果用户断线重连，需要回答哪些事件已被接收、服务端是否继续执行、是否能够从某个事件位置恢复显示。消息重复或丢失时应如何处理，也要由传输协议与应用状态共同定义。不能简单认为使用 SSE 就天然实现断点恢复、持久化和恰好一次交付。

**如何验证：** 分别在等待模型、执行只读工具、执行写操作前后和最终响应中断时触发取消；检查任务状态、下游动作和用户看到的提示是否一致。若平台无法取消某些下游请求，应如实展示状态并避免继续执行后续有风险动作。

#### 12. Agent Loop 需要哪些终止条件？

模型可能因为目标不明确、工具错误、上下文缺失或指令冲突，不断重试同一步。控制循环至少要分别考虑：

- **完成条件：** 预期产物或业务状态是否已得到可验证证据。
- **预算条件：** 总耗时、步骤数、模型与工具调用次数是否超过限制。
- **无进展条件：** 连续多步相同错误、状态未变化或相同参数反复失败。
- **外部中断：** 用户取消、权限撤销、依赖不可用或服务关闭。
- **人工接管：** 关键风险较高、信息矛盾或多次尝试后仍无法判断。

检测重复时不能只比较工具名：连续翻页或按不同区域查询，可能是正常工作；比较参数、业务状态和进展更有意义。预算达到上限后，系统应保留已完成内容、说明未完成原因，并明确后续如何继续；不应把未执行的步骤描述成已经完成。

**对模型的理解：** 模型提出下一步是概率性输出，程序仍可对动作、参数和预算做确定性检查。两者协作并不要求模型永远自主运行。

#### 13. 评测的统计口径如何避免误导？

假设有 100 个测试任务，其中 80 个最终答案正确。这个数字本身不足以判断系统是否可上线：可能其中 20 个正确回答绕过了权限检查，也可能测试集只覆盖简单问题。

建议至少补充：样本来源与覆盖范围；模型、提示词和工具版本；任务成功的判定规则；错误类型；安全违规；响应时间及成本。对于回答开放、没有唯一答案的任务，需要明确评审标准和人工抽样方式；对于金额、权限或事实核对等可验证项目，优先使用可重复的程序检查或权威数据。

比较两个方案时，尽量使用同一组样本、相同环境和相同评分口径。若样本只覆盖一个业务部门，结论就只对该范围有参考意义。不能把主观感觉、不同样本上的数字或未经验证的自动评分，包装成严格的业务提升。

**追问：模型裁判给了高分，就可以不用人工了吗？** 不能。评分模型也会受提示、顺序和偏好影响。应对代表性样本进行人工复核，检查一致性和系统性偏差；高风险任务还应使用更可靠的规则或人工验收。

#### 14. 应用侧开发与模型训练，分别解决什么问题？

如果助手不知道最新业务规则，首先应核实资料来源、检索和工具接入，而不是默认必须重新训练模型。如果问题是输出格式不稳定，可先尝试约束输出结构、调整工具 schema、改进上下文和样本评测；这些方法也不是所有场景都有效，需要对比实验。

训练或微调可能适用于需要改善特定行为、风格或任务能力的场景，但需要合适的数据、可重复的评测和对照实验。偏好优化、强化学习等也依赖明确目标与可靠反馈；不同算法的适用条件不相同，不应把它们当作可以替换检索、授权和业务校验的万能方案。

对面试准备而言，应用开发者可以解释何时优先从工具、数据和上下文解决问题；训练岗位还需要进一步理解样本构造、优化目标、评估、训练与推理一致性以及成本。是否真的参与过模型训练，应结合自己的经历准确说明。

#### 15. Coding Agent 如何验证自己“改对了”？

一个编程任务的表面结果可能是代码提交成功，但验收还需要检查需求、改动范围和行为。可按以下思路组织：

1. **理解问题：** 确认期望行为、影响范围和现有约束，避免仅凭一个错误字符串修改。
2. **定位代码：** 结合精确搜索、调用关系、相关文档和实际执行路径；搜索结果只是线索。
3. **控制变更：** 先形成可复现的问题与验证方式，避免把无关格式化、重命名和功能修改混在一起。
4. **验证行为：** 针对改动运行合适的测试或检查，必要时加入真正覆盖故障的回归测试。
5. **人工审阅：** 解释为什么改、验证了什么、哪些环境或异常路径尚未验证。

长任务跨会话时，还需要保留任务目标、已完成步骤、验证结果和未解决问题；但保存过程信息不等于外部业务动作自动安全。相关设计必须结合仓库权限、工具权限和执行环境具体分析。

#### 16. 从一条面经到一组独立证据：怎样避免重复计算？

实践中可能遇到同一篇候选人面经被复制到多个平台，甚至被拆成几篇题单。建议给每条原始记录建立稳定标识，记录原帖、作者、发布时间、面试时间、岗位和轮次；如果两个链接指向同一经历，就归并到同一事件，而不是算成两次。

另一种情况是同一个候选人写了一面、二面两篇。两篇可以分别描述不同轮次，但在统计“独立候选人”时不能算作两个人。岗位官网更新也可能反复产生多个快照，比较要求时要注明具体时间和版本。

即便完成去重，公开发布的面经依然不是随机样本：愿意发帖的人、能被搜索引擎收录的平台以及受欢迎的题目都有选择偏差。结构化题库能够辅助发现重复主题，但在原帖不可见、模型 OCR 或归并存在误差时，不宜给出精确的行业概率。

#### 17. 公司画像应该如何表达才不夸大？

一份可靠的公司章节可以按“样本 → 观察 → 准备建议 → 不确定性”写，而不是先给公司贴标签。例如：

- **样本：** 某团队、某方向的一次候选人自述，发布日期和面试日期分别是什么。
- **观察：** 这次记录中提到了哪些主题；若只是合集转述，要注明未找到原帖。
- **建议：** 如果目标岗位与样本相近，哪些能力值得准备。
- **限制：** 样本数少、岗位类型不同、某些原文已不可读，不能代表全公司。

例如我们目前能读到的[京东实习记录](https://linux.do/t/topic/1591661)涉及运维、知识检索和业务接入；它可以帮助理解该候选人描述的那次面试，不能说明所有京东团队都采用同一技术栈或同一套题。岗位和团队仍以招聘方当前信息为准。

#### 18. 反问面试官：怎样确认岗位真实工作内容？

反问不是背固定清单，而是用来判断岗位和个人能力是否匹配：

- 团队的主要交付是业务功能、平台运行能力，还是模型训练与优化？
- 已有框架提供哪些能力，新成员主要负责业务接入还是底层建设？
- 现阶段任务失败通常如何发现，团队有什么验证或评测方式？
- 涉及写入或外部操作时，用户确认、权限与审计如何分工？
- 入职后第一个阶段通常负责什么，怎样判断交付完成？

这些问题不要求面试官披露敏感信息。追问应结合岗位阶段、对方愿意分享的范围和面试时间调整。


#### 19. 工程能力优先级：怎样将“都要学”变成可执行计划？

上面的内容范围很广，不同岗位不需要把每一条学到同样深度。可以把准备分为三层：

- **共同基础：** 能解释一次模型与工具交互、上下文如何组成、用户身份如何验证、结果怎样评估，以及系统失败时如何反馈。
- **岗位专项：** 应用岗深入业务集成与错误处理；知识岗深入资料生命周期与检索评测；运行时岗深入状态、调度、隔离与恢复；训练岗深入数据、实验与模型优化。
- **项目证据：** 选择自己做过的一个模块，能说明为什么这样设计、通过什么证据判断有效、还有哪些未知。

这个优先级是学习建议，而非样本统计。目标岗位如果明确要求模型训练或多模态，专项部分应相应调整。没有的实践可以通过实验补齐，但要明确区分学习实验和生产经历。

#### 20. 同一条执行链，如何准备五层追问？

假设系统根据用户问题查询数据并给出解释，可以按由浅入深的五层练习：

1. **功能层：** 用户输入是什么，系统最后返回什么？
2. **链路层：** 身份从哪里来，哪些组件参与，工具参数和结果如何流转？
3. **数据层：** 数据源是什么，更新是否及时，空值和多租户范围如何处理？
4. **故障层：** 接口慢、结果错误、模型重复调用时如何定位和控制？
5. **验证层：** 怎样设计测试样本，记录变更前后的可比较结果？

例如，系统回答正确但依赖的是过期数据，功能层可能看不出问题；只有追到数据来源和版本，才能判断该答案是否可用。又如一次请求在下游写入后超时，单看前端错误也无法知道业务状态。每一层都需要对应的日志、数据或测试作为支撑。

**口述训练：** 先用一分钟讲正常流程，再挑一个具体失败场景讲发现、定位、处理和遗留限制。若无法说明某个底层步骤，先标记待核实，不要为了让叙述更完整而编造机制。


### 补充：如何继续扩充原文引用的面经

本次从参考文章提供的链接出发，新增了可直接阅读的[京东技术运维／Agent 实习记录](https://linux.do/t/topic/1591661)、[蚂蚁智能体与大模型应用记录](https://ac.nowcoder.com/discuss/1646420?type=0)和[阿里云 Agent 算法记录](https://www.nowcoder.com/feed/main/detail/3a61d09fc2fb436f9bd5c1ed99b43330?sourceSSR=users)。前者帖子日期明确为 2026 年 2 月 10 日，后两条页面仅能确定月日，不能仅凭页面推断年份。

另有一些原文链接在本次访问时出现空页面、超时或需要额外访问条件。它们仍可作为后续线索，但在重新取得原帖之前，不应写成“本次已独立读到的证据”。其中一篇小米相关页面带有较长的问答和示例，是否为未经加工的一次面试记录无法确认，暂不将其纳入一手样本。


### 九、项目表达：事实、边界、证据

准备一个简短版本，覆盖用户、问题、系统主链路、个人工作和结果；再准备一个深入版本，能说明状态、错误处理、权限和评估。对于每个宣称的结果，问自己：基线是什么？样本有多少？统计口径与时间范围是什么？改进是否已经验证？

公开写作中不要放内部资料或个人敏感信息；没有可靠数字时直接说明。已有平台提供的能力、业务侧实现以及仍在设计的方案，应分别表达。

### 十、14 天通用能力学习安排

| 天数 | 主题 | 可验证的产出 |
| --- | --- | --- |
| 1–2 | 角色划分与目标 | 选两类岗位，列出交付物和能力差异 |
| 3–4 | Agent 与工具调用 | 一张调用链图，标明模型与程序的责任 |
| 5–6 | 状态与上下文 | 一份多轮指代、状态变更和压缩案例 |
| 7–8 | 检索与知识 | 一张索引到回答的链路图，以及三个错误案例 |
| 9–10 | 安全与可靠性 | 参数、超时、权限、副作用的故障处理表 |
| 11–12 | 评测与可观测 | 一组测试样本、指标口径及执行轨迹 |
| 13 | 专项能力 | 按应用、模型、运行时或 Coding 方向补充基础 |
| 14 | 项目复盘 | 用事实、个人职责、证据和限制完成一次讲述 |

### 十一、参考资料

**一手记录：**[京东 B8](https://linux.do/t/topic/1591661)、[蚂蚁 B9](https://ac.nowcoder.com/discuss/1646420?type=0)、[阿里云 B10](https://www.nowcoder.com/feed/main/detail/3a61d09fc2fb436f9bd5c1ed99b43330?sourceSSR=users)、[贝壳 B7](https://www.nowcoder.com/feed/main/detail/b8c08e8aa1974639965a795a2d7f82d8)、[快手 B1](https://www.nowcoder.com/feed/main/detail/4344d0da296944f8b6bcdb0acdbe141e)、[字节 B2](https://api-cdn.nowcoder.com/feed/main/detail/b34f6902e8544fe2953696ed52e49dba?sourceSSR=dynamic)、[阿里云 B3](https://www.nowcoder.com/feed/main/detail/e2d7d87cab1c41c6a70d9eef166ec1c1?sourceSSR=subject)、[美团 B4](https://www.nowcoder.com/feed/main/detail/58159306df52463ab75d72daa80d66df?sourceSSR=enterprise)、[百度 B5](https://www.nowcoder.com/feed/main/detail/fb9d83f0b6cd44b1ac3e3cafed6a1cf0)、[腾讯 B6](https://www.nowcoder.com/feed/main/detail/5a6705afdb45487e852c29bad27575d5)。

**官方与协议：**[字节 Seed 官方方向](https://seed.bytedance.com/zh/seedearlycareer)、[AWS AgentCore 岗位](https://www.amazon.jobs/en/jobs/10591991/software-development-engineer-agentcore-aws-agentic-ai)、[Function Calling](https://developers.openai.com/api/docs/guides/function-calling)、[Retrieval](https://developers.openai.com/api/docs/guides/retrieval)、[Agent 安全](https://developers.openai.com/api/docs/guides/agent-builder-safety)、[Agent Evals](https://developers.openai.com/api/docs/guides/agent-evals)、[MCP 架构](https://modelcontextprotocol.io/docs/learn/architecture)。

**旁证与方法参考：**[开源面经题库](https://github.com/smile-struggler/kaomian)、[知识库岗位快照](https://jobs.ultraai.site/jobs/bytedance/7644505471581849909)、[跨平台岗位快照](https://jobs.ultraai.site/jobs/bytedance/7683834512367388933)、[Liuliming 的面经调研文章](https://keepkeen.github.io/blog/china-agent-interview-guide-2025-2026/)。


### 工程面试与项目复盘

这篇笔记面向准备 Agent 应用开发岗位的后端工程师。它参考了 [Liuliming 的 Agent 面经整理](https://keepkeen.github.io/blog/china-agent-interview-guide-2025-2026/)，也结合官方技术文档重新组织。下文中的业务场景都是为了说明问题而构造的例子，不代表某个真实项目或企业的面试要求。

学习时不必先背全所有框架：选一个能讲清楚的场景，沿着「用户问题 → 模型决策 → 工具执行 → 结果反馈 → 异常处理」梳理，通常比罗列名词更容易建立整体理解。

### 一、先理解 Agent 在系统里做什么

假设用户说：“帮我找出最近一周销量下降的商品，解释可能原因。”一个应用可能先识别用户意图，查询允许访问的商品数据，再结合分析结果组织回答。这个过程中，模型可以帮助理解问题、决定下一步需要什么信息或生成说明，但真实数据查询、权限控制和业务规则通常仍由外部程序承担。

可以将一个典型系统拆成几部分：

- **模型**：根据输入生成文本或结构化的工具调用请求。
- **工具**：提供查询、计算等具体能力，例如查销售数据。
- **运行时**：负责将模型请求转换成实际调用、管理调用结果，并决定是否继续执行。
- **上下文**：包含当前问题、必要的历史、工具说明和执行结果。
- **业务控制**：负责身份、权限、数据范围、超时和审计。

不同产品对 Agent 的定义和自主程度并不完全一致。面试时先讲清自己的系统允许模型做什么、哪些步骤由程序固定，就比争论名字更有帮助。

### 二、Agent 和固定工作流如何选择

**固定工作流** 适合步骤稳定、边界明确的任务。例如，按固定顺序完成参数校验、查询和结果展示。它的路径容易预测，便于测试和排障。

**模型参与决策的流程** 适合用户表达多样、需要根据中间结果选择下一步的问题。例如，模型可能先查销量，再根据缺失的信息决定是否查询库存。但选择范围需要被工具列表、权限和执行预算约束。

实际系统可以混合使用：身份校验与数据权限使用确定性逻辑，信息组织和工具选择交给模型。是否采用更自主的流程，要看问题复杂度、错误成本、可观测性和维护成本。

### 三、一次工具调用到底发生了什么

以查询商品销量为例，一次常见调用可以拆成：

1. 应用向模型描述可用工具，以及工具所需参数。
2. 模型根据问题提出调用请求，例如商品范围和时间范围。
3. 应用校验参数、用户身份和数据权限，决定是否允许执行。
4. 应用调用真正的数据服务，得到结果或错误。
5. 应用把结果返回给模型，模型据此继续调用或生成回答。

这里要强调：**模型生成工具调用请求，并不等于它已经直接执行了业务接口**。在常见的函数调用模式下，执行和结果回传由应用完成，具体机制可参阅 [OpenAI Function Calling 文档](https://developers.openai.com/api/docs/guides/function-calling)。

#### 失败时要考虑什么

- **参数不完整或不合法**：先校验，必要时向用户澄清，不能默默猜测关键条件。
- **接口超时**：区分读操作和写操作；重试需要预算、退避和明确的终止条件。
- **数据为空**：空结果、接口失败和没有权限是不同状态，不应合并成“没有数据”。
- **可能重复执行**：对写操作设计幂等机制，并明确确认与重试的边界。
- **数据量过大**：限制返回条数和字段，避免把无关数据全部交给模型。

面试中可以拿自己熟悉的后端经验作类比：模型给出的是调用意图，而网关和业务服务仍要做身份验证、参数检查、限流与异常处理。

### 四、工具、MCP 和业务流程如何区分

工具是一个可执行能力，例如“按时间范围查询销量”。业务流程是完成目标的一组步骤，例如“选定商品 → 查询销量 → 比较变化 → 输出解释”。MCP 是连接 AI 应用与外部能力的一种协议，提供能力发现与交互的标准化方式；它不自动替代权限校验，也不保证工具一定可靠。协议架构可参考 [MCP 官方文档](https://modelcontextprotocol.io/docs/learn/architecture)。

Skill 一词在不同产品中含义可能不同，可以先把它理解为围绕某类任务组织的说明、资源或操作方式；它的实际加载与执行机制，应以使用的平台文档和具体实现为准。

**回答示例：**“我会先说明工具做什么，再说明平台如何发现和调用工具，最后区分业务流程及权限控制。协议统一了交互方式，但不能替业务系统决定用户是否有权限。”

### 五、上下文、会话和记忆

一次请求送给模型的内容，可能包括当前问题、系统规则、历史消息、工具描述和工具结果。它们共同构成这次调用的上下文。**会话** 用于组织多轮交互；**记忆** 则通常指需要在更长时间或跨会话保留的信息。不同系统的实现和保存范围可能不同。

上下文变长时，关键问题不是简单地“全部保留”或“全部删除”，而是判断哪些信息对当前任务仍有用：

- 重要的用户约束和授权范围不能随意丢失。
- 较早的重复对话可以考虑摘要，但摘要可能遗漏细节。
- 大型工具结果可以筛选必要字段，同时保留可追溯的来源。
- 摘要或历史数据过期时，应有重新查询和纠正的机制。

如果没有参与过具体实现，面试中应把这些讲成设计思路，而不是宣称项目已经实现了对应机制。

### 六、RAG：把检索结果交给模型之前

检索增强生成通常包含：准备资料、建立索引、根据问题召回内容、选择适合的片段，再将其作为上下文生成回答。不同产品可以采用不同的切分、检索和排序方式，参见 [OpenAI Retrieval 文档](https://developers.openai.com/api/docs/guides/retrieval)。

评估问题时，要区分几个环节：

- **没有找到正确资料**：可能是资料缺失、索引过期或检索条件不合适。
- **找到了但排得太后**：可以检查召回与排序策略。
- **资料正确但回答错误**：需要检查上下文组织和模型生成过程。
- **资料本身过期或相互矛盾**：要核对来源、时间和适用范围。

例如，用户问“当前流程如何处理”，旧版流程即使相关，也可能误导回答。工程上需要考虑文档更新时间、版本和权限，而不仅是相似度。

### 七、如何评测 Agent

先定义目标，再决定指标。对于一个查询型助手，可以从以下几个层次观察：

| 层次 | 可以检查什么 |
| --- | --- |
| 结果 | 是否回答了用户问题，关键事实是否正确，是否标明不确定性 |
| 过程 | 是否选择了合适工具，参数是否正确，是否出现多余调用 |
| 工程 | 成功率、超时率、耗时、调用次数和成本 |
| 安全 | 是否越权、泄露无关数据或执行未经确认的高风险操作 |

一套可复用的测试集可以包含正常问题、缺少参数、无权限、空结果、工具失败等情况。每次调整模型、提示词或工具后，比较同一批样本，并抽查失败案例。评测需要记录样本来源、标准和版本，避免只展示少量成功例子。关于流程级评测，可参考 [OpenAI Agent Evals 文档](https://developers.openai.com/api/docs/guides/agent-evals)。

### 八、项目深挖怎么讲

建议准备两个版本：

**两分钟版本：** 先说业务问题和目标用户，再说系统的大致链路，接着说明自己负责的部分，最后讲一个可核实的结果或遇到的困难。

**深入版本：** 准备一张调用链图，并能回答：身份和权限从哪里来？工具失败后如何处理？如何判断回答质量？哪些方案上线了，哪些只是建议？

如果没有完整统计数据，可以明确说“目前没有可靠数据”或“这是方案设计，尚未验证”，再说明你准备怎样测量。不要把团队或平台的整体能力全部说成个人实现。

### 九、给自己的练习题

1. 画出你熟悉的一个 Agent 场景，从用户输入一直画到工具结果返回。
2. 任选一个工具，列出参数缺失、超时、空结果和无权限时的处理方式。
3. 解释上下文和长期保存的信息有什么不同，各自应保留什么。
4. 找一个检索回答错误的案例，分析是数据、召回还是生成环节的问题。
5. 设计十条测试样本，并为每条写出可判断的预期结果。
6. 用两分钟介绍自己的项目，再补充一个失败案例和改进思路。

### 十、先判断你准备的是哪一类岗位

同样叫 Agent 工程师，不同岗位需要的能力差别很大。读岗位描述时，先看它要交付什么，再决定投入比例。

| 岗位方向 | 主要关注 | 后端开发者的准备重点 |
| --- | --- | --- |
| Agent 应用开发 | 把智能能力接入真实业务，完成工具调用和用户交互 | 业务链路、接口、权限、可靠性、评测 |
| Runtime 与基础设施 | 提供任务执行、状态管理、工具接入和运行环境 | 并发、隔离、恢复、可观测性和资源控制 |
| 检索与知识应用 | 把企业资料转成可检索、可引用的回答 | 文档处理、检索质量、权限、更新与评测 |
| 模型训练与优化 | 提升模型特定能力，涉及数据、训练和实验 | 训练基础、数据质量、指标与实验设计 |

对于以 Java 后端和业务 Agent 为主要经历的候选人，可以先准备应用开发主线，再按目标岗位补足检索、评测或基础设施。不必把自己没有做过的训练工作包装成项目经验。

#### 如何看待网上的面经

公开面经适合作为找题线索，但不能保证题目真实、完整或对所有团队有效。看见“某公司必考某题”时，先检查来源、时间、岗位和面试者经历；无法核对的内容，用来练习即可，不要当成确定事实。原文列出了它采用的来源和证据边界，值得结合上下文阅读。

### 十一、十个值得练习的面试问题

以下是学习练习题及参考思路，不表示某公司真实考过。每道题要尽量结合自己的实际系统回答。

#### 1. 怎样判断一个任务需要 Agent？

先看任务是否需要根据中间结果选择下一步，以及这种灵活性是否能带来业务价值。如果执行顺序固定、失败成本高，确定性工作流通常更容易控制。即便需要模型决策，也可以只让它选择低风险步骤，高风险操作交给程序校验或人工确认。

**追问：** 模型判断错了会怎样？能不能给一个固定流程已经足够的反例？

#### 2. 如何让模型调用工具？

把工具名称、功能、参数约束提供给模型；模型提出调用请求后，应用校验并执行，再把工具结果返回。关键不是只让模型输出合法参数，还要确认用户身份、查询范围和实际执行权限。

**追问：** 模型填了不存在的商品编号，接口超时或返回空列表时分别如何处理？

#### 3. 如何处理可能产生副作用的操作？

把查询和写入区分开。写入前校验身份、权限、参数及业务状态；对关键操作展示确认信息；执行时使用适合业务的幂等键、审计记录和失败恢复机制。超时后不能直接假设操作失败，因为请求可能已被下游处理。

**追问：** 用户点击确认后网络断开，如何确认订单到底有没有创建？

#### 4. 模型一直重复调用工具怎么办？

先记录执行轨迹，区分是参数反复错误、工具结果无效、模型无法理解结果，还是任务本身无法完成。工程上可以设置最大调用次数、耗时和成本预算；对重复请求做检测，达到阈值后停止并解释原因。次数限制是兜底，还需要定位造成重复的具体原因。

**追问：** 如何避免误伤同一个工具上必要的多次分页查询？

#### 5. 上下文太长时如何处理？

先识别不可丢失的约束，再筛选和压缩历史信息。必要时保留摘要、最近对话和可重新查询的数据引用。压缩可能引入信息遗漏或错误，关键参数和用户授权不宜仅依赖模糊摘要。

**追问：** 摘要写错了用户选择的区域，怎样发现并纠正？

#### 6. 多轮对话中如何处理指代和记忆？

“继续推荐”这类请求往往依赖前文。可以在有明确范围和权限的前提下，从会话状态中读取上一次的条件；如果关键条件缺失或有歧义，应向用户确认。长期保存的偏好与本轮临时条件要区分，用户修改条件时应有明确的优先级。

**追问：** 用户更换了身份或经营区域，旧的默认值还能继续用吗？

#### 7. 检索结果相关但答案还是不对，怎么定位？

逐层检查原始文档是否正确且足够新、召回片段是否覆盖答案、重排是否保留关键内容、模型是否真正依据片段作答。记录问题、检索结果、版本和最终回答，用可重复的样本对比改动效果。

**追问：** 两份文档相互矛盾，但都能检索到，系统应该怎么回答？

#### 8. 怎样评测一个工具型 Agent？

先选具有代表性的样本，并为每条定义预期行为，如是否应调用工具、参数条件和最终回答的关键事实。再分别统计工具参数错误、调用失败和回答质量；高风险或难以自动判定的场景需要人工检查。不要仅以“模型说自己完成了”作为成功标准。

**追问：** 模型回答正确，但过程调用了一个不该调用的工具，这条算成功吗？

#### 9. 什么时候有必要引入多个 Agent？

当任务存在可分工的独立子问题，且收益能够覆盖协调、延迟和失败排查成本时，可以考虑多个执行者。例如一个负责收集资料，另一个负责校验。但多 Agent 并不自动提高正确性，任务边界、信息传递和最终责任仍需要设计。

**追问：** 如果两个执行者给出冲突结论，由谁裁决，依据是什么？

#### 10. 如何让 Agent 系统方便排查问题？

为每次请求建立可关联的标识，记录必要的模型调用、工具名称、参数校验结果、耗时、错误类型和重试情况。日志要注意脱敏和访问权限。排查时先判断是意图识别、工具执行、数据质量，还是回答生成环节出了问题。

**追问：** 用户只说“答案不对”，你需要哪些信息才能复现？

### 十二、一次项目追问如何展开

以下是虚构示例，只用于练习回答结构：某商家希望根据经营情况获得商品建议。系统需要理解需求，查询允许访问的数据，再组织推荐结果。

面试官可能从四个角度追问：

1. **业务价值**：原来的操作有什么困难？系统解决了哪一段问题？
2. **技术链路**：请求如何进入，模型能调用哪些能力，结果如何展示？
3. **异常和边界**：数据查不到、查询超时、用户条件不完整时分别怎么办？
4. **个人贡献和结果**：你具体实现或改进了哪部分？结果有什么证据？

准备时可以给每个问题写出一张小卡片：**事实、证据、限制**。例如“接口耗时降低”需要说明指标口径和比较范围；没有观测数据时，不要编造改善比例。公司内部信息是否能公开，也应事先确认。

### 十三、14 天准备路线

这是练习安排，不代表必须每天完成固定时长。已有经验的人可以跳过熟悉部分，把更多时间留给项目深挖和模拟面试。

| 天数 | 学习主题 | 当天产出 |
| --- | --- | --- |
| 第 1 天 | 选定目标岗位与一个真实项目 | 写清岗位要求、项目边界和个人职责 |
| 第 2 天 | 画项目主链路 | 一张从用户输入到结果展示的流程图 |
| 第 3 天 | 梳理模型与程序各负责什么 | 一份责任边界清单 |
| 第 4 天 | 工具调用 | 一个正常调用和三个异常场景 |
| 第 5 天 | 权限与安全 | 身份来源、数据范围、高风险操作确认 |
| 第 6 天 | 上下文与多轮对话 | 一组带指代和条件变更的例子 |
| 第 7 天 | 记忆与状态 | 说明保存范围、更新方式和过期处理 |
| 第 8 天 | 检索基础 | 画出文档到检索结果的处理链路 |
| 第 9 天 | 检索排障 | 分别构造数据缺失、召回错误和回答错误案例 |
| 第 10 天 | 评测设计 | 十条样本与判断标准 |
| 第 11 天 | 可观测性 | 一份工具调用轨迹和排障清单 |
| 第 12 天 | 项目表达 | 两分钟讲解稿与常见追问 |
| 第 13 天 | 模拟面试 | 记录不清楚的问题与需要核实的事实 |
| 第 14 天 | 查漏补缺 | 精简答案、复盘错误，并确认对外表述边界 |

完成一遍之后，不要只看是否记住了术语：尝试脱稿画出调用链，解释一个故障如何定位，以及自己在项目中承担了什么。

### 参考资料

- [Liuliming：2025–2026 国内大厂 Agent 面经](https://keepkeen.github.io/blog/china-agent-interview-guide-2025-2026/)：本文的选题参考；具体面经请回到原文核对来源与时间。
- [OpenAI：Function Calling](https://developers.openai.com/api/docs/guides/function-calling)
- [OpenAI：Retrieval](https://developers.openai.com/api/docs/guides/retrieval)
- [OpenAI：Agent Evals](https://developers.openai.com/api/docs/guides/agent-evals)
- [Model Context Protocol：Architecture](https://modelcontextprotocol.io/docs/learn/architecture)


### Java 后端学习路线

> 调研时间：2026 年 11 月 15 日。本文面向有后端开发经验、希望准备 Agent 应用岗位的读者。岗位与面经是有限公开样本，链接和招聘状态可能变化；技术讲解和准备建议是对资料的归纳，不代表企业官方面试标准。

### 一、结论先行

1. **先看岗位要做什么，再决定复习范围。** 同为 Agent 岗位，有的侧重业务应用，有的侧重知识库、研发工具或运行时基础设施。把所有岗位看成一种，会使准备范围过大。
2. **后端基础仍有价值。** 本次检索到的岗位快照和候选人记录涉及接口、数据库、并发、消息、权限或流式交互；但具体考察哪些内容，取决于岗位和候选人经历。
3. **项目讲述要能落到一次真实执行。** 准备用户问题、执行链路、自己负责的环节、异常处理和效果验证；不要因为接触了某个平台，就说自己实现了整个平台。
4. **工具调用、上下文和评测值得优先准备。** 这是一项基于本次有限样本和应用岗位需求的准备建议，不是对全行业出题频率的统计结论。
5. **岗位信息、候选人回忆和技术文档是不同证据。** 岗位描述说明企业公开列出的职责，个人面经反映单次体验，官方技术文档说明某项技术的机制；三者不能相互替代。

### 二、调研范围与证据说明

本次定向检索了公开招聘页面、牛客面试记录，以及模型工具调用、检索和评测的官方文档。我们没有取得企业内部题库，也没有用科学抽样统计出题频率。

| 类型 | 本次如何使用 | 限制 |
| --- | --- | --- |
| 候选人自述 | 观察具体问题和追问方式 | 来源是个人回忆，无法独立核实全部过程 |
| 面经合集或付费预览 | 发现主题线索 | 可能转述、混入答题建议或只展示部分内容 |
| 第三方岗位快照 | 比较公开列出的职责和要求 | 可能过期、下线或与官网不一致 |
| 官方技术文档 | 核对具体技术机制 | 只代表对应产品或协议，不能推出所有系统的实现 |

以下来源均为本次检索中可访问的内容；没有把一个合集中的多篇转述计算成独立的一手面经。

### 三、公开样本观察

#### 1. 候选人记录：项目与基础可能交叉出现

一篇[字节 AI 应用开发一面记录](https://api-cdn.nowcoder.com/feed/main/detail/b34f6902e8544fe2953696ed52e49dba?sourceSSR=dynamic)自述面试发生在 2026 年 8 月 27 日。作者记录的问题既有 Agent 执行循环、长任务取消、上下文和记忆，也包括 Java ThreadLocal 和 Spring 基础。这能说明**该次记录中** 应用能力与后端基础都被问到，但不能据此推断字节所有团队的统一面试流程。

另一篇[字节后端／Agent 开发一面记录](https://www.nowcoder.com/feed/main/detail/a3a5ae8345fc402c98cb8bf125e0b499?sourceSSR=subject)由作者描述为近期秋招面试，具体面试日未披露。帖子提到数据库优化、消息可靠性、缓存、系统设计以及上下文窗口问题。它更偏后端与系统设计，提醒我们即使岗位名称带 Agent，不同团队的考察侧重也可能差异很大。

一篇[标称社招一面的公开预览](https://api-cdn.nowcoder.com/discuss/926968209813639168?sourceSSR=post)列出了异步任务状态、工具返回数据过大、超时重试和评测等追问；该页面属于付费专栏，正文未完整开放。我们仅把可见部分当作**需要审慎核实的线索**，不把它当作已验证的整体题单。

#### 2. 岗位快照：相同名称可能覆盖不同交付物

| 公开来源 | 页面中的方向 | 可观察到的要求 | 证据边界 |
| --- | --- | --- | --- |
| [字节知识库方向岗位快照](https://jobs.ultraai.site/jobs/bytedance/7644505471581849909) | 社招、研发知识库 | 涉及知识、工具、评测和应用研发 | 第三方聚合，收录时间为 2026 年 8 月 3 日；当前在招状态需核实 |
| [字节跨平台 Agent 岗位快照](https://jobs.ultraai.site/jobs/bytedance/7683834512367388933) | 社招、开发工具与跨平台 | 涉及执行控制、沙箱、异常恢复和评测 | 第三方聚合，收录时间为 2026 年 9 月 10 日；不能代表全部后端岗位 |
| [腾讯 Agent 应用岗位快照](https://offerseek.ai/jobs/248405) | 社招、社交产品应用 | 涉及任务规划、记忆、知识检索和评测 | 第三方聚合，页面显示 2026 年 10 月 9 日更新；是否仍在招聘未核实 |
| [字节 Agent 应用岗位快照](https://jobs.ultraai.site/jobs/bytedance_campus/7667851701248739637) | 校招、应用工程 | 提及后端基础与工具调用、检索等应用能力 | 校招样本，不能直接等同于社招要求 |

这些记录适合用来判断“有哪些可能的职责组合”，不适合推断录用标准或公司统一偏好。投递时，应重新查看企业官方招聘页面和目标团队的最新要求。

### 四、面向后端开发者，怎么拆解岗位

#### 业务应用型

关注点是把模型、业务数据和用户操作串起来：需求理解、工具接入、结果呈现和失败处理。可以重点准备：如何判断能力边界、如何保证参数与身份正确、怎样把错误反馈给用户，以及如何验证业务效果。

#### 知识与检索型

关注资料从哪里来、如何更新、用户能查哪些内容，以及回答如何引用可核对的来源。准备时把问题拆成文档处理、召回、排序、上下文组织和回答校验，不要用“加一个向量库”概括全部工作。

#### 运行时与研发工具型

关注执行环境和长任务：任务状态、工具调度、中断恢复、隔离、权限和成本。这类岗位会与并发、分布式系统及工程工具联系得更紧，但不能假设每个应用开发岗位都要求实现底层运行时。

#### 算法和训练型

更可能需要训练数据、实验设计、模型优化和评估基础。如果自己的经历主要是业务应用，可以理解基本概念，但要将已有工作与没有参与过的训练工作分开表述。

### 五、从一个业务问题看完整调用链

以下为**虚构示例**：用户提出“查看最近一周某类商品的销售变化”。系统先确定用户和数据权限，再补齐时间、范围等条件；如果条件足够，应用让模型在允许的工具中选择合适能力，校验参数后调用业务服务；拿到结果后，根据需要进行确定性计算，再生成文字说明。

这里至少有四个责任边界：

1. **身份与权限** 由应用和业务服务校验，不能只相信模型生成的用户标识。
2. **工具调用** 是模型提出意图、应用执行实际操作；参数格式正确不等于业务权限正确。[Function Calling 官方说明](https://developers.openai.com/api/docs/guides/function-calling)
3. **计算结果** 由可验证的程序或可信数据服务产生，模型负责解释时也要避免超出数据范围。
4. **回答质量** 需要检查事实、缺失信息和不确定性，不能仅以语句流畅作为正确标准。

真实系统不一定采用这一流程。准备自己的项目时，应根据代码、日志和实际职责说明每一步，而不是套用示意图。

### 六、八组面试追问与回答思路

下面是根据前述公开样本和工程问题整理的**练习题**，不是某公司的真题清单。

#### 1. 模型决定调用工具后，谁来执行？

参考思路：模型产生调用意图和参数；应用或运行时验证请求并执行具体服务，之后把结果返回。再说明你使用的平台如何实现，以及你负责了哪一段。追问时重点准备参数校验、身份传递和工具返回值如何表示。

#### 2. 工具返回空结果、超时和无权限，回答应该一样吗？

不应该。空结果表示查询成功但没有匹配项；超时表示结果未知；无权限表示不能访问。应用应保留不同状态，避免把技术失败或权限拒绝解释成“数据不存在”。

#### 3. 工具超时能不能重试？

先区分读和有副作用的写操作。读操作还要考虑重试预算、退避和下游承载；写操作超时后可能已执行，需要查询状态、幂等保障或业务补偿。重试不是必然正确的通用答案。

#### 4. 上下文装不下全部工具结果怎么办？

先限制返回范围和字段，在数据源支持分页、筛选或聚合时尽量前置处理。保留必要的结构化信息和来源，必要时按需继续查询；摘要只是可选方案，也可能丢失细节。比较方案时应考虑数据准确性、隐私、耗时与费用。

#### 5. 会话、上下文和记忆有什么区别？

一次模型调用接收到的信息是上下文；会话组织多轮交互；记忆通常指需要持续保存和复用的信息。具体边界由平台实现决定。回答中应说清保存什么、何时读取、何时过期，以及用户更改条件如何覆盖旧值。

#### 6. 工具与 MCP 是一回事吗？

工具是一项具体能力，MCP 是连接应用和外部能力的一种协议。协议支持能力发现和交互，但业务授权、工具结果质量与高风险操作确认仍需系统处理。[MCP 架构说明](https://modelcontextprotocol.io/docs/learn/architecture)

#### 7. 如何定位 RAG 回答错误？

沿资料源、索引更新、召回、排序、上下文组织和最终生成逐层排查。记录问题、命中的资料版本与回答，先判断“资料有没有”和“系统有没有找对”，再讨论生成环节。[Retrieval 文档](https://developers.openai.com/api/docs/guides/retrieval)

#### 8. 如何证明 Agent 改动有效？

先固定评测样本、版本和判断口径，比较任务结果与执行过程。查询类场景可观察工具选择、参数正确性、权限处理、回答事实、耗时和失败类型；复杂结果需要适当人工复核。样本少或没有线上实验时，要明确结论范围。[Agent 评测文档](https://developers.openai.com/api/docs/guides/agent-evals)

### 七、Java 后端经验怎样转成 Agent 工程回答

| 后端经验 | 可以映射的 Agent 问题 | 回答时要补充 |
| --- | --- | --- |
| RPC 与接口治理 | 工具接入、超时、降级 | 模型输出不确定，参数和状态需要明确校验 |
| 幂等与消息处理 | 工具重复调用、副作用控制 | 先区分读写，解释执行状态未知时如何确认 |
| 缓存与数据库 | 会话状态、数据查询 | 区分临时上下文、持久状态和事实数据 |
| 日志与链路追踪 | 工具执行轨迹、故障定位 | 注意敏感信息脱敏和可访问范围 |
| 权限系统 | 细粒度工具与数据授权 | 不能把模型生成的身份当成可信凭证 |

这是一种准备视角，不表示所有 Agent 系统已经采用了表中的机制。

### 八、项目准备：从真实事实出发

可以先为自己的项目写下五个问题：

1. 用户是谁，原本遇到什么问题？
2. 哪些能力由已有平台提供，业务侧新增了什么？
3. 从输入到响应，自己负责的环节有哪些接口、校验和异常路径？
4. 有哪些可展示的证据：代码、测试、日志、上线记录或指标口径？
5. 哪些是已经实现，哪些是后续方案，哪些仍未验证？

面试表达时，用一个真实案例串起链路；公开发布时去掉不适合披露的业务信息。没有可靠指标就说明没有，不把团队成果或平台能力全部归到个人名下。

### 九、两周准备计划

| 时间 | 重点 | 可检查的产出 |
| --- | --- | --- |
| 第 1–2 天 | 阅读目标岗位，确定经历边界 | 一份岗位能力表和个人职责清单 |
| 第 3–4 天 | 梳理项目与工具调用 | 一张链路图、一份工具入参与异常表 |
| 第 5–6 天 | 权限、超时、重试与幂等 | 每项至少一个真实或明确标注为设计的例子 |
| 第 7–8 天 | 多轮上下文与检索 | 一组条件变更、召回错误和过期资料案例 |
| 第 9–10 天 | 评测与可观测 | 十条样本、判断标准及失败分类 |
| 第 11–12 天 | Java 后端基础与系统设计 | 数据库、缓存、消息、并发与流式接口复盘 |
| 第 13 天 | 两分钟项目陈述和追问 | 录音或文字稿，标明不确定之处 |
| 第 14 天 | 模拟面试及复盘 | 一份待补证据、待验证问题清单 |

### 十、参考资料与时效

- [字节 AI 应用开发候选人记录](https://api-cdn.nowcoder.com/feed/main/detail/b34f6902e8544fe2953696ed52e49dba?sourceSSR=dynamic)：个人回忆，记录面试日为 2026 年 8 月 27 日。
- [字节后端／Agent 开发候选人记录](https://www.nowcoder.com/feed/main/detail/a3a5ae8345fc402c98cb8bf125e0b499?sourceSSR=subject)：个人回忆，具体面试日期未披露。
- [字节 Agent 社招面经公开预览](https://api-cdn.nowcoder.com/discuss/926968209813639168?sourceSSR=post)：付费专栏的有限公开内容，仅用于发现线索。
- [字节知识库岗位快照](https://jobs.ultraai.site/jobs/bytedance/7644505471581849909)、[字节跨平台 Agent 岗位快照](https://jobs.ultraai.site/jobs/bytedance/7683834512367388933)、[腾讯 Agent 应用岗位快照](https://offerseek.ai/jobs/248405)：第三方聚合，岗位状态需到官方渠道复核。
- [OpenAI Function Calling](https://developers.openai.com/api/docs/guides/function-calling)、[OpenAI Retrieval](https://developers.openai.com/api/docs/guides/retrieval)、[OpenAI Agent Evals](https://developers.openai.com/api/docs/guides/agent-evals)、[MCP Architecture](https://modelcontextprotocol.io/docs/learn/architecture)：技术机制参考。
- [Liuliming 的 Agent 面经整理](https://keepkeen.github.io/blog/china-agent-interview-guide-2025-2026/)：调研结构的参考。本文的样本选择、归纳和建议为重新整理，不将其公司的结论直接当作本次调研结果。
