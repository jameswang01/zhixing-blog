---
title: "上下文、会话与压缩：长任务要保存什么"
description: "区分模型可见消息、会话记录和外部业务状态。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 三种经常混淆的东西

| 对象 | 主要用途 | 常见误区 |
| --- | --- | --- |
| 当前上下文 | 本轮模型实际接收的信息 | 以为等于完整历史 |
| 会话或事件记录 | 保存交互与执行事实，供查看或恢复 | 以为记录完整就能精确重放 |
| 业务状态 | 保存订单、权限、任务进度等权威事实 | 让模型摘要代替权威数据 |

## 在源码中看边界

Pi 在模型调用前提供上下文变换和消息转换；会话管理位于单独模块。阅读时应核对转换前后哪些信息仍可见。[循环](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts) · [会话](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/coding-agent/src/core/session-manager.ts)

DeepSeek 文档描述从会话事件投影模型历史，并区分持久事件与实时事件。外部写入是否完成仍需通过业务系统确认。[架构说明](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md) · [会话实现](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/session/src/index.ts)

Codex 有独立的压缩实现，可以从压缩相关模块继续追踪输入和产物；不能据此推断任意版本的具体压缩阈值。[压缩代码](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/compact.rs)

## 一个恢复场景

假设任务查询了三次，第四步提交申请时超时。恢复时需要保留目标、已完成步骤、业务请求标识和待确认状态；先查下游是否已经提交，再根据下游提供的幂等机制决定是否重试。只保存聊天记录无法证明申请没有成功。

## 评价压缩是否有效

选同一组任务，对比压缩前后能否保留用户约束、关键编号、工具结果来源和下一步行动；同时观察成本与延迟。需要注意模型与工具版本变化，否则对比结果难以归因。这里的评价方法是建议，并非声称仓库已内置完整评测。
