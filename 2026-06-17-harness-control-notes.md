# Harness × 控制论 × 子任务分解（对话总结）

## 1. 核心问题
用户在探索如何在 harness 系统中进行复杂任务分解，并关心：

- 子任务如何划分边界
- 是否可以用控制论解释 task decomposition
- 控制论是否提供“收敛/可控性”的理论保证
- 如何将理论落地到 agent / harness 设计中

---

## 2. 关键认知升级

### 2.1 子任务本质不是语义单位
子任务划分不基于语义完整性，而基于状态是否可度量与可控制。

### 2.2 控制论视角重构 task decomposition
子任务 = 状态空间中的一个“可控跃迁”

必须满足：
- 可观测性（observability）
- 可验证性（verifiability）
- 低耦合（low coupling）

### 2.3 feedback loop 是 harness 核心结构
系统建模为：
- state（任务状态 / repo / context）
- controller（LLM + orchestration）
- feedback（eval / verifier）
- transition（task execution）

---

## 3. 控制论讨论

### 3.1 维纳控制论
- 强调 feedback / noise / regulation
- 提供系统直觉
- 不提供严格可控性证明

结论：解释框架

### 3.2 现代控制论
核心工具：
- state-space
- controllability / observability
- Lyapunov stability
- dynamic programming

结论：工程主体系

### 3.3 收敛性
系统稳定条件来源：
- Lyapunov 函数下降
- 特征值稳定性
- 可控性矩阵条件

---

## 4. harness 重构

- task decomposition → state partitioning
- eval → error function
- retry → feedback correction
- harness → controller design

---

## 5. push-github 定义

push-github = 将对话内容总结为 Markdown 并提交到 GitHub 仓库 wanghuaiqing2023-blip/cold-river

---

## 6. 当前阶段
用户已进入：
- control-theoretic system design
- agent harness architecture
