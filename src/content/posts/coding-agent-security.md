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
