# OpenKnowledge 学习路线总览

这份路线面向“熟悉 React 页面开发，但不熟悉 Electron / Bun / monorepo server-runtime 的开发者”。目标不是逐行阅读全部源码，而是建立“看源码的地图”：知道每个 package 在干什么、关键连接链路是什么、什么时候该深入哪个模块。

## 1. 先建立整体认知：这个项目到底解决了什么问题

OpenKnowledge 不是简单的 Markdown 编辑器，而是一个“本地知识库 + 协作编辑 + AI Agent 入口”的桌面/网页应用。它的核心特征包括：

- 以 Markdown / MDX 文件为主的知识文档存储
- 真正的 WYSIWYG 编辑体验（与 Notion / Google Docs 相近）
- 本地文件系统作为底层源头，而非纯数据库
- Git 作为版本控制与共享基础能力
- Agent / MCP / Skills / semantic search 等 AI 能力嵌入
- 桌面端与 web UI 共享一套核心能力

从架构看，它并不是“一个前端页面 + 一张数据库表”，而是“本地 file-system + server + editor + git sync + electron shell + AI integration”的组合系统。

## 2. 你应该如何学习：按“从外到内、从运行时到数据流”读

建议顺序：

1. 先读 package 级架构和启动入口
2. 再读 app 层 UI 入口与页面状态管理
3. 然后读 server 层的 document lifecycle 与 persistence
4. 再看 desktop 层的 Electron main process 与 window manager
5. 最后看 cli / MCP / skill / agent 集成与项目初始化流程

这条路径能最快建立“它是怎么跑起来的”。

## 3. 项目的核心 package 关系

- `packages/core`：纯业务模型、schema、协议、共享逻辑。它像“领域层”。
- `packages/server`：运行时服务，管理项目文件、Git/Collab、API、watcher、search、MCP 等。
- `packages/app`：React 前端编辑器和页面 UI。
- `packages/desktop`：Electron 主进程，负责窗口、应用生命周期、native 能力。
- `packages/cli`：`ok` 命令行入口，负责启动本地项目 server / UI / 初始化项目。
- `packages/plugin`：与外部 Agent 扩展/集成相关。
- `packages/native-config`：native 层配置与本机能力桥接。

## 4. 学习路线总表

| 阶段 | 目标 | 关键阅读文件 | 产出 |
| --- | --- | --- | --- |
| 1 | 读懂项目整体结构 | `README.md`、`package.json`、各 package package.json | 建立 monorepo 认知 |
| 2 | 从入口看运行时启动 | `packages/cli/src/commands/start.ts`、`packages/server/src/boot.ts`、`packages/server/src/server-factory.ts` | 理解 server 是怎么起的 |
| 3 | 看前端编辑器是如何挂起来的 | `packages/app/src/App.tsx`、`packages/app/src/main.tsx` | 理解 UI 与 document context |
| 4 | 学会数据流：文档、watcher、git、冲突 | `packages/server/src/file-watcher.ts`、`packages/server/src/persistence.ts`、`packages/server/src/sync-engine.ts` | 明确内容源头与写回 |
| 5 | 看桌面端如何接住 app | `packages/desktop/src/main/index.ts`、`packages/desktop/src/main/window-manager.ts` | 理解 Electron shell |
| 6 | 看 CLI + MCP + skill 集成 | `packages/cli/src/index.ts`、`packages/cli/src/commands/start.ts`、`packages/server/src/index.ts` | 理解产品入口 |
| 7 | 形成架构判断 | 各阶段笔记 + 设计问题 | 能围绕代码作结构分析 |

## 5. 推荐阅读顺序（最实用）

### 第 1 轮：建立地图

- 先看顶层 `README.md`
- 再看根 `package.json` 中的 script 与 workspace 组织
- 再看 `packages/*/package.json`，确认每个 package 的职责

这一步的目标是“知道项目的部件图”。

### 第 2 轮：理解启动链路

建议阅读：

- `packages/cli/src/commands/start.ts`
- `packages/server/src/boot.ts`
- `packages/server/src/server-factory.ts`
- `packages/desktop/src/main/index.ts`

你要关注：

- server 是怎么被创建的
- 用户打开项目时，应用如何决定是 spawn 还是 attach
- app shell 是如何通过 HTTP / WebSocket / local API 访问服务器的

### 第 3 轮：理解前端编辑器

建议阅读：

- `packages/app/src/App.tsx`
- `packages/app/src/main.tsx`
- `packages/app/src/editor/DocumentContext.tsx`
- `packages/app/src/components/EditorPane.tsx`

你要关注：

- hash 路由、tab、navigation target 是怎么处理的
- editor context 如何管理 active document / open tabs / document transitions
- React 侧如何和底层 server/API 存续交互

### 第 4 轮：理解 backend runtime 核心机制

建议阅读：

- `packages/server/src/file-watcher.ts`
- `packages/server/src/persistence.ts`
- `packages/server/src/sync-engine.ts`
- `packages/server/src/derived-document-index.ts`
- `packages/server/src/shadow-repo.ts`

你要关注：

- 文件监控和变更回流
- markdown 解析、doc 的读写
- git + shadow repo 是如何兼容本地文档与协作状态的
- 服务端如何保持一致性与故障恢复

### 第 5 轮：拆解 Electron / desktop 细节

建议阅读：

- `packages/desktop/src/main/index.ts`
- `packages/desktop/src/main/window-manager.ts`
- `packages/desktop/src/main/bootstrap.ts`

你要关注：

- BrowserWindow 的生命周期
- “one window ⇄ one utility process ⇄ one collab server” 的设计
- windows / attach / project 路径管理

### 第 6 轮：理解产品能力扩展面

建议阅读：

- `packages/server/src/api-extension.ts`
- `packages/server/src/agent-sessions.ts`
- `packages/server/src/mcp-mount.ts`
- `packages/server/src/skill-*` 模块

你要关注：

- 这里是 Agent、MCP、skills、semantic search 这些高层能力真正落地的位置

## 6. 建议的阅读原则

### 先抓“主链路”，不要一开始追细节

很多源码里细小的实现很容易让人迷失。建议第一次阅读时，优先回答：

- 谁启动了程序？
- 谁监听了文件变更？
- 谁决定当前 doc 的状态？
- 谁把 Markdown 写回磁盘？
- 谁把 UI 和 server 连起来？

### 先看 public API，再看内部实现

比如：

- `server-factory.ts` 给出 createServer 的接口
- `boot.ts` 给出 bootServer 的公开环境
- `index.ts` 的 export 说明了公开能力边界

这会帮助你快速理解模块边界。

### 把“函数名当成注释”

OpenKnowledge 的源码很注重函数命名和注释，很多函数名本身就编码了设计意图。像：

- `createServer`
- `bootServer`
- `createContentFilter`
- `startWatcher`
- `resolveNavigationTarget`
- `registerWrite`
- `DocumentDurabilityState`

这些名字就是阅读线索。

## 7. 如果你只想快速建立理解：最小必要阅读列表

如果你没有时间读全部代码，至少重点看下面几份：

- `README.md`
- `package.json`
- `packages/server/src/server-factory.ts`
- `packages/server/src/boot.ts`
- `packages/server/src/index.ts`
- `packages/desktop/src/main/index.ts`
- `packages/desktop/src/main/window-manager.ts`
- `packages/app/src/App.tsx`
- `packages/cli/src/commands/start.ts`

这 9 份文件足以建立 70% 以上的系统认知。

## 8. 进一步深入时的研究问题

建议你在读每个模块时带着这些问题：

1. 这个模块在系统里属于哪一层？
2. 它的输入输出是什么？
3. 它依赖哪些底层原语（file watcher / git / Yjs / Hocuspocus / Electron IPC）？
4. 它的错误边界是什么？
5. 它如何做到本地优先、协作优先、状态恢复优先？
6. 它和其他 package 之间的接口约束是什么？

## 9. 这份学习路线的目标

你的最终目标不是“熟悉每个文件”，而是建立这种理解：

- OpenKnowledge 不是一个单体 React 应用，它是一个分层的本地知识工作台
- 编辑器层、server 层、desktop shell 层、CLI 层分工明确
- 它把本地文件、Git、协作、AI Agent 这些能力揉在一起
- 真正难的不是 UI，而是“如何在本地文件系统中保持稳定、可恢复、可协作、可追踪”的运行时设计

这也是为什么它的核心代码能出现在 `packages/server` 和 `packages/core` 中，而不是只停留在 `packages/app`。

## 10. 建议的第一个研究任务

如果你现在要开始动手实践，建议先做一项“最小闭环研究”：

- 追踪：用户打开一个项目 → CLI / Desktop 启动 server → app 连接 collab → 文档读写 → file watcher 触发存盘

你可以把这个闭环拆成 4 个任务：

1. 从 `cli`/desktop 入口看启动入口
2. 从 `server-factory` 看 server 创建
3. 从 `App.tsx` 看前端页面挂载
4. 从 `file-watcher`/`persistence` 看文档回写机制

一旦你能把这条链路讲清楚，你就已经对项目架构有了真正的掌握。

---

后续文档里会进一步拆解这条路径，按“架构地图 → 前端 UI → server/runtime → desktop + CLI → 学习计划”分层展开。
