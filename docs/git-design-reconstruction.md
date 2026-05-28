# 基于 Git 历史的代码设计过程重建

> 目标：通过分析一个 Git 仓库的 commit 历史、代码结构变化、测试、文档、issue / PR 等证据，重建项目设计从无到有、从简单到复杂的演化过程。

## 1. 与 Understand-Anything 的关系

`Understand-Anything` 的核心价值在于把当前代码库转化为一张“认知地图”：

- 使用 Tree-sitter 等确定性解析工具抽取文件、函数、类、依赖、调用关系等结构事实；
- 使用 LLM 生成摘要、标签、架构层、业务域、导览路线等语义解释；
- 将结果组织成知识图谱，并通过 Dashboard、搜索、聊天、diff 分析等方式帮助用户理解当前代码库。

因此，它更像是：

```text
当前代码库状态 → 结构分析 → 语义解释 → 当前知识图谱
```

而本项目想探索的方向是：

```text
Git 历史 + diff + AST / CST + 文档 / issue / PR → 设计事件链 → 设计过程重建
```

可以借鉴 `Understand-Anything` 的“确定性结构分析 + LLM 语义层 + 图谱化输出”思想，但需要额外引入时间维度。

## 2. 目标定位

这个方向不应被定义为“还原作者真实想法”，因为代码和 Git 历史无法证明作者当时脑中的全部动机。

更稳妥的定位是：

> 基于 Git 证据重建项目设计决策与架构演化过程。

也就是说，工具应输出“有证据支持的设计推断”，而不是无依据地讲故事。

## 3. 核心区别：当前状态图谱 vs 时间设计图谱

| 维度 | Understand-Anything | 本项目方向 |
|---|---|---|
| 核心问题 | 这个代码库现在是怎么组织的？ | 这个代码库是如何一步步变成现在这样的？ |
| 分析对象 | 当前代码快照 / 当前 HEAD | 完整或选定范围内的 commit 历史 |
| 输出形态 | Current-state knowledge graph | Temporal design knowledge graph |
| 主要节点 | 文件、函数、类、依赖、架构层 | commit、设计事件、模块演化、决策证据 |
| 主要能力 | 当前代码理解、搜索、导览、diff 影响 | 设计阶段识别、演化路径、设计意图推断、证据链 |

## 4. 不建议逐 commit 生成完整图谱

一个常见陷阱是：对每个 commit 都 checkout，然后完整运行一次代码理解，最后把所有结果串起来。

这样会带来几个问题：

1. 成本很高；
2. 大量 commit 只是格式化、依赖升级、拼写修复；
3. 许多设计变化跨多个 commit，单个 commit 难以解释；
4. squash merge 可能抹掉中间探索过程；
5. LLM 容易被噪声淹没，生成看似合理但证据不足的叙事。

更好的基本单位不是 commit，而是：

> Design Episode：设计片段 / 设计阶段 / 设计事件簇。

例如：

```text
Episode: 引入插件系统
包含 commit:
- add plugin interface
- move hardcoded logic into registry
- add dynamic loader
- update tests
- document extension points

推断结论:
作者从硬编码流程演进到可扩展插件架构。

证据:
接口新增、依赖方向变化、测试新增、commit message、README 更新。
```

## 5. 建议的数据模型：Temporal Knowledge Graph

普通代码图谱关注：

```text
现在有哪些模块？它们怎么依赖？
```

时间设计图谱关注：

```text
这些模块是什么时候出现的？
为什么出现？
替代了什么？
解决了什么问题？
后续又被怎样修改？
```

### 5.1 节点类型

建议至少包含以下节点：

```text
Commit
File
Symbol              # function / class / interface / method
Module
ArchitectureLayer
DesignDecision
DesignEpisode
Issue / PR
Test
Document
Concept             # auth, plugin, cache, scheduler, renderer 等
```

### 5.2 边类型

建议包含以下关系：

```text
introduces          # 引入某个模块 / 抽象
modifies            # 修改某个符号 / 文件
removes             # 删除
renames             # 重命名
extracts            # 从 A 抽取出 B
splits              # 拆分模块
merges              # 合并模块
depends_on          # 依赖
inverts_dependency  # 依赖反转
adds_test_for       # 为某功能添加测试
documents           # 文档解释某设计
motivates           # commit message / issue 解释某决策
supersedes          # 新设计替代旧设计
reverts             # 回滚
```

### 5.3 证据与可信度

每个设计推断都应附带：

```text
confidence: high / medium / low
evidence: [commit, diff hunk, code symbol, test, doc, issue]
```

这可以避免 LLM 无证据地“编故事”。

## 6. 推荐处理流水线

```text
Git history ingest
    ↓
Commit normalization
    ↓
Diff + AST / CST structural analysis
    ↓
Symbol identity tracking
    ↓
Commit semantic classification
    ↓
Design episode clustering
    ↓
Rationale / decision extraction
    ↓
Temporal knowledge graph
    ↓
Timeline + graph dashboard
    ↓
Narrative report / chat interface
```

### 6.1 Git history ingest

提取每个 commit 的基础信息：

```text
commit hash
parent hash
author
date
message
changed files
diff hunks
rename / delete / add
tags
branches
PR / issue references
```

不要只看 `git log --oneline`。至少要保留 full message、diff summary、changed paths 和 parent relationship。

### 6.2 Diff + AST / CST 分析

对每个 commit 前后的结构变化进行解析：

```text
新增了哪些函数？
删除了哪些类？
哪些函数签名变了？
新增了哪些 import？
调用关系有没有变化？
模块边界有没有移动？
测试有没有同步变化？
```

这部分应尽量使用 Tree-sitter、AST、CST 或语言服务完成，而不是完全依赖 LLM。

### 6.3 Symbol identity tracking

这是该项目的技术难点之一。

历史演化中会出现：

```text
函数重命名
文件移动
类拆分
方法抽取
模块合并
复制后改造
```

需要识别“同一个概念”的生命周期，而不是路径一变就认为是新对象。

可以组合使用：

```text
路径相似度
函数签名相似度
AST subtree fingerprint
代码 embedding
调用上下文相似度
git rename detection
commit message hint
```

示例：

```text
src/auth/login.ts::loginUser
↓ rename / move
src/modules/auth/service.ts::authenticateUser
```

工具应判断：这可能是同一个设计概念的演化，而不是两个无关函数。

### 6.4 Commit semantic classification

每个 commit 可以分类为：

```text
feature introduction
bug fix
refactor
abstraction extraction
dependency inversion
API change
test addition
performance optimization
security hardening
configuration change
documentation
cleanup
revert
migration
```

LLM 可以参与分类，但输入最好是结构摘要，而不是完整 diff。

示例：

```text
Commit message:
"extract cache interface"

Structural diff:
- added interface CacheStore
- RedisCache now implements CacheStore
- MemoryCache added for tests
- UserService depends on CacheStore instead of RedisCache

Classifier output:
type: abstraction_extraction
design_signal: dependency inversion for cache backend
confidence: high
```

### 6.5 Design episode clustering

不要直接让用户阅读几千个 commit。应将相关 commit 聚类成设计事件或阶段。

可使用的聚类依据：

```text
时间接近
修改同一批文件
涉及同一批 symbol
commit message 语义相近
同一个 PR / issue
共同引入或修改某个架构边
共同服务某个业务概念
```

输出应类似：

```text
阶段 1：最小可用版本
阶段 2：抽象出数据访问层
阶段 3：引入插件架构
阶段 4：性能优化与缓存
阶段 5：稳定 API 与测试体系
```

## 7. 如何表达“设计意图”

为了避免过度推断，建议将结论分为三层可信度。

### 7.1 高可信：结构事实

```text
这个 commit 新增了 AuthService。
UserController 从直接访问 DB 改为调用 AuthService。
新增了 AuthServiceTest。
```

### 7.2 中可信：设计推断

```text
这看起来是在把认证逻辑从 Controller 中抽离出来，形成 Service 层。
```

### 7.3 低可信：动机假设

```text
作者可能是为了提升可测试性和职责分离。
```

低可信结论必须用“可能”“推测”等措辞，并展示证据。

## 8. 可借鉴的现有工具

目前还没有一个成熟工具完整做到：

```text
Git commit 历史
+ AST / 调用关系演化
+ commit message / PR / issue / 文档
+ 设计事件聚类
+ 作者设计过程重建
+ 证据链展示
+ 可信度标注
```

但以下工具和研究方向值得借鉴。

### 8.1 CodeScene / Code Maat

这类工具最接近“代码考古”。

它们关注：

```text
热点文件
逻辑耦合
代码年龄
知识分布
作者贡献
bus factor
经常一起修改的模块
```

它们适合提供行为数据基础，但通常不会直接回答“为什么引入这个设计抽象”。

### 8.2 Githru

Githru 的核心思想是：不要直接展示所有 commit，而是把大型 Git graph 聚类、抽象化，然后提供 summary view 和 comparison view。

可以理解为：

```text
Githru:
commit graph → 聚类 → 历史总览 / cluster 对比

本项目:
commit graph + diff + AST → 设计事件聚类 → 作者设计过程重建
```

其中：

- summary view：帮助用户获得项目历史的全局总览；
- comparison view：帮助用户比较不同 commit cluster 或阶段之间的差异。

### 8.3 GitEvo

GitEvo 关注代码演化分析，并使用 Tree-sitter 在 CST 层面定义代码演化指标。

它适合借鉴：

```text
函数 / 类 / 接口何时出现
代码结构如何变化
某些语法结构如何随时间增长或消失
```

### 8.4 PyDriller

PyDriller 是一个 Python 的 Git 仓库挖掘框架，适合作为底层数据采集层。

可用于：

```text
遍历 commit
读取 commit message
获取 modified files
获取 diff
获取前后版本源码
提取作者、时间、分支信息
```

### 8.5 Gource

Gource 适合做项目演化动画：目录像树枝，文件像叶子，开发者在时间轴上对文件做贡献。

它适合借鉴“时间播放”和“文件树随时间变化”的视觉表达，但不是设计意图分析工具。

### 8.6 git-of-theseus / Hercules

这类工具适合分析：

```text
代码增长趋势
代码行存活率
作者贡献
文件维度统计
所有权变化
模块耦合
```

它们可以补充“历史行为指标”，但不能直接重建设计过程。

## 9. 建议的 MVP 路线

### MVP 1：Commit Timeline Summarizer

输入一个 repo，输出：

```text
项目经历了哪些阶段？
每个阶段主要修改了哪些目录？
哪些模块出现、消失、重命名？
每个阶段的代表性 commit 是什么？
```

先做到文件级别，不必一开始就做函数级别。

### MVP 2：Symbol Evolution Tracker

追踪函数、类、接口的生命周期：

```text
AuthService:
- introduced in commit A
- extracted from LoginController in commit B
- interface added in commit C
- tests added in commit D
- renamed in commit E
```

### MVP 3：Design Episode Detector

把多个 commit 聚类成设计事件：

```text
Episode: 从硬编码支付流程演进到 Provider 架构
Evidence:
- added PaymentProvider interface
- added StripeProvider
- added PaypalProvider
- CheckoutService now depends on PaymentProvider
- README documents provider extension
```

### MVP 4：Design Reconstruction Report

生成类似这样的报告：

```text
作者的设计过程大致经历了 5 个阶段：

1. 先实现最小功能闭环
2. 发现认证逻辑散落在多个 controller
3. 抽出 AuthService
4. 为测试引入接口
5. 后来把 auth 模块独立成 package

证据强的结论：
...

可能但不确定的动机：
...
```

## 10. 推荐 UI

建议 UI 不只是 Git graph，而是三栏结构：

```text
左侧：时间线
中间：架构图随时间变化
右侧：设计解释 + 证据
```

用户点击某个阶段后，展示类似 Evidence Card 的内容：

```text
阶段：抽象缓存层
时间：2024-03-12 ~ 2024-03-18
涉及 commits：8
涉及模块：cache, user-service, tests

设计变化：
- 新增 CacheStore interface
- RedisCache 变成实现类
- UserService 不再直接依赖 Redis

推断意图：
- 降低对 Redis 的直接耦合
- 方便单元测试和未来替换缓存后端

证据：
- commit message
- diff hunk
- 新增测试
- README 更新
```

## 11. 实用原则

最重要的原则是：

> 不要让 LLM 直接读历史；先把历史压缩成结构化事实，再让 LLM 解释。

不推荐：

```text
LLM，请阅读这 300 个 commit，然后告诉我作者设计意图。
```

推荐：

```text
Parser：先提取每个 commit 的结构变化。
Clusterer：先聚合成设计事件。
Retriever：找出相关 commit、diff、测试、文档。
LLM：基于这些证据生成设计过程解释。
```

这可以显著降低幻觉，也更容易评估。

## 12. 推荐项目定位

可以将项目定位为：

> 一个 Git 设计考古工具：从 commit 历史、代码结构变化、测试和文档中，重建项目设计从无到有、从简单到复杂的演化过程。

差异化核心是：

> 把 Git 历史从“提交记录”重构成“设计事件链”。

优先级最高的功能建议是：

> Design Episode + Evidence Card

只要能把一段 commit 历史变成“设计阶段 + 证据链 + 架构变化图”，这个项目就已经和现有代码理解工具拉开差距。
