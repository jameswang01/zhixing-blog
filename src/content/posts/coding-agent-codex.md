---
title: "Codex：工具路由、执行与上下文压缩"
description: "从公开 Rust 代码梳理工具调用的职责划分。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 先看结论

Codex 的公开仓库把工具定义、路由、执行编排和上下文压缩分成不同模块。本篇基于提交 `ee344ca`，重点是这些边界，不把仓库代码自动等同于某个线上产品的实际配置。

## 工具先被注册，再被路由

`ToolRegistry` 管理工具及其运行时能力；`ToolRouter` 构造调用并派发，也提供工具是否支持并行的查询。这里要区分两个问题：模型看到哪些工具，与程序如何处理模型提出的调用。[工具注册](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/registry.rs) · [工具路由](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs)

一条适合阅读的链路是：工具定义 → 模型返回调用 → 路由器识别工具 → 对应处理器执行 → 结果回到对话。具体工具可能走不同处理器，不能把某条命令执行路径概括为所有工具的共同实现。

## 执行编排与安全边界

`ToolOrchestrator` 是执行编排相关代码的入口之一。审批、沙箱和具体运行时还分布在其他模块中。模型发出调用请求并不意味着动作已经获得许可；要追踪某个动作，应继续沿处理器、审批策略和运行时读取。[执行编排](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs)

例如分析执行 Shell 时，需要核对命令、工作目录、权限策略、沙箱环境和最终执行结果。单独看到一个工具 schema，不能证明该命令可以访问哪些文件或网络。

## 压缩是状态变化，不是把数据库清空

仓库中有独立的上下文压缩模块。阅读时应区分模型下一轮能看到的内容、会话记录以及压缩过程生成的结果，不要仅凭模块名称断定历史存储策略。[压缩代码](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/compact.rs)

## 验证边界

本文为静态源码阅读，没有运行完整客户端或验证每个产品入口的策略。不同版本和部署配置可能使用不同能力；线上故障应以当时版本与实际配置为准。


## 从响应对象到工具调用：逐步追代码

从 [`ToolRouter::build_tool_call`](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs#L278-L349) 看起，它匹配不同的模型响应类型。普通 `FunctionCall` 会提取工具名、命名空间、参数和调用 ID，并封装成统一的 `ToolCall`；自定义工具调用走另一种 payload；某些不属于客户端执行的搜索调用则不会构造成普通客户端调用。

这层解决的是“**把不同响应表示转换为运行时能识别的调用**”。它本身不是业务权限系统：要判断一个命令最终能否访问文件，需要继续读注册信息、处理器、审批策略和运行环境。

可以这样画一条排查路径：

```text
模型响应 → build_tool_call → ToolCall（名称、ID、payload）
        → 工具注册与处理器 → 执行编排 → 结果返回
```

这个图是帮助阅读的简化关系；不同工具可能使用不同处理器，不能把它当成所有工具的完整时序。[路由源码](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs) · [注册源码](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/registry.rs)

## 注册工具：模型可见不等于一定可并行

`ToolRegistry` 管理工具定义和处理器。其 `supports_parallel_tool_calls` 查询会结合工具暴露状态与运行时能力；`ToolRouter::tool_supports_parallel` 再使用注册表判断。[注册表](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/registry.rs#L753-L755) · [路由查询](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs#L267-L270)

对实际排障来说，至少要区分：工具是否注册、是否对模型暴露、是否支持并行、参数是否有效、是否允许在当前环境执行。把这些都统称“工具调用失败”，往往会丢掉定位线索。

## 审批与沙箱：源码中的顺序和分支

[`ToolOrchestrator::run`](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs#L125-L255) 读取审批策略、工作区、权限配置和运行环境，先计算审批要求。源码中的分支包括跳过、禁止、需要审批和需要人工审批；随后代码进入所选沙箱下的首次尝试。

这个顺序解释了两个不同的控制点：审批判断某个动作是否可以继续；沙箱决定执行时的资源边界。严格自动审查等分支会影响具体路径，因此不能简单断言“每次工具调用都要人工点击”或“配置允许就一定没有额外审查”。

## 一个可操作的排查例子

假设 Agent 提议执行一条构建命令，但请求被拒绝，可以按顺序检查：

1. **响应解析：** 模型输出的名称和参数是否构成可识别的 `ToolCall`？
2. **注册与路由：** 目标工具是否存在，是否被当前配置暴露？
3. **审批判断：** 拒绝来自明确禁止、等待审批，还是审查过程出错？
4. **运行环境：** 使用什么工作目录和权限配置，沙箱能否访问所需资源？
5. **实际结果：** 进程有没有启动、退出码和输出是什么？

这是一套定位方法，不代表每种失败都会走过同一条路径。日志不包含敏感参数时，应使用脱敏信息或安全的复现样本。

## 后续值得读的模块

本次分析聚焦路由、注册和执行编排，没有完整追踪所有工具处理器、会话生命周期、远程环境和模型协议。要研究某次真实故障，下一步应固定当时版本，进入具体 handler 与配置来源，而不是根据一张总图猜测。
