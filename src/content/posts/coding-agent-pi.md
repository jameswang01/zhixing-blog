---
title: "Pi Agent：从一次请求读懂模型与工具循环"
description: "Pi 核心循环、工具批次与会话边界，基于固定源码版本。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 先看结论

Pi 把 Agent 的一轮执行拆成模型调用、工具调用、结果回填和再次调用。阅读时先盯住循环边界与消息，而不是从界面猜测运行机制。本文核对的版本是 `4f95fbe`；这里讨论的是仓库中可见实现，不意味着所有宿主配置都相同。[核心循环](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts)

## 从用户输入到下一次模型调用

入口函数 `agentLoop` 和 `agentLoopContinue` 对应新输入与继续执行；`runLoop` 管理推进过程。发送请求前，代码提供 `transformContext` 和 `convertToLlm` 两个步骤：前者可调整工作消息，后者将内部消息转换为模型接口接受的格式。两者解决的是不同的问题：整理上下文，与适配模型协议。[查看实现](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts)

可以用一条线理解：用户消息 → 工作上下文 → 转换为模型消息 → 流式接收回复 → 如有工具调用则执行并把结果加入上下文 → 再次请求模型。没有工具调用时也不代表整个任务一定成功，业务验收仍需要另外定义。

## 工具并行不等于无约束并发

`executeToolCalls` 检查同批调用是否包含需要顺序执行的工具，再进入顺序或并行路径。源码还对因输出长度限制而截断的工具调用返回错误，避免把可能不完整的参数当作完整调用直接执行。[查看工具调度](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts)

这带来的工程问题是：即使模型一次提出多个动作，执行器仍必须了解动作之间的顺序和副作用。是否允许并行应由工具契约与实际资源共同决定。

## 中途输入与会话

循环分别处理 steering 与 follow-up：前者可在执行过程中介入，后者在当前阶段结束后作为后续输入。会话管理另有独立模块；保存对话、构造本轮上下文以及恢复任务是相关但不同的问题。[Agent 循环](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts) · [会话管理](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/coding-agent/src/core/session-manager.ts)

## 可以继续验证什么

源码展示了扩展点和执行分支，但本文没有运行不同 provider 的集成测试，也未验证所有插件组合。阅读时可以从一次实际 trace 出发，记录输入消息、工具名、参数、结果及后续模型请求，对照代码确认控制流。
