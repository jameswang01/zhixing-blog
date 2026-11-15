---
title: "权限与沙箱：Agent 执行风险如何分层"
description: "从源码和官方文档理解决策、隔离和审计的边界。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 两个问题先分清

权限回答“这次动作能否做”，沙箱回答“获准执行后能触达什么”。两者并不互相替代；此外还需要关注凭证、确认、日志和外部业务系统自身的授权。

## Pi：默认进程权限要看清

Pi 仓库 README 声明，默认不内置限制文件、进程、网络或凭证访问的权限系统，进程以启动它的用户和进程权限运行，并建议需要更强边界时使用外部隔离。这个结论针对所读版本和默认描述，扩展或宿主可以改变实际边界。[Pi README](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/README.md)

## Codex：沿具体工具继续追

Codex 公开代码包含工具执行编排、审批与沙箱相关模块。一次调用的实际权限取决于工具、策略、环境和配置；不能仅从 `ToolRouter` 的存在推断所有工具执行方式相同。[执行编排](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs) · [工具路由](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs)

## DeepSeek 与 Claude Code

DeepSeek 架构文档将工具和文件系统等能力放在可扩展的接口中；实际权限要看所加载的插件和配置，不能只凭框架结构断言隔离强度。[架构说明](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)

Claude Code 官方文档分别说明权限规则和沙箱配置。具体部署应核对平台支持与当前配置。[权限](https://code.claude.com/docs/en/permissions) · [沙箱](https://code.claude.com/docs/en/sandboxing)

## 面试中怎样展开

以“Agent 修改文件并执行测试”为例：先确定用户身份和允许的工作目录，再判断命令是否需要确认，接着限制进程可访问的文件、网络和凭证，最后记录动作与结果。对于发消息或修改线上数据等外部副作用，还应增加业务授权和确认，不能只依赖模型承诺。

本文不是安全审计；各产品具体的绕过条件、系统支持和安全保证需要结合版本进行测试。


## 设计一张动作风险表

| 动作 | 主要风险 | 可以考虑的控制 |
| --- | --- | --- |
| 读取代码 | 误读敏感文件或跨项目数据 | 限制可读路径、凭证范围，按需脱敏 |
| 修改工作区 | 覆盖无关内容或破坏构建 | 限定工作区、查看差异、运行检查 |
| 执行命令 | 进程访问文件、网络或环境变量 | 审批策略、系统隔离、最小凭证 |
| 调用外部写接口 | 产生不可逆或重复副作用 | 业务授权、确认、幂等及审计 |

表中的措施互相补充；是否生效取决于具体平台及部署配置。

## Pi：默认边界从哪里来

Pi 的 [README](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/README.md#L90-L96) 说明，默认以启动用户及进程的权限运行，不内置文件、进程、网络或凭证的限制系统，并列出容器化等选择。这意味着安全评估必须从启动用户、宿主环境、扩展代码和外部隔离一起看。

一个误区是“项目目录是当前工作目录，所以工具只能访问这个目录”。当前目录和系统级文件权限不是同义词；是否能访问其他路径必须在实际环境中确认。

## Codex：审批决定和运行隔离是不同判断

[`ToolOrchestrator::run`](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs#L125-L255) 读取策略和权限配置，计算审批要求，处理禁止、需要审批等情况，再进行所选沙箱下的首次执行。这个代码路径体现的是层次划分；实际动作是否经过这条路径、每个分支怎样配置，还要追具体工具。

当用户说“只读”时，至少要问清：是限制业务工具的写接口，还是限制进程文件写入？两者可能由不同机制实现。仅凭模型提示词不能完成这两类控制。

## Claude Code 与 DeepSeek 的证据边界

Claude Code 的[权限](https://code.claude.com/docs/en/permissions)和[沙箱](https://code.claude.com/docs/en/sandboxing)以官方文档为依据，本次没有审计闭源内部执行器。DeepSeek 的[架构文档](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)提供插件和能力接缝；要判断某次操作的边界，需要继续核对该部署实际加载的权限与工具插件。

## 失败时怎么验证

准备一个无敏感数据的隔离环境，分别尝试允许路径与拒绝路径、允许网络与不允许网络、确认后执行与拒绝后执行。记录产品版本、操作系统、配置及结果；如果某条行为无法稳定复现，就标记为待验证。测试不能证明所有绕过方式都不存在。
