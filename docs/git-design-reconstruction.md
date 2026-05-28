# 基于 Git 历史的代码设计过程重建

> 整理日期：2026-05-28  
> 目标：通过分析一个 Git 仓库的 commit 历史、代码结构变化、测试、文档、issue / PR 等证据，重建项目设计从无到有、从简单到复杂的演化过程。

本文整理自一次关于代码重建、设计意图推断、非 LLM 分析、筛子模型和 commit cluster 的讨论。

---

## 1. 项目目标

这个方向不应被定义为“还原作者真实想法”，因为代码和 Git 历史无法证明作者当时脑中的全部动机。

更稳妥的定位是：

> 基于 Git 证据重建项目设计决策与架构演化过程。

也就是说，工具应输出“有证据支持的设计推断”，而不是无依据地讲故事。

最终目标不是简单生成 Git log 摘要，而是把 Git 历史从：

```text
提交记录
```

重构成：

```text
设计事件链
```

---

## 2. 与 Understand-Anything 的关系

`Understand-Anything` 的核心价值在于把当前代码库转化为一张“认知地图”：

- 使用 Tree-sitter 等确定性解析工具抽取文件、函数、类、依赖、调用关系等结构事实；
- 使用 LLM 生成摘要、标签、架构层、业务域、导览路线等语义解释；
- 将结果组织成知识图谱，并通过 Dashboard、搜索、聊天、diff 分析等方式帮助用户理解当前代码库。

它更像是：

```text
当前代码库状态 → 结构分析 → 语义解释 → 当前知识图谱
```

本项目想探索的方向则是：

```text
Git 历史 + diff + AST / CST + 文档 / issue / PR
→ 结构变化证据
→ commit cluster
→ design episode
→ 设计过程重建
```

可以借鉴 `Understand-Anything` 的“确定性结构分析 + 语义解释 + 图谱化输出”思想，但本项目必须额外引入时间维度。

---

## 3. 当前状态图谱 vs 时间设计图谱

| 维度 | Understand-Anything | 本项目方向 |
|---|---|---|
| 核心问题 | 这个代码库现在是怎么组织的？ | 这个代码库是如何一步步变成现在这样的？ |
| 分析对象 | 当前代码快照 / 当前 HEAD | 完整或选定范围内的 commit 历史 |
| 输出形态 | Current-state knowledge graph | Temporal design knowledge graph |
| 主要节点 | 文件、函数、类、依赖、架构层 | commit、commit cluster、design episode、模块演化、决策证据 |
| 主要能力 | 当前代码理解、搜索、导览、diff 影响 | 设计阶段识别、演化路径、关键节点发现、证据链 |

---

## 4. 一个关键认知：非 LLM 也可以逐 commit 分析

不使用 LLM，并不意味着只能做简单 Git 统计。

非 LLM 方法也可以做：

```text
逐 commit diff
逐 commit AST / CST parse
逐 commit 依赖图构建
逐 commit 调用图变化检测
逐 commit 结构变化检测
逐 commit symbol tracking
逐 commit survival analysis
逐 commit 打分
```

区别在于：

```text
LLM 方法：把语义推理成本交给模型。
非 LLM 方法：把分析成本转移到历史遍历、结构计算和图分析。
```

这种方式虽然更耗时，但有几个优点：

1. 可复现；
2. 可解释；
3. 低幻觉；
4. 适合作为 baseline；
5. 后续可以用 LLM 只解释高价值候选点，而不是让 LLM 直接阅读全部历史。

核心原则可以写成：

> LLM 不负责发现事实；LLM 只负责解释由确定性分析发现的事实。

更严格地说：

> 关键 commit 或关键 commit cluster 的发现，应优先由可复现的非 LLM 指标完成。

---

## 5. 去掉非普适规则

讨论中形成了一个非常重要的约束：

> 核心算法只使用普适结构信号，不使用项目命名、目录习惯、关键词、框架经验来判断关键节点。

也就是说，不应把下面这些作为核心判断逻辑：

```text
Service / Provider / Adapter / Repository 等类名关键词
controller / model / repository 等目录名
fix / refactor / feature 等 commit message 关键词
某种框架的目录结构
某个团队的命名习惯
```

这些信息可以作为可选插件或辅助标签，但不应该进入核心算法。

更好的核心原则是：

> 不看它叫什么，只看它在结构中改变了什么。

也就是从：

```text
名字像不像一个设计模式？
目录像不像一个架构层？
commit message 像不像一次重构？
```

转成：

```text
依赖图是否改变？
调用图是否改变？
控制流是否改变？
公开契约是否改变？
模块边界是否改变？
新结构是否长期存活？
后续代码是否围绕它增长？
```

---

## 6. 非 LLM 如何分析“功能性变化”

非 LLM 可以分析功能性变化，但它分析的不是业务语义，而是程序结构和行为路径。

核心公式：

```text
功能性变化
≈ 可执行语法节点变化
+ 控制流变化
+ 调用图变化
+ 数据流变化
+ 契约边界变化
```

### 6.1 从文本 diff 升级到语法 diff

普通 `git diff` 只能告诉我们哪些行变了。

更可靠的方法是：

```text
git diff
  ↓
changed line ranges
  ↓
parse before / after code
  ↓
AST / CST diff
  ↓
把变化映射到函数、类、语句、表达式、调用、条件、返回值
```

例如：

```diff
- if user.is_active:
+ if user.is_active and user.email_verified:
```

非 LLM 可以识别为：

```text
changed node: if condition
change type: boolean condition update
category: control-flow change
functional impact: high
```

### 6.2 控制流变化

控制流变化是功能变化的强信号。

可检测对象包括：

```text
if / else
switch / match
for / while
return
break / continue
throw / catch
guard clause
```

如果这些节点发生变化，通常说明执行路径发生了变化。

### 6.3 调用图变化

另一个重要信号是调用关系变化。

可检测对象包括：

```text
新增调用边
删除调用边
调用目标变化
调用链长度变化
直接调用变间接调用
新增外部依赖调用
```

例如：

```diff
- user = db.find_user(id)
+ user = cache.get(id) or db.find_user(id)
```

可识别为：

```text
added call: cache.get
existing call: db.find_user
data access path changed
```

### 6.4 数据流和输出结构变化

可以跟踪：

```text
变量定义和使用
状态字段读写
数据库字段读写
配置项读写
请求参数映射
返回对象结构
序列化 / 反序列化字段
```

例如返回对象 shape 变化：

```diff
- return { "id": user.id, "name": user.name }
+ return { "id": user.id, "name": user.name, "role": user.role }
```

可识别为：

```text
return object shape changed
public response shape changed
functional_delta: high
```

### 6.5 边界

非 LLM 不能永远证明“用户可见行为一定改变”。

它能较可靠地说明：

```text
这个 commit 修改了可执行语法节点；
这个 commit 修改了控制流 / 调用关系 / 数据结构；
这个 commit 很可能改变功能行为。
```

如果要进一步接近“证明”，需要结合测试、覆盖率、静态调用图到测试的映射、mutation testing 等方法。

---

## 7. 非 LLM 如何发现关键设计决策候选点

非 LLM 不能直接知道作者动机，但可以识别设计决策留下的结构痕迹。

核心公式：

```text
关键设计决策候选点
≈ 新抽象
+ 契约边界变化
+ 依赖变化
+ 模块边界变化
+ 结构重组
+ 图拓扑变化
+ 长期影响
```

### 7.1 新抽象和契约边界变化

不要看名字是否包含某些关键词，而要看结构：

```text
新增可被其他单元依赖的类型或函数
新增 public / exported symbol
新增接口、trait、protocol、抽象类型
函数签名变化
模块入口变化
schema shape 变化
```

如果一个新结构被其他模块依赖，它就是一个潜在契约边界。

### 7.2 依赖图变化

构建 commit 前后的依赖图：

```text
module / package / file / class = node
import / call / inheritance / implements = edge
```

比较：

```text
新增依赖边
删除依赖边
依赖方向变化
核心节点入度变化
环依赖出现 / 消失
模块之间耦合增强 / 减弱
调用路径长度变化
```

例如：

```text
Before:
A → C

After:
A → B → C
```

这可能说明中间抽出了一层结构。

再例如：

```text
Before:
HighLevelModule → ConcreteImplementation

After:
HighLevelModule → AbstractContract
ConcreteImplementation → AbstractContract
```

这可能说明依赖关系被重新组织。

### 7.3 结构重组

设计变化经常表现为：

```text
节点移动
节点拆分
节点合并
函数抽取
类抽取
模块拆分
模块合并
签名变化
调用重连
```

这些都可以通过 AST / CST edit script、symbol tracking、dependency graph diff 来识别，不需要依赖命名关键词。

### 7.4 历史持久性

历史持久性是最普适的设计信号之一。

如果一个 commit 引入的新结构：

```text
长期存在；
被后续代码依赖；
成为依赖图中心节点；
有越来越多调用方；
在后续多个阶段持续被修改或扩展；
```

那么它比一个短期实验更可能是关键设计节点。

---

## 8. 指标不能直接“证明关键”，只能支撑操作性定义

一个重要结论是：

> 指标不能天然证明某个 commit 是关键节点；只有在先定义“关键”的操作性标准之后，指标才能证明它满足这个标准。

不应说：

```text
score > 80，所以它客观上就是关键节点。
```

更严谨的说法应该是：

```text
在本项目定义中：
关键节点 = 结构变化显著 + 影响范围大 + 后续长期存活 + 不是噪声提交。

这个 commit 满足这些条件，所以它是高置信关键节点。
```

阈值不应使用固定绝对值，因为不同仓库规模差异很大。

更可靠的方法是：

```text
仓库内部分布定阈值
+ 强结构信号做硬门槛
+ 功能变化和设计变化分开打分
+ 用长期存活率验证影响
+ 输出证据链而不是只输出分数
```

可使用：

```text
top 1%   = 极高置信关键节点
top 5%   = 候选关键节点
top 10%  = 进入 episode 聚类的候选集合
```

但这只是候选阈值，最终仍应输出证据。

---

## 9. 功能变化分数与设计变化分数应分离

功能变化多，不一定是设计关键节点。

设计变化明显，也不一定伴随大量功能行变化。

因此建议至少分离两个分数：

```text
functional_change_score  # 功能 / 行为变化强度
design_decision_score    # 设计 / 架构变化强度
```

### 9.1 functional_change_score

示例：

```text
functional_change_score =
  control_flow_changes
+ call_graph_changes
+ public_contract_behavior_changes
+ data_model_changes
+ exception_handling_changes
+ executable_statement_changes
- normalized_equivalent_or_mechanical_changes
```

### 9.2 design_decision_score

示例：

```text
design_decision_score =
  new_abstractions
+ dependency_direction_changes
+ module_boundary_changes
+ public_contract_changes
+ structural_reorganization
+ cycle_removed_or_created
+ centrality_change
+ persistence_score
- mechanical_change_penalty
```

注意：这里的 `new_abstractions`、`module_boundary_changes` 等都应由结构事实推出，而不是由名字关键词判断。

---

## 10. 筛子模型：先筛掉最不可能的

讨论中形成的核心算法范式是“筛子模型”。

它不是一开始判断谁最关键，而是先判断：

> 谁最不可能是关键设计节点？

然后逐层压缩历史：

```text
10,000 commits
↓ 第一层：去掉明显机械变化或语义等价变化
3,000 commits
↓ 第二层：保留有可执行结构变化的 commit
800 commits
↓ 第三层：保留有契约 / 结构 / 依赖变化的 commit
200 commits
↓ 第四层：做历史持久性和影响范围分析
50 commits
↓ 第五层：聚类成 commit clusters
10~20 个 design episodes
```

### 10.1 筛子模型原则

前面的筛子负责排除，后面的筛子负责确认。

```text
早期筛子：便宜、快速、高召回、低误杀
后期筛子：昂贵、精细、高精度、强证据
```

第一遍不要太激进。

建议每层筛子有三个出口：

```text
Discard       确定排除
Keep          明显保留
Quarantine    暂存，后面结合上下文再判断
```

`Quarantine` 很重要，可以降低误杀。

### 10.2 无关键词筛子模型

```text
Sieve 1: 归一化差异筛子
判断是否只是文本 / 格式 / 语法等价变化。

Sieve 2: 可执行结构筛子
判断 AST / CST 中是否有可执行节点变化。

Sieve 3: 行为图筛子
判断控制流、调用图、数据流是否变化。

Sieve 4: 契约边界筛子
判断外部可达接口、签名、导出边界、schema shape 是否变化。

Sieve 5: 结构重组筛子
判断是否出现节点移动、拆分、合并、抽取、依赖重连。

Sieve 6: 图拓扑筛子
判断依赖图中心性、环、模块耦合、边界清晰度是否变化。

Sieve 7: 历史持久性筛子
判断新结构是否长期存活并影响后续代码。

Sieve 8: Commit cluster 聚类
把多个相关结构变化聚合成更高层历史单元。
```

---

## 11. Commit cluster：核心概念

Commit cluster 是这次讨论中最重要的概念之一。

它不是 Git 原生对象。Git 原生只有：

```text
commit
branch
merge
tag
parent
```

Commit cluster 是分析工具人为构造出来的概念，含义是：

> 一组在时间、Git 拓扑、修改对象、结构变化、依赖图影响和历史持久性上具有高相关性的 commit。

它的作用是把多个低层 commit 压缩成一个更高层的“变化单元”。

### 11.1 概念来源

Commit cluster 的思想来自软件仓库挖掘和 Git 历史可视化领域。

一个重要参考是 Githru。Githru 面对大型 Git commit graph 时，不直接展示所有 commit，而是使用 graph reconstruction、clustering、Context-Preserving Squash Merge 等方式对历史做抽象，然后提供 summary view 和 comparison view。

因此：

```text
Githru:
large Git commit graph
→ graph reconstruction
→ clustering
→ abstracted history
→ summary view / comparison view
```

在本项目中，这个思想被扩展为：

```text
Git commit graph + diff + AST / CST + dependency graph
→ commit cluster
→ design episode
→ 设计过程重建
```

### 11.2 Commit cluster 与 design episode 的区别

```text
Commit cluster = 算法层概念
Design episode = 解释层 / 产品层概念
```

也就是说：

```text
Commit:
一次原子修改。

Commit cluster:
一组在结构变化、依赖变化、时间、文件、symbol、图影响上高度相关的 commit。

Design episode:
一个具有设计演化意义的 commit cluster。
```

例如：

```text
commit A: 新增一个抽象边界
commit B: 迁移旧实现
commit C: 修改调用方
commit D: 删除旧路径
commit E: 补测试
commit F: 修复迁移后的问题
```

单看每个 commit，可能只是 Candidate 或 Important。

但合起来可能就是：

```text
Critical design episode
```

### 11.3 为什么 commit cluster 比 key commit 更重要

真实开发过程中，一个设计决策往往不是一个 commit 完成的。

它可能分散在多个 commit 中：

```text
第一步：引入新结构
第二步：迁移旧代码
第三步：修改调用方
第四步：补测试
第五步：清理旧路径
```

因此，系统不应该只寻找：

```text
key commit
```

更应该寻找：

```text
key commit cluster
```

Commit cluster 是从“commit 级分析”过渡到“设计过程重建”的桥梁。

---

## 12. Commit cluster 的形成方式

在不使用关键词、不依赖项目习惯的前提下，可以根据普适结构信号聚类：

```text
时间接近
Git DAG 拓扑接近
修改同一组文件
修改同一组 AST symbol
影响同一片调用图
改变同一组依赖边
共同改变同一组 public contract
围绕同一个新引入结构持续变化
共同引入并稳定保留某些结构
```

算法上可以把每个 commit 表示为一个向量或图差异签名：

```text
commit_signature = {
  touched_files,
  touched_symbols,
  ast_edit_events,
  call_graph_delta,
  dependency_graph_delta,
  contract_delta,
  control_flow_delta,
  introduced_entities,
  removed_entities,
  persistence_candidates
}
```

然后根据相似度进行聚类。

聚类不是最终解释，只是产生候选历史单元。

---

## 13. Temporal Knowledge Graph 数据模型

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

### 13.1 节点类型

```text
Commit
CommitCluster
DesignEpisode
File
Symbol              # function / class / interface / method
Module
ContractBoundary
DependencyEdge
CallGraphEdge
ControlFlowRegion
DesignDecision
Issue / PR
Test
Document
Concept
```

### 13.2 边类型

```text
contains_commit
introduces
modifies
removes
renames
moves
extracts
splits
merges
depends_on
calls
changes_contract
changes_control_flow
adds_test_for
documents
motivates
supersedes
reverts
survives_until
```

### 13.3 证据与可信度

每个设计推断都应附带：

```text
confidence: high / medium / low
evidence: [commit, diff hunk, AST event, symbol, graph edge, test, doc, issue]
```

核心目标是避免无证据叙事。

---

## 14. 推荐处理流水线

```text
Git history ingest
    ↓
Commit normalization
    ↓
Diff + AST / CST structural analysis
    ↓
Functional change detection
    ↓
Contract boundary detection
    ↓
Dependency / call graph delta
    ↓
Symbol identity tracking
    ↓
Sieve model filtering
    ↓
Historical persistence analysis
    ↓
Commit cluster detection
    ↓
Design episode construction
    ↓
Temporal knowledge graph
    ↓
Timeline + graph dashboard
    ↓
Narrative report / optional LLM explanation
```

### 14.1 Git history ingest

提取：

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

### 14.2 Diff + AST / CST 分析

对每个 commit 前后的结构变化进行解析：

```text
新增了哪些函数？
删除了哪些类？
哪些函数签名变了？
新增了哪些依赖边？
调用关系有没有变化？
控制流有没有变化？
模块边界有没有移动？
```

### 14.3 Symbol identity tracking

历史演化中会出现：

```text
函数重命名
文件移动
类拆分
方法抽取
模块合并
复制后改造
```

需要识别“同一个结构实体”的生命周期，而不是路径一变就认为是新对象。

可以组合使用：

```text
路径相似度
函数签名相似度
AST subtree fingerprint
调用上下文相似度
git rename detection
结构位置相似度
```

这里也应尽量避免语义关键词。

---

## 15. 重要性等级与变化类型应分离

不要把重要性和变化类型混在一起。

不应设计成：

```text
Critical
Important
Refactor
Bug fix
Noise
```

因为前两个是重要性等级，后几个是变化类型。

更合理的是二维分类：

```text
重要性等级：Critical / Important / Candidate / Background / Noise
变化类型：functional / structural / design / architecture / test / docs / dependency / mechanical
```

例如：

| Commit | 重要性等级 | 变化类型 |
|---|---|---|
| A | Critical | architecture, design |
| B | Important | functional, contract |
| C | Candidate | structural, test |
| D | Background | small functional |
| E | Noise | mechanical |

---

## 16. 一个 commit 的数据结构示例

```json
{
  "commit": "abc123",
  "importance_level": "Important",
  "change_types": ["design", "structural", "test"],
  "scores": {
    "functional_change_score": 42.5,
    "design_decision_score": 81.2,
    "dependency_graph_delta": 76.8,
    "survival_score": 68.0,
    "noise_penalty": 0
  },
  "percentiles": {
    "overall": 96.4,
    "design": 98.1,
    "functional": 72.0
  },
  "signals": [
    "new_contract_boundary",
    "dependency_rewiring",
    "test_cochange"
  ],
  "evidence": [
    "added exported contract-like symbol X",
    "caller A now depends on X instead of concrete symbol Y",
    "added tests covering new call path"
  ]
}
```

---

## 17. Commit cluster 的数据结构示例

```json
{
  "cluster_id": "cluster_001",
  "commits": ["a1b2c3", "d4e5f6", "g7h8i9"],
  "time_range": {
    "start": "2024-03-12",
    "end": "2024-03-18"
  },
  "touched_files": [],
  "touched_symbols": [],
  "structural_events": [],
  "dependency_graph_delta": {},
  "functional_delta": {},
  "persistence": {},
  "importance_level": "Important",
  "evidence": []
}
```

Design episode 可以建立在 commit cluster 之上：

```json
{
  "episode_id": "episode_001",
  "source_cluster": "cluster_001",
  "level": "Critical",
  "interpretation": "一次关键设计演化",
  "evidence": []
}
```

---

## 18. 不建议逐 commit 生成完整图谱

一个常见陷阱是：对每个 commit 都 checkout，然后完整运行一次代码理解，最后把所有结果串起来。

问题包括：

1. 成本很高；
2. 大量 commit 只是机械变化；
3. 许多设计变化跨多个 commit，单个 commit 难以解释；
4. squash merge 可能抹掉中间探索过程；
5. 如果直接让 LLM 读历史，容易生成看似合理但证据不足的叙事。

更好的基本单位不是 commit，而是 commit cluster / design episode。

---

## 19. MVP 路线

### MVP 1：Commit Timeline Summarizer

先做到文件级和结构级：

```text
项目经历了哪些阶段？
每个阶段主要修改了哪些文件 / 模块？
哪些结构出现、消失、移动？
每个阶段的代表性 commit 是什么？
```

### MVP 2：Functional / Structural Change Detector

加入 AST / CST：

```text
可执行语法节点变化
控制流变化
调用图变化
函数 / 类 / 类型生命周期
契约边界变化
```

### MVP 3：Sieve-based Key Commit Candidate Finder

实现筛子模型：

```text
先排除最不可能的
再保留功能变化
再保留结构变化
再验证长期影响
```

### MVP 4：Commit Cluster Detector

把多个相关 commit 聚类成更高层历史单元。

### MVP 5：Design Episode Report

生成设计演化报告：

```text
作者的设计过程大致经历了几个阶段；
每个阶段由哪些 commit cluster 支撑；
哪些证据强，哪些只是推测；
哪些结构长期存活并影响后续代码。
```

---

## 20. 推荐 UI

建议 UI 不只是 Git graph，而是三栏结构：

```text
左侧：时间线 / commit clusters
中间：结构图 / 依赖图随时间变化
右侧：证据卡片 / 设计解释
```

用户点击某个 cluster 后，展示 Evidence Card：

```text
Cluster: cluster_001
时间：2024-03-12 ~ 2024-03-18
涉及 commits：8
涉及结构：若干 symbols / dependency edges / contract boundaries

结构变化：
- 新增一个可被多个模块依赖的契约边界
- 多个调用方迁移到该边界
- 删除旧的直接依赖边

功能变化：
- 调用图变化
- 部分控制流变化

持久性：
- 新结构存活到 HEAD
- 后续被多个模块依赖

结论：
- 高置信设计事件候选
```

---

## 21. 最终项目定位

可以将项目定位为：

> 一个 Git 设计考古工具：从 commit 历史、代码结构变化、测试和文档中，重建项目设计从无到有、从简单到复杂的演化过程。

差异化核心是：

> 把 Git 历史从“提交记录”重构成“设计事件链”。

最核心的理论基础是：

> 关键设计节点不是名字像设计模式的 commit，而是在结构图、行为图、契约边界和长期演化中产生显著、持续、可解释影响的 commit 或 commit cluster。

最重要的算法范式是：

> 使用筛子模型逐层排除最不可能的 commit，最后把高价值候选聚类成 commit cluster，再上升为 design episode。

最重要的数据模型是：

```text
Commit
  ↓
Commit Cluster
  ↓
Design Episode
```

也就是说，本项目的核心不应只是“找出最重要的 commit”，而应是：

> 找出最重要的 commit cluster，并解释它如何构成一次设计演化。
