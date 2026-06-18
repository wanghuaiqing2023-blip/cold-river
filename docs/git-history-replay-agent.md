# Git History Replay Agent：基于 Git 历史的项目设计演化重放

> 本文档整理自一次围绕“如何利用 Git 历史理解大型开源项目设计决策，并最终驱动 coding agent 复刻核心演化路径”的讨论。

## 1. 核心目标

我们想构建的不是一个普通的 `git log` 可视化工具，也不是一个简单的代码总结器，而是一个 **Git History Replay Agent**。

它的理想效果是：

> 让用户像跟着作者从 0 到 1、再从 1 到 N 开发了一遍项目一样，理解项目的用途、需求节点、设计节点、架构概念、演化压力和后续稳定过程。

传统代码理解通常是：

```text
当前代码快照 → 总结项目是什么
```

我们想做的是：

```text
Git 历史 → 关键开发片段 → 设计动机推断 → 开发重放叙事 → 可执行复刻路线
```

最终不是只回答：

> 这个项目现在是什么？

而是回答：

> 它为什么一步步变成现在这样？

---

## 2. 为什么不能逐个 commit 分析

对于 OpenClaw 这类已经有 5 万多个 commit 的项目，把每个 commit 逐个发给 LLM 非常不现实。

问题不仅是 token 成本，更重要的是：设计决策通常不是单个 commit 能表达的。单个 commit 常常只是：

```text
fix typo
add test
update config
refactor gateway
bump version
```

这些 commit 是原子事实，但不是合适的理解单位。

更合理的处理方式是：

```text
commit → commit-cluster → history event → development episode → replay narrative
```

其中：

- **commit** 是原子证据；
- **commit-cluster / PR / CSM event** 是基本分析单位；
- **development episode** 是用户理解项目演化的一节课；
- **replay narrative** 是最终的“跟着作者开发一遍”的叙事路径。

---

## 3. 更严谨的抽象层次

最初我们曾提出：

```text
commit
→ commit-cluster
→ cluster role label
→ development episode
→ design rationale hypothesis
→ replay narrative
```

后来重新审视后发现，这条链条混合了数据对象、标签、推理结果和表达形式，存在冗余。

更干净的抽象应该是：

```text
Layer 1: Evidence Unit
commit / diff / PR / issue / test / doc

Layer 2: History Event
commit-cluster / CSM event / PR event

Layer 3: Development Episode
一组相关 history events，表达一个开发阶段或设计演化片段

Layer 4: Replay Narrative
把 episodes 串成“跟着作者开发一遍”的学习路线
```

一些原先被误认为“层”的概念，应当作为属性存在：

```text
cluster role label       = History Event 的属性
design relevance         = History Event 的属性
design rationale hypothesis = Development Episode 的解释结果
confidence               = Event / Episode 的置信度属性
```

这样系统结构更清晰，也更容易工程化。

---

## 4. commit-cluster 的角色分类

我们不仅要识别哪些 commit-cluster 是高价值设计事件，也要解释那些低价值、非核心 cluster 在项目演化中扮演什么角色。

低价值 cluster 不等于没有价值。它们可能是在：

```text
填充设计框架
迁移旧代码
稳定边界行为
修补设计缺陷
暴露设计摩擦
文档化概念
补充测试契约
做发布维护
```

因此，每个 cluster 应该同时具有两个维度：

```text
Design Relevance：设计相关度
Evolution Role：演化角色
```

建议的角色分类：

| 角色 | 含义 |
|---|---|
| 核心设计决策 | 引入新抽象、改变模块边界、改变控制流或数据流 |
| 设计落地 | 把新设计应用到已有模块或调用方 |
| 设计框架填充 | 在已有框架下增加新实现，例如新增一个 channel/provider/driver |
| 设计稳定化 | bugfix、测试补充、边界条件修复 |
| 设计摩擦 / 债务 | workaround、bypass、revert、反复修补同一边界 |
| 文档化 / 测试固化 | 让概念变成用户可见模型或行为契约 |
| 发布维护 | release、version bump、changelog、依赖更新 |
| 普通工程噪声 | format、typo、机械重命名等 |

例如对于 OpenClaw 的 `channel` 概念：

```text
核心设计决策：引入 Channel 抽象
设计落地：Gateway 迁移到 Channel
设计框架填充：新增 Telegram/Slack/Discord channel
设计稳定化：修复 group chat routing、session restore、permission 检查
设计摩擦：某些平台能力绕过 channel 或反复修补 session mapping
文档化：channel config、routing example、migration guide
```

---

## 5. 大型项目中的核心主干与外围填充

大型项目中有大量非核心演化。以 Linux kernel 为例，大量底层驱动代码并不代表 kernel 的核心设计演化。

所以不能把所有 cluster 平等放进 replay narrative。

我们需要区分：

```text
核心设计主干
子系统设计节点
外围生态填充
设计压力证据
普通维护噪声
```

可以引入 **core impact score**，判断一个 cluster 是否属于核心设计演化。

参考信号：

```text
是否修改核心目录
是否修改 public/internal framework API
是否影响多个子系统
是否导致大量调用方变化
是否引入新抽象
是否改变模块边界
是否改变测试契约
是否出现在文档概念模型中
后续是否被大量 cluster 依赖或填充
```

最终 replay 应该是：

```text
核心主干 spine：解释项目为什么这样设计
代表性肋骨 ribs：展示外围实现如何填充、验证、挑战核心设计
```

以 OpenClaw 为例：

```text
Spine:
gateway → channel → routing → session → plugin → multi-agent

Ribs:
WhatsApp channel
Telegram channel
Slack channel
某个权限 bugfix
某个 session 恢复问题
某个 plugin integration
```

---

## 6. Githru PDF 对我们的启发

讨论中分析了 Githru 论文：*Githru: Visual Analytics for Understanding Software Development History Through Git Metadata Analysis*。

Githru 的价值不只是可视化，而是提供了一套大规模 Git 历史压缩方法：

```text
原始 Git DAG
→ Stem 化
→ Context-Preserving Squash Merge, CSM
→ Commit Clustering
→ Stem Graph Blocks
→ Grouped Summary View
→ File Icicle Tree
→ Comparison View
```

其中几个概念特别关键：

### 6.1 DAG

Git 历史原本是 DAG：

```text
commit = node
parent-child = edge
branch / merge = topology
```

完整 DAG 在大型项目中不可读。

### 6.2 Stem

Githru 将复杂 DAG 转换为 stem structure：

```text
main stem
dev stem
fix stem
implicit stem
PR stem
```

它类似 `git log --first-parent`，但不是只作用于一条主分支，而是用于整体历史抽象。

### 6.3 CSM：Context-Preserving Squash Merge

CSM 把 merge / PR 背后的多个 commits 压缩到一个 CSM-base 上，同时保留上下文：

```text
authors
commit types
messages
PR number
PR title/body
changed files
```

对我们而言，CSM 的意义是：

> 把低层 commit group 压缩成一个高层 history event，同时保留足够证据供 LLM 推理。

### 6.4 Commit Clustering

Githru 使用以下 metadata 聚类：

```text
author
commit date
commit type
file
message
```

它说明：分析单位不应是 commit，而应是 commit-cluster。

### 6.5 Stem Graph Visualization

Githru 的图形构造过程大致是：

```text
1. 画二维 grid
   column = time slot
   row = stem

2. 每个 commit 按时间填入 cell

3. 相邻 commits 聚合成 block

4. block squeezed 到单列

5. block 画成 box
   height = block 中 commit 数量

6. cluster 外面画 outline

7. 不重叠的 stem relocation 到同一行

8. 每条 stem 底部加 thin strip
```

我们如果严格复刻 Githru 风格网页，应优先实现：

```text
Global Temporal Filter
Clustering Step slider
Stem Graph
Grouped Summary View
File Icicle Tree
Commit List
Comparison View
```

---

## 7. born open-source 项目更适合本方法

这套方法最适合从早期就开源、历史连续可见的项目。

如果一个项目是内部开发多年后才开源，公开仓库可能只有：

```text
Initial public release
一次性导入大量成熟代码
```

这时早期核心设计已经发生在不可见的私有历史中，Git 历史无法完整还原从 0 到 1 的过程。

因此需要先判断仓库的 **History Replayability Score**。

高可重放性特征：

```text
初始 commit 很小
早期能看到最小骨架
架构概念逐步出现
commit 粒度较自然
PR / issue / release 历史连续
测试随功能逐步增加
```

低可重放性特征：

```text
巨大 initial import
commit message 类似 sync from internal
一开始就是成熟系统
公开历史缺少真实设计讨论
作者/date 被重写
```

不同项目应进入不同模式：

```text
完整开发重放：born open-source / early-open 项目
公开历史重放：late-open 项目
当前架构考古 + 后续演化分析：snapshot-open / mirror 项目
```

---

## 8. 从设计理解到 coding agent 复刻

最理想的情况是：

> 根据关键需求节点和设计节点，利用 coding agent 沿着历史路线复刻出整个项目的核心演化过程。

这不是逐字复制原项目，而是：

```text
按照作者当年的需求压力和设计选择，
重新构建出行为等价、架构相似、演化路径一致的项目。
```

复刻是设计理解的最高级验证标准。

如果设计分析只是输出：

```text
这里可能是为了扩展性。
```

很难验证。

但如果它输出：

```text
这一阶段要引入 Channel interface、ChannelRegistry、Gateway 通过 Channel 收发消息，并通过 fake channel 测试。
```

coding agent 能够实现并测试通过，就说明 episode spec 抓住了真实设计结构。

---

## 9. Episode Spec：从历史事件到可执行任务

commit-cluster 不能直接喂给 coding agent。

它需要被编译成 **Development Episode Spec**。

示例：

```text
Episode：引入 channel 抽象

历史状态：
在这个节点之前，gateway 直接处理具体平台消息。

需求压力：
平台数量增加，消息格式、身份、权限、session、群聊/私聊差异开始污染 gateway。

设计目标：
引入统一 channel 边界，隔离平台差异。

需要实现：
- Channel interface
- FakeChannel
- ChannelRegistry
- Gateway 通过 Channel 接收消息
- Gateway 通过 Channel 发送回复
- 基础权限检查

验收测试：
- fake channel 可以注入消息
- gateway 能路由到 agent
- agent 回复能回到同一 channel
- 未授权 sender 被拒绝

参考证据：
相关 commits / PR / changed files / tests / docs

非目标：
暂不实现所有真实平台，只实现接口和 fake channel。
```

这才是 coding agent 可以执行的上下文包。

---

## 10. LLM 的无 state 问题与 context 构造

LLM 本身无长期 state。它总结项目时本质上是在当前上下文窗口里临时构造一个 working model。

没有 README 时，它依靠以下信号反推项目用途和架构：

```text
目录结构
入口文件
构建系统
配置文件
测试用例
依赖库
API route
CLI 参数
examples / demo
Dockerfile / CI
模块名 / 类名 / 函数名
错误信息 / 日志文本
Git 历史
```

这本质上是信息压缩：

```text
海量原始信息
→ 保留对理解项目最有解释力的信息
→ 丢弃低价值噪声
→ 形成可推理的中间状态
```

但是低质量压缩容易产生“听起来合理”的幻觉。

所以我们需要把压缩过程结构化、证据化、可验证。

---

## 11. 主动结构化 compact，而不是被动等上下文爆掉

coding agent 可能会通过不断读文件、触发 compact / summarization 来维持长任务上下文。

但对我们的目标来说，不能依赖 hidden compact。

我们需要自己构造外部 state：

```text
repo_inventory.json
module_summaries.json
commit_clusters.json
cluster_role_labels.json
development_episodes.json
design_rationale_hypotheses.json
evidence_table.json
open_questions.json
```

建议的 compact 模式：

```text
Map compact:
  单个文件 / 单个 commit-cluster → 局部摘要

Reduce compact:
  多个局部摘要 → 模块摘要 / 历史事件摘要

Episode compact:
  多个 history event → development episode

Replay compact:
  多个 episode → 项目重放叙事
```

每一层都要保留：

```text
证据来源
置信度
开放问题
反证路径
```

---

## 12. Codex / coding agent 的启发

Codex 这类 coding agent 的长任务上下文不是一次性把整个项目塞进模型，而是由多种外部状态共同构成：

```text
任务 prompt
仓库文件系统
AGENTS.md
Git diff
terminal output
测试结果
thread 状态
环境状态
已读文件片段
当前计划
```

因此，我们的 Git History Replay Agent 也应该把项目历史编译成 coding agent 每一步都能使用的 context pack。

一个 episode 的 context pack 应包含：

```text
当前 episode 目标
相关 cluster 摘要
相关模块摘要
关键文件片段
验收测试
已知约束
开放问题
历史证据
```

---

## 13. MVP 建议

不要一开始复刻整个 OpenClaw 或任何 5 万 commit 的大项目。

最小可验证闭环应该是：

> Replay one design concept.

例如先做：

```text
OpenClaw channel 概念重放
```

流程：

```text
1. 抽取 channel 相关 commits / PR / files
2. 聚成 commit-cluster
3. 给 cluster 打角色标签：核心设计、迁移、填充、稳定、摩擦、文档、测试
4. 聚成 5~10 个 development episodes
5. 每个 episode 生成 episode spec
6. 让 coding agent 从简化骨架逐步实现
7. 每一步测试通过
8. 最后输出 replay narrative
```

如果这个能跑通，方法就成立。

然后再扩展到：

```text
gateway
session
plugin
multi-agent
permissions
tool runtime
```

---

## 14. 成功标准

这个系统成功与否，不看总结是否流畅，而看它是否能让用户获得以下能力：

```text
能说清楚项目最初解决什么问题
能说清楚核心概念为什么出现
能说清楚某个抽象吸收了什么复杂度
能区分核心设计 cluster 与外围填充 cluster
能指出哪些 bugfix 暴露设计边界
能判断当前架构哪里稳定、哪里危险
能生成 coding agent 可执行的 episode spec
coding agent 能按 episode spec 实现核心行为并通过测试
```

---

## 15. 下一步行动

建议下一步不要继续抽象讨论，而是做实验：

```text
1. 选择一个 born open-source 或 early-open 项目
2. 抽取 git metadata：commit、parents、author、date、message、changed files、numstat
3. 实现 Githru-style clustering sweep
4. 观察 5 万 commit 最后在不同阈值下聚合成多少 cluster
5. 抽样检查 cluster 是否可解释
6. 专门选一个核心概念，例如 channel，构建完整 episode chain
7. 将 episode spec 交给 coding agent 尝试复刻
```

第一个硬验证点是：

```text
真实项目的 5 万 commits
在 Githru-style 聚类下
到底能否变成数量可控、语义可解释的 commit-clusters？
```

第二个硬验证点是：

```text
这些 clusters 能否进一步编译成 coding agent 可执行的 development episodes？
```

如果这两个问题的答案是肯定的，这个方向就真正站住了。

---

## 16. 一句话总结

> 我们不是让 LLM 读完 5 万个 commit；我们是把 5 万个 commit 压缩成几十到几百个可解释的历史事件，再把这些事件编译成开发 episode，让用户像跟随作者开发项目一样理解设计演化，并最终让 coding agent 沿着关键需求和设计节点复刻项目的核心演化路径。
