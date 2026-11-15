---
title: "Agent Loop 与工具执行：从三份源码看控制流"
description: "比较模型调用、工具派发与循环退出的工程边界。"
date: "2026-11-15"
category: "Agent 与 AI"
tags: ["Coding Agent", "源码阅读", "工程实践"]
---

## 先确定比较范围

这里对照 Pi、Codex 和 DeepSeek Harness 的公开代码，Claude Code 只采用官方行为文档。它们的代码基线分别是 `4f95fbe`、`ee344ca`、`39163dd`；不同模块的名称与能力并非一一对应。

## 相同问题：模型提出动作之后怎么办

| 环节 | 需要回答的问题 |
| --- | --- |
| 模型请求 | 当前请求带哪些消息和工具定义？ |
| 结果解析 | 响应是普通文本、工具调用、错误还是被截断？ |
| 工具执行 | 参数是否可用，动作是否允许，并行是否安全？ |
| 结果回填 | 工具输出如何变成下一步可用的上下文？ |
| 停止与恢复 | 用户取消、预算耗尽、错误或完成如何表达？ |

## 三种可核对的代码入口

- Pi 的 `runLoop`、`executeToolCalls` 展现消息推进及工具批次分支，包含顺序和并行路径。[Pi 循环](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts)
- Codex 的 `ToolRouter` 与 `ToolRegistry` 将工具路由、注册和并行能力查询分开；具体循环还需继续沿调用链阅读。[路由](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs) · [注册](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/registry.rs)
- DeepSeek Harness 的 Agent 循环包含 turn、step 与工具集合解析等步骤，并与会话事件协作。[循环代码](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts) · [架构说明](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/docs/architecture.md)
- Claude Code 官方资料描述收集上下文、行动和验证的反复过程，但这不足以证明其内部函数调用顺序。[官方工作方式](https://code.claude.com/docs/en/how-claude-code-works)

## 如何用于面试

画一个最小执行图，再补三个失败路径：模型参数不完整、工具请求超时、用户中途取消。每条路径说明谁发现问题、状态记录在哪里、是否允许重试，以及用户看到什么。对外部写入，恢复前还要核实副作用是否已经发生。

## 未验证范围

这里没有做跨产品基准测试，也不据此比较成功率、速度或成本。实际行为还取决于模型、工具、版本与配置。


## 一次工具调用，具体有哪些状态

把“模型调用工具”压成一句话容易漏掉状态。以查询库存为例，至少可以区分：模型输出调用意图、程序识别调用、校验参数、权限判断、请求发出、下游返回、结果进入上下文、模型生成解释。中间任何一步失败，用户看到的都可能只是“没有回答”，但补救方式不同。

| 停在哪一步 | 有什么可观察证据 | 首先要核对什么 |
| --- | --- | --- |
| 解析之前 | 模型响应、结束原因 | 参数是不是被截断 |
| 路由阶段 | 工具名、命名空间、调用 ID | 工具是否注册并对当前请求可见 |
| 执行之前 | 校验结果、策略决定 | 参数合法不等于用户有权限 |
| 请求途中 | 关联 ID、超时信息 | 下游是否可能已经收到请求 |
| 返回之后 | 返回状态、数据范围 | 成功响应是否满足用户问题 |

这些是分析维度，不表示每个产品都暴露了表中全部字段。

## Pi：截断和批次是明确分支

Pi 的 [`runLoop`](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L268-L291) 在响应因长度限制截断时，不直接执行该消息中的工具调用；[`executeToolCalls`](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L518-L535) 则根据批次与配置选择顺序或并行执行。两者分别处理输入完整性和执行顺序，不能相互替代。

想象模型输出一个工具参数对象但后半截未生成。即使前半截看似包含了 `region`，系统也不应凭剩余字段猜测原始意图。相反，两个完全独立的只读查询可能适合并行，但是否安全仍需要工具或宿主声明约束。

## Codex：调用对象与执行环境是不同层

Codex 的 [`build_tool_call`](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/router.rs#L278-L349) 将响应转换为带工具名、ID 和 payload 的调用对象；[`ToolOrchestrator::run`](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/src/tools/orchestrator.rs#L125-L255) 则处理审批要求及沙箱下首次执行等步骤。这个拆分说明：能解析出调用并不代表实际动作一定被批准，更不代表已经成功。

## DeepSeek：step 前先准备当前能力

在 DeepSeek 的 [`preStep`](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts#L289-L322) 中，工具集合按当前 Agent 与模型选择解析，同时组装上下文；接着在 [`turn`](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/src/agent.ts#L332-L390) 内形成 step 边界。排查某一次调用时，必须同时看 session 记录、运行时配置和工具集合。

## 面试回答可以怎样组织

用一分钟说明正常链路，再挑一个故障向下展开。例如“写操作超时”：我会先查询下游状态；如服务支持幂等，使用同一业务标识按其契约重试；若结果仍未知，就保留待确认状态，而非简单告诉用户失败。这里的幂等和查询接口属于业务设计，并非上述所有项目已经替你实现。


## 三个测试案例怎样放在一起看

- **输入完整性：**[Pi 截断测试](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/test/agent-loop.test.ts#L509-L579) 构造被长度截断的响应，断言工具未执行，并允许循环继续。
- **并行观察：**[Codex 并发测试](https://github.com/openai/codex/blob/ee344ca9458351a6ea102795fa73110ad3ede5dc/codex-rs/core/tests/suite/tool_parallelism.rs#L117-L173) 利用两个开始标记和共同释放点，检查指定测试配置中命令确实重叠运行。
- **执行前失败：**[DeepSeek 工具顺序测试](https://github.com/deepseek-ai/deepseek-harness/blob/39163dd30f703a79079e2d2446545cff3c453738/packages/core/agent-loop/tests/tool-order.spec.ts#L102-L115) 在配置引用未知工具时，检查没有向模型适配器发送请求。

这三条分别验证不同问题，**不能用来给三个产品排性能或可靠性名次**。要做横向比较，需要统一模型、工具、工作负载、权限配置与指标。
