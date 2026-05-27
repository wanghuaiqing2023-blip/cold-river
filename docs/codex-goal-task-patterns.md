# 适合使用 Codex `/goal` 的任务类型

`/goal` 适合的不是“问一个问题”，而是让 Codex 在一个明确、可验证的目标下持续推进：目标清楚，过程可能需要多轮迭代，每一步都可以通过测试、构建、基准、截图或人工可审查产物来验证。

一个简单判断法：

> 适合 `/goal` 的任务 = 目标明确 + 路径不完全明确 + 可以自动或半自动验证 + 需要多轮迭代 + 有边界约束。

不太适合的任务包括：单次解释、简单问答、小范围代码片段改写、没有明确验收标准的创意发散。

## 官方参考

- Codex use case: Follow goals: https://developers.openai.com/codex/use-cases/follow-goals
- OpenAI Cookbook: Using goals in Codex: https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex

## 1. 技术栈迁移，但行为保持一致

典型任务：

- React 项目迁移到 Vue、Svelte 或 Solid。
- Express 后端迁移到 FastAPI、Go Fiber 或 Spring Boot。
- Electron 桌面应用迁移到 Tauri。
- REST API 服务迁移到 GraphQL，但对外能力保持等价。
- Webpack 迁移到 Vite 或 Rspack。
- npm/yarn 项目迁移到 pnpm 或 monorepo 工具链。

适合原因：迁移过程通常不是一次改完，而是“迁移 → 构建 → 测试 → 修复 → 再验证”。

示例：

```text
/goal Migrate this React + Webpack project to Vite while preserving all existing runtime behavior. Verify by passing the full test suite, keeping existing routes working, and confirming the production build succeeds.
```

## 2. 依赖替换，但对外 API 不变

典型任务：

- 把 Moment.js 替换成 date-fns 或 Day.js。
- 把 lodash 的使用逐步替换成原生 JavaScript。
- 把旧 ORM 替换成 Prisma、Drizzle 或 SQLAlchemy。
- 把某个支付、邮件、日志、缓存 SDK 替换成新的供应商。
- 把自研工具函数替换成标准库实现。

判断标准：内部实现变了，但调用方不应该感知变化。

示例：

```text
/goal Replace Moment.js with Day.js across the codebase while preserving all date formatting, parsing, and timezone behavior. Verify with existing tests and add regression tests for any ambiguous date behavior.
```

## 3. 大版本升级并修复破坏性变更

典型任务：

- Python 2 迁移到 Python 3。
- Django 3 升级到 Django 5。
- Rails 6 升级到 Rails 8。
- React 17 升级到 React 19。
- Node 16 升级到 Node 22。
- Android Gradle Plugin、Xcode 或 Swift 大版本升级。

适合原因：升级过程中经常会遇到“修一个错又暴露下一个错”的连续验证循环。

示例：

```text
/goal Upgrade this project from Node 16 to Node 22. Keep all public behavior unchanged, update deprecated APIs where necessary, and stop only when install, lint, tests, and production build all pass.
```

## 4. 类型化改造

典型任务：

- JavaScript 项目迁移到 TypeScript。
- Python 项目补全类型注解并通过 mypy 或 pyright。
- Ruby 项目加入 Sorbet。
- PHP 项目加入 Psalm 或 PHPStan。
- Kotlin、Swift、TypeScript 中系统性消除 nullable 风险。

成功条件：类型检查通过、测试通过、运行行为不变。

示例：

```text
/goal Convert this JavaScript package to TypeScript without changing runtime behavior. Add types for public APIs, preserve the generated package interface, and verify with typecheck, tests, and build.
```

## 5. 测试补齐到指定覆盖率或行为保证

典型任务：

- 为一个没有测试的库补单元测试。
- 为核心业务流程补集成测试。
- 为 API 补 contract tests。
- 为 UI 补 Playwright 或 Cypress 端到端测试。
- 给修复过的 bug 补回归测试。

适合原因：可以把成功标准写成“直到覆盖率、关键路径或失败用例被验证为止”。

示例：

```text
/goal Add regression and integration tests for the checkout flow until the critical user paths are covered. Verify by passing the full test suite and documenting any behavior that cannot be tested locally.
```

## 6. 性能优化到明确指标

典型任务：

- p95 延迟降到指定阈值以下。
- 首屏加载时间减少指定比例。
- 构建时间从几分钟降到一分钟以内。
- 内存占用降低到某个阈值。
- SQL 查询次数减少。
- 批处理吞吐量提升。

适合原因：Codex 可以反复执行“测量 → 定位瓶颈 → 修改 → 再测量”。

示例：

```text
/goal Reduce p95 API latency below 150ms on the local benchmark while keeping correctness tests green. Record each attempted optimization and stop if no defensible improvement path remains.
```

## 7. 修复 flaky tests 或难复现 bug

典型任务：

- CI 上偶发失败的测试。
- 并发 race condition。
- 特定平台才出现的路径、编码、时区 bug。
- 偶发 E2E 失败。
- 缓存失效或状态污染问题。

适合原因：路径通常未知，但成功条件明确：能稳定复现、定位原因、修复，并重复验证不再失败。

示例：

```text
/goal Reproduce and fix the flaky user-session test. Verify by running it 100 times locally and ensuring the full related test suite passes. If it cannot be reproduced, report all evidence and likely causes.
```

## 8. 架构重构，但外部行为不变

典型任务：

- 把巨型文件拆成模块。
- 把 MVC 改成分层架构。
- 把同步逻辑改成异步队列。
- 把 monolith 中某个模块抽成 package。
- 把业务逻辑从 controller 移到 service/domain 层。
- 清理循环依赖。

适合原因：可以分 checkpoint，每次重构后跑测试，确保没有行为回归。

示例：

```text
/goal Refactor the billing module into clear domain, service, and adapter layers without changing public API behavior. Verify with existing tests, add tests for extracted boundaries, and keep commits logically grouped.
```

## 9. 平台移植

典型任务：

- Linux-only 工具移植到 macOS 或 Windows。
- 浏览器端库移植到 Node.js。
- Node CLI 移植到 Bun 或 Deno。
- CPU 实现迁移到 GPU、CUDA 或 WebGPU。
- 本地文件存储迁移到 S3 或 GCS。
- SQLite 迁移到 Postgres。

适合原因：核心语义要保留，但底层平台能力不同，需要迭代修补。

示例：

```text
/goal Make this CLI work on Windows, macOS, and Linux without changing command behavior. Verify path handling, shell execution, tests, and documented examples on all supported platforms where possible.
```

## 10. UI 视觉或交互等价迁移

典型任务：

- 旧页面重写成新的组件系统。
- CSS Modules 迁移到 Tailwind。
- Bootstrap 迁移到 shadcn/ui。
- 移动端布局重写但视觉保持一致。
- Figma 设计稿落地到现有前端。
- 旧 dashboard 重构为响应式布局。

适合原因：可以用截图、Playwright、视觉回归测试做验证。

示例：

```text
/goal Rebuild the legacy dashboard using Tailwind and shadcn/ui while preserving the current visual layout and interactions. Verify with Playwright screenshots for desktop and mobile breakpoints.
```

## 11. 文档、示例、教程与代码保持同步

典型任务：

- 为 SDK 补完整文档。
- 自动检查 README 中所有命令是否可运行。
- 根据代码生成 API reference。
- 把旧文档迁移到 Docusaurus、Mintlify 或 VitePress。
- 为每个核心功能补可运行示例。
- 修复文档中失效的命令、截图、配置项。

适合原因：只要要求“示例可运行、命令真实、文档构建通过”，它就从纯写作任务变成可验证的目标任务。

示例：

```text
/goal Update the documentation so every public API has a working example. Verify docs build locally and every documented command or code snippet runs successfully.
```

## 12. 安全或合规硬化到明确检查通过

典型任务：

- 修复 npm audit、pip-audit 或 cargo audit 问题。
- 消除硬编码 secrets。
- 给 API 加权限检查。
- 修复 SAST 报告中的高危项。
- 加 CSRF、XSS、SQL injection 防护。
- 给敏感日志脱敏。

适合前提：有明确扫描器、测试或报告作为验证面。

示例：

```text
/goal Resolve all high-severity security findings from the audit report without changing public behavior. Verify by rerunning the scanner, passing tests, and documenting any accepted residual risk.
```

## 13. 数据库或数据模型迁移

典型任务：

- schema 重构但业务查询保持一致。
- 从 MongoDB 迁移到 Postgres。
- 增加 migration 脚本和 rollback。
- 重写索引策略。
- 数据清洗脚本从一次性脚本变成可重复任务。
- 老字段迁移到新字段并保证兼容期。

示例：

```text
/goal Migrate the user profile schema to the new normalized design while preserving application behavior. Add forward and rollback migrations, update queries, and verify with integration tests and seed data.
```

## 14. CLI、SDK 或 API 的兼容实现

典型任务：

- 用新语言重写一个 CLI，但命令、参数、输出保持兼容。
- 为现有 REST API 生成新的 SDK。
- 重新实现一个协议解析器。
- 替换内部服务，但保持 HTTP contract 不变。
- 写一个 drop-in replacement library。

适合原因：这类任务的核心目标是 behavioral parity，即行为等价。

示例：

```text
/goal Reimplement this CLI in Rust while preserving command names, flags, exit codes, stdout/stderr behavior, and config file compatibility. Verify against the existing CLI test fixtures.
```

## 15. Prompt、agent 或 eval 驱动优化

典型任务：

- 有 eval suite 的 prompt 优化。
- RAG 检索质量调参。
- agent 工具调用成功率优化。
- 分类器 prompt 优化到某个准确率。
- 自动分析失败样例并迭代提示词。

适合原因：当存在 eval suite 时，可以检查失败样例、更新 prompt、重跑 eval，直到分数改善或达到停止条件。

示例：

```text
/goal Improve the support-triage prompt until the eval score is above 0.92 without increasing false positives above the current baseline. Inspect failures, update the prompt, rerun evals, and summarize tradeoffs.
```

## 写好 `/goal` 的模板

推荐结构：

```text
/goal <目标>

Constraints:
- Preserve <必须保持不变的行为/API/输出/兼容性>.
- Do not change <禁止改动的范围> unless necessary.
- Keep changes minimal and reviewable.

Verification:
- Run <install/build/test/typecheck/lint/benchmark/eval>.
- Add regression tests for any behavior changed or clarified.
- Document any assumptions, skipped checks, or residual risks.

Stop condition:
- Stop when <明确通过条件> is satisfied.
- If blocked, report the blocker, evidence gathered, and next best action.
```

## 总结

最适合 `/goal` 的任务通常具有以下特征：

1. 不是一次性回答，而是一个需要执行的工程目标。
2. 可以被测试、构建、截图、benchmark、eval 或文档构建验证。
3. 任务路径可能不确定，需要 Codex 自己探索。
4. 需要在保持既有行为的同时完成迁移、重构、优化或修复。
5. 有清晰的停止条件，避免无限迭代。
