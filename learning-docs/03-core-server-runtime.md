# 03. 核心运行时：server、persistence、watcher 与 git

## 1. 真正的系统中枢在这里

如果前端层告诉你“用户看得到什么”，那么 server 层告诉你“系统到底如何保持状态、同步、恢复、协作”。

OpenKnowledge 的 server 层不是一个简单的 API 服务。它整合了：

- HTTP + WebSocket 入口
- project config
- file watcher
- document persistence
- generated index / graph / backlinks
- git / shadow repo
- agent + MCP + skills

这就是为什么它的核心代码大多放在 `packages/server` 和 `packages/core`。

## 2. 从 `server-factory.ts` 开始

`packages/server/src/server-factory.ts` 是非常关键的入口文件。它定义了 `createServer`，这是整个运行时的核心构造函数。

从名字可以猜到：

- 这是“创建一个 server instance”的地方
- 这个实例通常持有项目所有关键信息
- 它在后台监听文档变化、git 变更、agent 请求、search 请求等

`createServer` 的职责通常包括：

- 初始化 HTTP / Hocuspocus / API route
- 建立 project context
- set up document durability state
- connect file watcher and persistence
- register API extensions
- manage agent session / server observers / health checks

这也是读源码时最值得抓住的“主入口”。

## 3. `boot.ts` 是更高一层的启动器

`boot.ts` 负责更高层的 “启动并运行” 流程：

- 读取 project root / config
- 初始化 project runtime
- 启动 server
- 准备 API / UI / collab 服务
- 返回 booted handle 供 CLI / desktop 调用

它是“启动链”中很关键的中间层：

- CLI / desktop 调用 boot
- boot 组装 project runtime
- runtime 构造 server

所以你可以把它理解为“项目启动编排器”。

## 4. document lifecycle：文档不是一个简单对象

OpenKnowledge 中一个文档可能对应：

- 文件系统中的 markdown/mdx 文件
- in-memory document state
- collaboration state（Yjs/Hocuspocus）
- derived index 结果
- links / backlinks / tags / asset refs

所以 `DocumentDurabilityState`、`DerivedDocumentIndex`、`ContentFilter` 等模块非常重要。

这说明：

> 文档在 OpenKnowledge 中不是单纯的数据对象，而是一组 lifecycle state 的组合。

## 5. file watcher：它如何感知文件变化

`packages/server/src/file-watcher.ts` 是你必须读的一个关键模块。它负责：

- 监控文件变化
- 识别新增、删除、重命名、更新
- 统一写入落盘和索引更新
- 让编辑器 + git + persistence 三者保持同步

这里的核心设计是：

- file system is source of truth
- application state is derived and reconciled
- multiple event sources are normalized

如果你理解了 watcher，你就理解了为什么 OpenKnowledge 能够“本地实时同步、跨窗口同步、支持外部变更检测”。

## 6. persistence：写回与恢复机制

`persistence.ts` 是非常关键的稳定性部分。它和 watcher 配合，解决：

- 用户修改后如何写回磁盘
- changes 如何做 durable store
- 冲突如何检测
- state 如何恢复

这一层在 OpenKnowledge 里的难度相当高，因为它不能只保证“能写进去”，还要保证：

- 不丢内容
- 不覆盖误写
- 能恢复崩溃状态
- 能处理多来源同时更新

## 7. sync engine：不是简单的同步器，而是协作状态机

OpenKnowledge 依赖协作编辑/同步能力（Yjs + Hocuspocus）。`sync-engine.ts` 之类模块负责：

- 协作状态
- document branch / merge / 存活状态
- agent writes / external writes / conflict handling

关键理解：

- 这里的“同步”并不是单纯把两个浏览器的文本同步
- 它更像是“文档状态在本地 fs、collab layer、derived index 之间的逐步 reconcile”

## 8. git 和 shadow repo：为什么它像一个本地知识工作台，而不是单纯编辑器

OpenKnowledge 把 Git 视为基础设施。看这些模块：

- `packages/server/src/shadow-repo.ts`
- `packages/server/src/git-preflight.ts`
- `packages/server/src/project-git.ts`
- `packages/server/src/managed-rename-*` 相关实现

这些代码说明：

- 不是把 Git 当“可选功能”，而是把它当“版本和共享底座”
- 它可以支持 branch, restore, content tracking, diff, persisted documents
- 这也与 share / sync / recovery 能力直接关联

## 9. AI 与 skills 是 server 层的一部分，而不是前端按钮

OpenKnowledge 的高级功能并不是前端里随手写一个 dialog。

关键模块：

- `packages/server/src/agent-sessions.ts`
- `packages/server/src/mcp-mount.ts`
- `packages/server/src/skill-*`
- `packages/server/src/embeddings/...`
- `packages/server/src/api-extension.ts`

这些模块说明：

- agent 管理不是前端状态，而是 server runtime 级能力
- MCP 是一个真正的系统入口
- skills 作为知识库能力的扩展，不只是“插件列表”

## 10. 最重要的读代码策略

读 server 层时建议按这个顺序：

1. `server-factory.ts`
2. `boot.ts`
3. `file-watcher.ts`
4. `persistence.ts`
5. `sync-engine.ts`
6. `shadow-repo.ts`
7. `api-extension.ts`
8. `agent-sessions.ts`

这是最自然的“从环境启动 → 文档生存 → 冲突恢复 → 协作 → AI 能力”的链条。

## 11. 你要学会用的几个问题

每读一个模块，都问自己：

- 这个模块的输入是什么？
- 它维护的状态有哪些？
- 它如何跟 file system / git / collab / UI 交互？
- 它如何处理错误和恢复？
- 它有什么 side effects？

如果你能持续这样追踪，就很容易从“知道代码”走到“知道设计”。

## 12. 结论

OpenKnowledge 的 server 层是它最核心的工程化部分。它不是为了“提供接口”而存在，而是为了：

- 保持文档一致性
- 支持本地和协作状态共存
- 整合 Git / search / skills / agent / MCP
- 让 UI 层成为一个稳定的交互入口

如果你想真正学习 OpenKnowledge 的设计思想，这一层是最重要的。

下一步看：

- [04-desktop-electron-and-cli.md](./04-desktop-electron-and-cli.md)
