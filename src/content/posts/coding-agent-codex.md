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
