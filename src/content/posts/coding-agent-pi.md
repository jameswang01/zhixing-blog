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


## 源码精读：循环什么时候继续，什么时候退出

先看 [`runLoop`](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L172-L325)。它维护 `currentContext`、待处理的 `pendingMessages` 和上一轮结果。外层循环在正常结束之前检查是否还有 follow-up；内层则处理模型输出、工具调用和 steering。两层循环并不表示“一次用户请求一定对应两次模型调用”：是否继续取决于工具调用、队列以及结束回调的决定。

按代码执行顺序，可以拆成六步：

1. **整理待处理输入。** 已准备的消息和排队中的 steering 被加入当前上下文，并发出消息生命周期事件。
2. **准备请求。** 可选的 `prepareRequest` 能返回更新后的上下文、模型或推理设置，因此不能只看初始配置就断言下一次请求用什么。
3. **调用模型。** `streamAssistantResponse` 处理模型输出，得到最终 assistant message。
4. **处理错误与取消。** 当结束原因是 `error` 或 `aborted` 时，代码调用结束处理并发出结束事件，然后返回。
5. **执行工具。** 代码从 assistant message 内容里筛选 `toolCall`；执行后的结果加入上下文，供后续请求使用。
6. **决定是否再次推进。** 工具结果、steering、follow-up 或显式继续决定，分别可能让循环继续；都没有时才结束。

把这个过程和业务系统对照：用户点击停止，不一定等于下游业务动作撤销。循环接收到取消信号，与外部服务有没有响应取消，是两个需要分别验证的问题。

## 源码精读：流式消息如何进入上下文

[`streamAssistantResponse`](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L390-L460) 先执行可选的 `transformContext`，再调用 `convertToLlm`。流开始时它将部分 assistant message 加入上下文，收到文本、思考内容或工具调用的增量事件时更新该消息；完成或出错后，再以最终消息替换部分结果。它同时向宿主发出事件，供界面等消费者使用。

这里有两个容易混淆的点。第一，**界面上的增量展示不是多个互相独立的最终消息**；代码是在更新当前部分消息。第二，提供上下文变换扩展点，并不等于框架对每一种压缩策略都提供质量保证：如何保留目标、关键编号和权限约束，需要宿主设计并测试。

## 源码精读：为什么工具批次可能退回串行

[`executeToolCalls`](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L518-L580) 会先判断：配置是否要求串行，或者当前批次是否有工具声明 `executionMode` 为 `sequential`。满足任一条件就进入顺序路径，否则进入并行路径。

举个例子：模型一次要求“读取配置”和“修改同一配置”。两项工作是否允许并行，不能只看它们是否是不同工具名；如果修改动作依赖读取结果或共享文件，应该把依赖纳入执行策略。代码中的模式判断提供了控制入口，但工具作者怎样声明、宿主怎样配置，仍需逐项检查。

当模型响应因长度限制截断时，[循环会为该批工具调用生成失败结果](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L268-L280)，而不是直接执行可能不完整的参数。这是一个值得复用的边界：**JSON 勉强可解析，不代表模型原本想表达的参数已经完整。**

## 带着问题再读一次

- `getSteeringMessages` 与 `getFollowUpMessages` 分别在哪个边界被读取？消息到达时机不同会影响什么？
- `prepareNextTurn` 在压缩等耗时操作期间，如何避免重复取走 steering？
- 工具产生外部副作用后进程退出，哪些信息已经保存？这一点不能只看循环文件，需要继续查具体工具与会话存储。
- 工具超时是从未执行、正在执行还是已经成功但响应丢失？要结合工具和下游服务验证。

**验证范围：** 以上是固定提交的静态代码分析；没有在所有 provider、扩展与宿主配置下运行实验。


## 用测试交叉验证：三种不容易从概念看出来的行为

### 案例一：参数看似合法，为什么仍不执行

[测试用例](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/test/agent-loop.test.ts#L509-L579) 构造了一条因长度限制而结束的模型消息。消息里仍有一个参数符合类型要求的工具调用，但测试断言工具的 `execute` 没有被执行、结果被标记为错误，随后循环继续，让模型有机会重新发出调用。

对照[实现](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L268-L291)，这里的判断依据是 assistant 消息的结束原因，而不是只看参数能否通过类型校验。假设工具参数是一个字符串，截断后的前半段仍可能是合法字符串；如果这段字符串代表路径、订单号或查询条件，仅靠 schema 不能判断语义是否完整。

**能得出的结论：** 在该测试构造的 `length` 停止场景中，工具不会按该消息执行。**不能直接推出：** 所有 provider 的截断都一定以同一种方式报告，或所有语义错误都能由此发现。

### 案例二：并行完成顺序和结果顺序并不相同

[测试](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/test/agent-loop.test.ts#L721-L813) 故意让第一个调用等待，让第二个先完成。它分别检查两种序列：工具执行结束事件按实际完成顺序为第二个、再第一个；写入消息结果以及 `turn_end` 里的结果则按原始工具调用顺序排列。

这在排障中很重要。假如你只按日志时间看，会以为第二个工具“先被模型要求”；但完成顺序只能说明谁先执行完，不能单独证明模型提出调用的顺序。做追踪时应同时保留调用 ID、原始批次顺序和时间，而不是依赖单一排序。

### 案例三：校验之后的 hook 仍可能改变参数

[`prepareToolCall`](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/src/agent-loop.ts#L708-L770) 先准备并验证参数，再将验证后的参数传入 `beforeToolCall`，最终以该对象继续执行。[对应测试](https://github.com/badlogic/pi-mono/blob/4f95fbe2f117d288610e937889ad03575777841c/packages/agent/test/agent-loop.test.ts#L581-L640) 在 hook 中把原本通过字符串校验的值改成数字，断言工具实际接收到了修改后的值。

这里体现的是**该测试覆盖的可变参数行为**：如果扩展拥有修改参数的能力，前一次校验不一定代表执行时的值仍符合 schema。它不等于证明某个部署存在可利用漏洞，也不说明 hook 来自不可信方；是否增加不可变对象或二次校验，应结合扩展信任模型和兼容性评估。

## 修改这段代码，哪些行为可能一起变化

| 想改哪里 | 不能只看什么 | 需要回归的行为 |
| --- | --- | --- |
| 调整批次并行 | 平均耗时 | 单个串行工具是否使批次串行、结果顺序、取消时未完成调用如何收尾 |
| 调整截断处理 | JSON 是否能解析 | `length` 结束时是否执行工具、错误是否进入后续上下文 |
| 调整 hook 参数 | hook 是否能运行 | 校验先后、对象是否可变、阻断结果以及已有扩展的兼容性 |
| 调整流式事件 | UI 看起来是否正常 | 部分消息与最终消息是否一致、事件消费者是否收到完整生命周期 |

这些回归项依据当前实现和仓库测试归纳；本文阅读了测试代码，**没有在本次环境中执行这些测试**。
