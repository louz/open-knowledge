# 05. 实战学习计划：从 0 到 1 建立 OpenKnowledge 架构认知

## 1. 学习目标

你不是要把整个 OpenKnowledge 源码都记住，而是要建立一个“能够看源码、判断模块职责、追踪关键链路”的能力。核心目标有三点：

1. 能说出各 package 的职责边界
2. 能画出启动链和文档流转链路
3. 能明确哪些模块是真正决定系统复杂度的核心实现

## 2. 第一阶段：建立地图（1~2 天）

### 任务

- 读顶层 `README.md`
- 看 root `package.json` 与 workspace 配置
- 看 `packages/*/package.json` 了解职责划分

### 目标

你需要回答：

- 这个项目的“主角包”是什么
- 哪层是 UI，哪层是 runtime，哪层是 native shell
- 这个项目到底想做什么

### 输出

一页纸总结：

- 核心 package 列表及职责
- “OpenKnowledge 是什么”一句话总结
- 最关键的启动入口有哪些

## 3. 第二阶段：看启动链（2~3 天）

### 任务

优先阅读：

- `packages/cli/src/index.ts`
- `packages/cli/src/commands/start.ts`
- `packages/server/src/boot.ts`
- `packages/server/src/server-factory.ts`
- `packages/desktop/src/main/index.ts`
- `packages/desktop/src/main/window-manager.ts`

### 目标

你要掌握：

- openknowledge 是如何启动的
- server 是如何创建的
- desktop / cli 如何复用同一套运行时
- project window 的 lifecycle 是怎么组织的

### 关键问题

- server 启动时执行了哪些前置步骤？
- 为什么 attach 模式和 spawn 模式都很重要？
- 它怎么在桌面端和 CLI 场景中复用相同能力？

## 4. 第三阶段：看前端编辑器（2~3 天）

### 任务

阅读：

- `packages/app/src/App.tsx`
- `packages/app/src/main.tsx`
- `packages/app/src/editor/DocumentContext.tsx`
- `packages/app/src/components/EditorPane.tsx`
- `packages/app/src/lib/doc-hash.ts`

### 目标

你要掌握：

- app shell 是怎么组织的
- editor tab 和 navigation 是怎么工作的
- hash / state / document 之间如何协同
- 前端层如何与 server 对接

### 关键问题

- 这个项目的 UI 是“页面式”还是“document-centric”式？
- tab / navigation / document state 是怎么协调的？
- 为什么它不只是一个普通 React 页面？

## 5. 第四阶段：看文档与存储链路（3~5 天）

### 任务

阅读：

- `packages/server/src/file-watcher.ts`
- `packages/server/src/persistence.ts`
- `packages/server/src/sync-engine.ts`
- `packages/server/src/shadow-repo.ts`
- `packages/server/src/derived-document-index.ts`

### 目标

你要掌握：

- 本地文件如何进入工作流
- 文档如何被写回和恢复
- git / shadow repo 如何支持稳定状态
- 复杂协作状态如何被维护

### 关键问题

- 谁负责持久化？
- 谁负责监听外部变更？
- 谁负责维护 derived index？
- 这个系统如何避免“写丢失 / 冲突 / 状态漂移”？

## 6. 第五阶段：看 AI / skills / MCP / agent 集成（2~3 天）

### 任务

阅读：

- `packages/server/src/api-extension.ts`
- `packages/server/src/agent-sessions.ts`
- `packages/server/src/mcp-mount.ts`
- `packages/server/src/skill-*` 模块
- `packages/server/src/embeddings/...`

### 目标

你需要理解：

- 它和传统文档编辑器的差别在哪里
- AI feature 是如何嵌入知识库工作流的
- skill 和 MCP 是怎样连接到项目内容的

## 7. 第六阶段：做一份“系统结构说明”

在看完关键模块后，你可以用 1~2 页文档写出下列内容：

- OpenKnowledge 的分层设计
- 关键启动链路
- 文档生命周期
- ai/skills/mcp 层的角色
- desktop + cli 之间的关系
- 你认为最关键的 5 个模块

这一步会让你真正从“看代码”进化到“能解释设计”。

## 8. 你的学习顺序建议（最稳）

建议按这个路径：

1. `README.md` + `package.json`
2. `packages/cli/src/commands/start.ts`
3. `packages/server/src/server-factory.ts`
4. `packages/server/src/boot.ts`
5. `packages/app/src/App.tsx`
6. `packages/app/src/editor/DocumentContext.tsx`
7. `packages/server/src/file-watcher.ts`
8. `packages/server/src/persistence.ts`
9. `packages/desktop/src/main/index.ts`
10. `packages/desktop/src/main/window-manager.ts`

如果你只做前 5 个，已经能建立很强的认知基础。

## 9. 学习时的注意事项

### 不要一开始追求“完整理解所有细节”

这个项目很大，特别是 server 层非常复杂。你要先持续地回答：

- 这块代码在系统中属于哪一层？
- 这块状态是什么？
- 它和其他层的连接是什么？

### 先抓主链路，后追细节

最有效的方式不是“逐个文件看”，而是：

- 先定位 node / project lifecycle
- 再看 doc lifecycle
- 再看 editor interaction
- 最后看 AI / MCP / skills extension

## 10. 最终判断标准

当你达到下面标准时，说明你已经进入状态：

- 能说出 `core / server / app / desktop / cli` 各自职责
- 能讲清项目启动链
- 能讲清 document storage / watcher / sync 的职责
- 能解释为什么 OpenKnowledge 不是一个纯前端项目
- 能描述 Electron + Bun + Node + server 的关系

## 11. 结论

OpenKnowledge 的难点不在前端，而在“把本地文件、协作状态、Git、AI、桌面运行时、web UI 整合成统一工作流”。这也是本项目最有价值的学习点。

如果你按这个路线推进，之后再去读某个具体模块，比如 `persistence`、`file-watcher`、`agent-sessions`，你就不会再像“看代码海洋”那样迷失。

---

后续你可以按这些路线继续深入：

- 继续阅读 [README.md](../README.md)
- 回到 [01-architecture-map.md](./01-architecture-map.md)
- 再看 [02-react-ui-layer.md](./02-react-ui-layer.md)
- 然后按 [03-core-server-runtime.md](./03-core-server-runtime.md) 与 [04-desktop-electron-and-cli.md](./04-desktop-electron-and-cli.md) 深挖
