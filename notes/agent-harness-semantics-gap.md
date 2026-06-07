# Agent Harness 的语义—结构化鸿沟

> 讨论日期：2026-06-07  
> 主题：OpenClaw、harness engineering、复杂长程任务、自然语言任务与计算机结构化状态之间的表示鸿沟

## 1. 核心结论

这次讨论围绕一个核心问题展开：

> 大模型擅长理解自然语言意图，但复杂任务的可靠执行依赖计算机可管理、可验证、可恢复的结构化状态。自然语言与结构化状态之间并不天然等价，这构成了当前 Agent 系统从 demo 走向工程可靠性的根本断裂点之一。

更简洁地说：

> 自然语言适合表达意图，结构化状态适合承载执行。Agent harness 的核心任务，就是在这两者之间建立一个不断编译、约束、验证和校正的中间层。

这不是一个凭空捏造的问题，而是当前 Agent OS / harness engineering 的核心矛盾。

## 2. 创新的边界：围绕大模型固有属性做系统设计

Cursor、OpenClaw、OpenHands、Claude Code、Codex 等 AI coding / agent 产品之所以能体现出实际价值，不是因为它们假设大模型已经具备完美自治能力，而是因为它们围绕大模型的固有能力与缺陷做了系统设计。

大模型擅长：

- 语义理解；
- 代码阅读；
- 局部推理；
- 模式迁移；
- 自然语言到操作计划的转换；
- 根据工具反馈进行短闭环修正。

大模型不稳定或不擅长：

- 长期状态管理；
- 复杂项目级目标保持；
- 严格完成判定；
- 精确权限边界控制；
- 完整记忆；
- 长程任务中的自我纠偏；
- 对开放目标的可靠验收。

因此，真正能落地的 Agent 创新不是幻想模型拥有它没有的能力，而是：

> 把模型放进一个有状态、有边界、有工具、有验证、有恢复能力的系统里，让模型在最强的位置发挥，让系统在模型最弱的位置兜底。

## 3. 当前顶级大模型是否具备复杂任务规划能力

讨论中的判断是：

> 顶级大模型已经具备局部复杂任务的推理与短中程规划能力，但还不具备稳定、长期、跨环境、低监督的复杂任务自治能力。

它们可以：

- 把任务拆成步骤；
- 阅读代码库局部上下文；
- 根据错误日志修正方案；
- 生成 patch；
- 调用工具；
- 在有限闭环内迭代。

但它们仍然难以可靠做到：

- 长期保持目标不漂移；
- 跨几十或上百步维护完整状态；
- 判断何时真正完成；
- 在无明确反馈信号时持续优化；
- 保证没有遗漏隐含语义；
- 自动区分“看似完成”和“语义完成”。

所以，复杂任务里的核心不是让 LLM 单独承担自治，而是通过 harness 把大任务拆成一系列可观察、可验证、可恢复的小闭环。

## 4. OpenClaw / harness 的角色：任务操作系统，而不是聪明 prompt

讨论中形成了一个关键表述：

> 不要把 OpenClaw harness 当成“聪明 prompt”，而要把它当成“任务操作系统”。

### 4.1 聪明 prompt 的问题

如果把 harness 当成聪明 prompt，本质上是在期待模型自己完成以下工作：

- 理解目标；
- 拆解任务；
- 记住状态；
- 控制范围；
- 执行步骤；
- 检查质量；
- 判断何时停止。

这相当于把复杂任务的可靠性全部压在模型的自然语言推理上。短任务可以，但长程任务很容易出现漂移、遗漏、重复、过度自信和状态混乱。

### 4.2 任务操作系统的含义

如果把 harness 当成任务操作系统，它应该承担类似操作系统的职责：

- 调度：决定哪个 agent、工具或流程先运行；
- 状态管理：记录任务进展、当前阶段、历史决策和未完成事项；
- 权限管理：限制模型能改什么、不能改什么；
- 上下文管理：给模型提供当前步骤需要的上下文，而不是全部塞入窗口；
- 资源管理：管理 workspace、文件、session、builder agent、review agent；
- 错误处理：失败后重试、降级、回滚、重新规划；
- 验收机制：用测试、lint、review、score、人工确认决定是否通过；
- 日志系统：记录过程，方便审计、恢复和追责。

这一层的核心不是让模型“一次性想对”，而是让模型在可恢复、可验证、可迭代的系统中持续前进。

## 5. 长程复杂任务：适合 OpenClaw harness，但不能全自动托付

对于大型开源项目架构迁移、跨语言转写等长程任务，OpenClaw / harness 的设计方向是相关的，但不能理解为“全自动魔法迁移器”。

### 5.1 大型架构迁移

这类任务适合 harness 化，因为可以拆成连续 sprint：

1. 全局扫描；
2. 识别依赖面；
3. 生成迁移设计文档；
4. 选择低风险模块做 pilot；
5. 跑测试；
6. 总结失败模式；
7. 扩大迁移范围；
8. 持续 review；
9. 清理旧路径；
10. 人工验收关键 milestone。

在这种场景中，harness 的价值不是一次性完成迁移，而是把迁移变成一系列可验证的小迁移。

### 5.2 大型项目跨语言转写

跨语言转写更难，因为它涉及：

- 语言语义差异；
- 类型系统差异；
- 运行时差异；
- 异步模型差异；
- 标准库和生态替换；
- 错误处理范式变化；
- 构建系统迁移；
- 测试体系迁移；
- API 行为一致性验证。

合理做法不是“整库翻译”，而是：

1. 先建立行为画像；
2. 提取公共接口；
3. 建立 golden tests / parity tests；
4. 选择边界清晰的模块；
5. 生成目标语言实现；
6. 用 fixture、快照、协议测试比较行为；
7. 旧实现和新实现并行；
8. 分阶段切换入口。

因此，harness 可以成为迁移工厂的控制平面，但不能替代人类架构师和严格的行为等价验证。

## 6. 语义意图与结构化状态之间的表示鸿沟

这次讨论最重要的问题是：

> 系统侧没有自然语言理解能力，因此无法直接判定一个自然语言任务是否已经完成；系统只能判断结构化条件是否满足。

例如，人类说：

> 把这个项目迁移到新架构，并保持行为一致。

这里包含大量隐含语义：

- 什么叫新架构；
- 哪些行为必须一致；
- 哪些行为可以改变；
- 哪些历史 bug 应该保留；
- 哪些 undocumented behavior 已经被用户依赖；
- 哪些性能退化不可接受；
- 什么程度才算迁移完成。

但计算机能判断的是：

- 文件是否被修改；
- 测试是否通过；
- typecheck 是否通过；
- lint 是否通过；
- diff 是否越界；
- API snapshot 是否一致；
- coverage 是否下降；
- benchmark 是否退化；
- 某个字段是否相等；
- 某条结构化断言是否成立。

两者不是同一种东西。

所以，系统侧不能判断 `semantic done`，只能判断 `verified done`。

### 6.1 semantic done

人类意义上的完成：

> 这件事是否真的达到了原始意图。

例如：

- 架构迁移是否合理；
- 行为是否真的一致；
- 长期维护性是否变好；
- 设计是否符合项目方向；
- 是否遗漏了隐含场景。

### 6.2 verified done

系统意义上的完成：

> 所有被形式化出来的验收条件都通过。

例如：

- 测试通过；
- 类型检查通过；
- 快照一致；
- 没有 forbidden diff；
- 覆盖率达标；
- 性能退化低于阈值。

关键结论：

> 结构化验收标准不是自然语言目标的等价物，而是自然语言目标的一个近似可验证投影。

## 7. 自然语言子任务不是可靠执行单元

如果一个大任务被拆成自然语言 todo，例如：

- 优化认证模块；
- 重构数据加载逻辑；
- 提升测试覆盖率；
- 迁移旧 API 到新架构。

这些对子任务的人类理解是自然的，但对系统来说是不可靠的，因为边界不清、完成标准不明、允许操作不明确。

模型可能把“优化认证模块”理解成：

- 修改错误码；
- 重构异常类；
- 调整日志格式；
- 改 API 行为；
- 增加依赖；
- 修改测试以适配错误实现。

因此：

> 只要子任务仍然主要以自然语言存在，它就不是可靠的执行单元，只是一个意图片段。

## 8. 解决方向：Task Contract，而不是 Task Description

更可靠的做法是将子任务变成结构化任务合约。

示例：

```yaml
id: auth-refactor-003
title: Refactor auth error handling
goal: Replace legacy auth error mapping with centralized error registry.
scope:
  include:
    - src/auth/**
    - tests/auth/**
  exclude:
    - src/api/public/**
    - package.json
allowed_operations:
  - edit
  - add_tests
forbidden_operations:
  - change_public_api
  - add_runtime_dependency
  - remove_existing_tests
inputs:
  - docs/auth-error-policy.md
  - src/auth/errors.ts
expected_outputs:
  - updated error handling implementation
  - new or updated unit tests
acceptance:
  commands:
    - npm test -- tests/auth
    - npm run typecheck
    - npm run lint
  invariants:
    - existing auth error codes remain unchanged
    - public API responses preserve schema
    - no changes to package.json
status:
  phase: planned
  owner: builder-agent-2
  attempts: 0
  blockers: []
```

这类 Task Contract 至少应该包含：

- 目标字段；
- 范围字段；
- 输入字段；
- 输出字段；
- 约束字段；
- 验收字段；
- 状态字段；
- 依赖字段；
- 回滚策略；
- 审计记录。

自然语言仍然存在，但不再是任务状态本身。

## 9. 任务 DAG 与状态机

复杂任务不只是多个子任务，还包括依赖关系。

自然语言式依赖：

> 先完成认证模块重构，然后迁移调用方。

这对系统不够明确。

结构化后应该是：

```yaml
tasks:
  - id: T1
    title: Centralize auth error definitions
    outputs:
      - src/auth/errors.ts
      - docs/auth-error-map.md
    acceptance:
      - npm test -- tests/auth/errors.test.ts

  - id: T2
    title: Update auth middleware to use centralized errors
    depends_on:
      - T1
    required_inputs:
      - T1.outputs.docs/auth-error-map.md
    acceptance:
      - npm test -- tests/auth/middleware.test.ts

  - id: T3
    title: Verify public API compatibility
    depends_on:
      - T1
      - T2
    acceptance:
      - npm test -- tests/api/auth-compat.test.ts
      - snapshot_compare auth_error_response
```

系统可以由此理解：

- T2 不能在 T1 未完成前开始；
- T3 依赖 T1/T2 的产物；
- 每个任务完成都有证据；
- 失败时可以定位影响范围；
- 重新规划时可以知道下游依赖。

这就是从自然语言计划转成任务 DAG。

## 10. OpenClaw 是否解决了这个问题

讨论中的判断是：

> OpenClaw 并没有从根本上解决自然语言与结构化状态不匹配的问题，但它试图把这个问题放到 harness 层去工程化缓解。

它的价值不在于让系统突然能理解自然语言完成态，而在于：

- 把 agent 从聊天框移到运行时；
- 通过 session / workspace 外部化状态；
- 通过 tools 把输出落到真实环境；
- 通过 multi-agent routing 支持角色分离；
- 通过 Plan → Build → Review → Iterate → Ship 流程让执行可循环；
- 通过 trace、review、测试、日志和权限边界提高可观测性与可治理性。

但它仍然不能自动回答：

> “这个自然语言目标是否已经被完整实现？”

这仍然需要：

- 结构化验收条件；
- 外部 verifier；
- LLM reviewer；
- 人类 milestone review。

## 11. 当前是否已经很好解决

讨论中的最终判断是：

> 这个问题目前还没有被很好解决。

已有的工程解法包括：

- 状态图 / 工作流：例如 LangGraph、CrewAI Flows；
- 多 agent runtime：例如 AutoGen、OpenHarness、OpenClaw harness；
- 软件工程执行环境：例如 OpenHands、SWE-agent、Aider；
- 过程观测与评测：例如 ClawBench、trace / telemetry / benchmark 工具；
- schema 化任务合约：例如 Pydantic / JSON Schema / YAML task specs；
- sandbox 与验证：例如 Docker、pytest、lint、typecheck、snapshot、contract tests。

但这些只是局部工程解法，还没有形成一个成熟、通用、可靠的方案，可以把任意自然语言复杂任务稳定转换成计算机可验证的结构化状态与完成判定。

原因是：

> 系统只能验证被形式化出来的断言，不能验证没有被形式化出来的语义。

## 12. 更现实的混合架构

较可靠的复杂 Agent 架构应该是：

```text
自然语言目标
  ↓
LLM 辅助生成结构化 Task Spec
  ↓
Schema Validator
  ↓
人类 / Lead Agent 审核任务合约
  ↓
Task DAG / State Machine
  ↓
Executor Agent 执行局部任务
  ↓
Sandbox / Tool / Test 验证
  ↓
LLM Reviewer 检查语义风险
  ↓
State Store 记录证据与进度
  ↓
失败则 replan / retry / rollback
  ↓
关键 milestone 由人类验收
```

在这个架构中：

- LLM 负责理解、生成候选计划和局部执行；
- 系统负责状态、权限、验证、审计和恢复；
- LLM reviewer 负责发现语义风险；
- 人类负责关键语义完成标准和最终 milestone 判断。

## 13. 核心设计原则

这次讨论可以沉淀为几个原则。

### 13.1 不要让自然语言承载长期状态

自然语言适合表达意图，但不适合承载任务执行状态。长期状态应落到文件、数据库、task graph、issue、commit、trace、log 中。

### 13.2 不要把模型自述当完成证据

“我已经完成了”不是证据。证据应该来自 diff、测试、快照、日志、benchmark、review record、人工验收。

### 13.3 子任务应该是合约，而不是描述

子任务不应该只是 `description`，而应该包含 scope、constraints、inputs、outputs、acceptance、status、owner、dependencies。

### 13.4 系统验证断言，不验证意图

计算机不能直接验证“意图是否实现”，它只能验证“被形式化后的意图约束是否满足”。

### 13.5 复杂任务必须逐步收窄语义不确定性

从自然语言目标开始，不断下沉为设计文档、模块清单、任务合约、验收断言、工具结果和人工 milestone。

### 13.6 Harness 的壁垒在中间表示层

真正的壁垒不是写更长 prompt，而是设计自然语言意图到结构化状态之间的中间表示：Task Contract、Task DAG、State Machine、Verifier、Permission Policy、Progress Ledger。

## 14. 对 OpenClaw / Agent OS 的最终判断

OpenClaw / harness engineering 的方向是正确的：

> 它不是假设模型已经具备完美复杂规划能力，而是通过 session、workspace、tools、multi-agent routing、review/iterate、权限边界和过程审计，让模型的局部智能可以被组织成长程任务推进能力。

但它的边界也必须清楚：

> 它不能根本消除自然语言语义与计算机结构化状态之间的鸿沟，只能通过工程机制把这个鸿沟变小、变窄、变得可观测和可治理。

最终结论：

> 复杂长程 Agent 的真正瓶颈，不是模型会不会拆任务，而是自然语言任务能否被持续编译成足够完整、足够可验证、足够可追踪的结构化任务状态。

如果做不到，Agent 系统最终会退化成“会说很多计划，但执行状态不可控”。  
如果做到了，即使 LLM 仍然不完美，也可以通过系统设计获得局部可靠、渐进推进、可审计、可恢复的复杂任务执行能力。

## 15. 一句话总结

> Agent harness 的核心不是让 LLM 自治，而是让 LLM 在一个可控、可验证、可恢复的结构化系统中表现得像能自治。
