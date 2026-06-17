# Runtime-Centered Agent Architecture：从 LLM 中心到 Runtime 中心

日期：2026-06-17

## 1. 核心结论

今天的讨论敲定了一个非常重要的范式转移：

> 不应该把 harness 看成以 LLM 为中心的外围补丁，而应该把 LLM 看成 runtime 中可调用的一类智能工具。

传统视角是：

```text
LLM 是中心
Harness 是外围辅助
Tools / Memory / State / Eval 都是给 LLM 打补丁
```

更正确的视角应该是：

```text
Runtime 是中心
LLM 是 runtime 调用的一类认知工具
Tools / Memory / State / Eval / Permission / Router 都是 runtime 的组成部分
```

这个视角一旦切换，很多问题的优先级都会重新排序。我们不再只是问“如何写更好的 prompt”或“如何让模型更聪明”，而是开始问：

- 任务状态如何表示？
- 上下文如何编译？
- 哪些信息进入模型，哪些留在外部状态？
- LLM 输出如何被验证？
- 工具调用如何被权限控制？
- 失败如何回滚？
- 长任务如何 checkpoint？
- 多个 LLM 调用如何协作？
- 不同模型如何被 router 调度？
- 经验如何沉淀为可复用规则？

这意味着 agent 工程的中心，不是模型本体，而是围绕模型构建的运行时系统。

## 2. LLM 的本体缺陷决定了 harness 的长期价值

LLM 本身更像一个强大的无状态认知函数：

```text
LLM = f(context, weights) -> next tokens
```

它可以理解、推理、生成、规划，但它本体上并不天然具备可靠的长期记忆、状态管理、权限控制、事务管理、执行验证和恢复机制。

真正的 agent 更接近：

```text
Agent = Runtime + LLM + Memory + State + Tools + Planning Loop + Validation + Permissions + Feedback
```

LLM 单独并不知道：

- 上一步真实执行是否成功；
- 当前任务已经做到哪一步；
- 哪些约束不能忘；
- 哪些文件已经改过；
- 哪些命令失败过；
- 哪些结论已经被验证；
- 哪些工具不能调用；
- 用户长期偏好是什么；
- 本次任务的最终完成标准是什么。

这些都不是模型本体天然可靠拥有的能力，而是 runtime / harness 必须补上的能力。

因此，今天敲定的第二个核心判断是：

> 只要 LLM 本体仍然是无状态的，harness/runtime 就不会过时。它会像操作系统一样，成为智能系统的基础层。

## 3. Harness 类似操作系统，而 LLM 类似 CPU

一个非常有力的类比是：

```text
CPU 负责计算
OS 负责进程、内存、文件、权限、调度、I/O、异常恢复

LLM 负责认知计算
Harness/Runtime 负责状态、记忆、工具、权限、验证、调度、恢复
```

CPU 再强，也不能替代操作系统。CPU 负责算力，但真正让计算机成为可用系统的是操作系统。

同样，LLM 再强，也不能天然替代 agent runtime。LLM 提供认知能力，但真正让 AI 变成可持续、可验证、可恢复、可审计的生产系统的是 runtime。

可以进一步类比：

```text
LLM ≈ CPU / 认知核心
Harness ≈ OS / Runtime / 控制系统
Tools ≈ 外设与系统调用
Memory ≈ 文件系统与数据库
Eval/Test ≈ 监控与单元测试
Permission ≈ 权限模型
Compaction ≈ 内存分页 / 垃圾回收
Router ≈ 调度器
```

这个类比帮助我们避免一个常见误判：harness 不是模型的临时补丁，而是智能系统的基础结构。

## 4. Runtime-centered 视角改变了 agent 的定义

传统说法常常是：

```text
Agent = LLM + Tools
```

但这个定义太浅。更准确的定义应该是：

```text
Agent = Runtime + 可插拔认知模型 + 状态化执行闭环
```

在这个定义中，LLM 只是 runtime 调用的一类智能算子。runtime 才负责：

- 持久化任务状态；
- 管理上下文窗口；
- 决定调用哪个模型；
- 控制工具权限；
- 执行测试与验证；
- 处理失败与恢复；
- 管理多线程、多 agent、多 worktree；
- 记录日志和成本；
- 沉淀经验和规则；
- 判断任务是否真正完成。

因此，一个核心原则是：

> 能状态化的，不 prompt 化。能外部化的，不塞进模型脑子里。能验证的，不靠模型自信。

进一步展开：

```text
不要让模型“记住状态”，而是让 runtime 存状态。
不要让模型“保证验证”，而是让 runtime 执行验证。
不要让模型“自觉控制权限”，而是让 runtime enforce 权限。
不要让模型“自己知道任务进度”，而是让 runtime 提供当前任务快照。
不要让模型“凭感觉完成”，而是让 runtime 检查完成条件。
```

这就是从 prompt engineering 进入 runtime engineering 的关键。

## 5. 为什么模型变强也不会消灭 runtime

模型能力每次跃迁，都会吞掉一部分低级 harness 复杂度。例如，以前需要复杂 prompt 才能做的事情，强模型可能直接做好；以前需要多 agent 协作的任务，未来可能一个强模型就能完成。

但这并不意味着 harness 会过时。

弱模型时代，harness 主要补：

```text
不会规划
容易忘
工具调用差
上下文短
容易跑偏
```

强模型时代，harness 主要管：

```text
状态持久化
权限边界
成本控制
可审计性
可回放性
任务调度
验证闭环
外部记忆
多 agent 协作
生产系统集成
```

也就是说，模型变强会吃掉一部分低级 prompt engineering，但不会吃掉高级 runtime engineering。

原因很简单：真实生产系统不可能把状态、权限、验证、审计、恢复全部交给模型“凭感觉”维护。这些必须显式存在于系统结构中。

## 6. OpenClaw 的启发

OpenClaw 很可能已经在架构层面完成了类似的范式转移。

它不是简单地把 LLM 放在中心，然后在外围加工具，而是更接近构建一个 personal agent runtime：

```text
Gateway / Runtime 维护 session、routing、channel、workspace、policy、memory、sandbox
LLM / CLI agent / native harness 是 runtime 调用的执行器
```

这说明 runtime-centered agent architecture 不是空想，而是已经在真实 agent 系统中出现。

但 OpenClaw 的重点更像是 personal assistant runtime，而不是大型代码迁移 runtime。

因此，对我们的启发不是“再做一个 OpenClaw”，而是：

> 借鉴 OpenClaw 的 runtime-centered 思路，构建一个垂直领域的 code migration runtime。

## 7. 对 Codex Rust → Python 迁移项目的意义

这个范式转移对当前 Codex Rust → Python 迁移项目尤其重要。

错误设计是：

```text
把整个迁移目标丢给 Codex
让 Codex 自己读代码、自己规划、自己修改、自己验证
```

这仍然是 LLM-centered 视角。

更好的设计是：

```text
Runtime 维护迁移总状态
Runtime 解析 crate 依赖
Runtime 创建 worktree
Runtime 生成任务包
Runtime 调用 LLM 完成局部迁移
Runtime 运行测试
Runtime 记录失败
Runtime 更新状态
Runtime 决定是否 merge
Runtime 沉淀经验
```

在这个系统中，LLM 只负责高认知密度动作：

```text
读懂 Rust 模块
解释 API 语义
生成 Python 等价实现
分析测试失败原因
提出修复方案
总结迁移经验
```

但以下问题不能交给 LLM 的临时上下文，而应该由 runtime 的显式状态管理：

```text
哪个 crate 已完成
哪个 crate 正在迁移
哪个 worktree 对应哪个任务
哪些测试已通过
哪些失败是阻塞性问题
哪些经验可以复用
哪些文件不能动
是否符合“尽量不用第三方依赖”的约束
是否可以 merge
```

这会把迁移项目从“使用 AI 写代码”升级为“构建代码迁移工厂”。

## 8. Code Migration Runtime 的雏形

围绕代码迁移，可以设计一个专用 runtime：

```text
Code Migration Runtime
├── 任务状态表
├── crate / module 依赖图
├── worktree 管理器
├── 任务包生成器
├── 上下文编译器
├── LLM 调用接口
├── 测试执行器
├── 失败归因库
├── checkpoint / rollback
├── merge 策略
├── 经验沉淀系统
├── 成本统计
└── 回归评测
```

这个 runtime 的价值不是“包装一个模型”，而是掌握某类任务的 runtime contract：

```text
大型代码迁移任务如何被表达？
如何拆分？
如何分配给模型？
如何限制修改范围？
如何测试？
如何判断完成？
如何失败恢复？
如何沉淀经验？
如何持续降低 token 成本？
```

这正是可能形成壁垒的地方。

## 9. 最重要的工作假设

今天的讨论可以固化为以下工作假设：

```text
LLM is not the center.
Runtime is the center.

LLM is a cognitive tool.
Runtime is the system.

Prompt is not state.
Context is not memory.
Self-reflection is not validation.
Tool calling is not workflow.
Agent loop is not production system.
```

中文表达为：

```text
LLM 不是中心，Runtime 才是中心。
LLM 是认知工具，Runtime 才是系统。
Prompt 不是状态。
Context 不是记忆。
自我反思不是验证。
工具调用不是工作流。
Agent loop 不是生产系统。
```

## 10. 结论

今天的讨论至少敲定了两件事：

第一，不应该再用“LLM 中心 + 外围 harness”的视角理解 agent。

第二，只要 LLM 本体仍然无状态，harness/runtime 就会像操作系统一样越来越重要，而不会过时。

最终可以总结为一句话：

> Runtime 的本质，是把不稳定的智能转化为稳定的生产力。

这意味着，真正值得长期研究的不是狭义 prompt engineering，而是：

> Agent Runtime Engineering。

对个人开发者来说，机会不在于训练一个更大的通用模型，而在于围绕高价值场景，设计一个能够持续调用模型、管理模型、验证模型、放大模型的专用 runtime。

对当前路线来说，最自然的落点就是：

> Code Migration Runtime / 代码迁移运行时。

这不仅可以成为一个软件项目，也可以成为内容、课程、产品和个人方法论的统一主线。
