---
title: "接入与可观测性：怎样追踪一次 Agent 任务"
description: "区分接入方式、实时事件和可回放的记录。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 先问使用者需要什么

终端界面适合交互调试，SDK 适合嵌入应用，服务接口便于远程连接；它们都是接入方式，并不自动定义相同的权限、持久化或故障恢复能力。

## 从源码找到可观察的边界

Pi 的循环在模型与工具推进过程中产生事件，宿主可以基于这些事件更新界面或记录过程；具体事件字段以对应代码版本为准。[Pi 循环](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts)

DeepSeek 文档区分可持久化的 session 事件与运行中的事件，二者服务于不同目的。实时界面看到的信息不应直接等同于已经落盘、可恢复的记录。[DeepSeek 事件说明](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)

Codex 的工具路由与执行编排为追查工具调用提供代码入口，但是否记录完整参数、怎样脱敏以及保存多久，应以具体产品和环境为准。[Codex 路由](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs) · [执行编排](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs)

Claude Code 的使用行为和扩展入口可以从官方文档了解；内部遥测实现未在本次资料中验证。[官方工作方式](https://code.claude.com/docs/en/how-claude-code-works)

## 设计一条可排障记录

至少关联任务、轮次、工具调用与尝试次数，并保存必要的状态、耗时、错误类别和版本信息。对用户输入、工具参数和返回值应先确定是否需要记录，再考虑脱敏、权限及保留时间。

例如一个结果错误的任务，可以依次核对用户输入、解析出的参数、工具返回的数据范围、最终回答引用。HTTP 请求成功只是其中一个信号，不足以证明结果正确。

## 验证建议

在安全的测试环境注入超时、空结果和取消，检查事件顺序是否清楚、界面显示与后端状态是否一致。本文没有运行这些产品的统一观测实验。
