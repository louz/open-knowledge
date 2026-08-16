# 02. React UI 层：从 App.tsx 开始读前端

## 1. 这个层最适合你

如果你有 React 页面开发背景，那么 OpenKnowledge 的前端层是最容易进入的入口。你不需要先懂 Electron，也不需要先看每个复杂的 server 模块；先从 App 层看状态驱动和 UI 结构，就能建立“它如何像一个编辑器工作台”这个感觉。

## 2. 先看入口：`packages/app/src/App.tsx`

`App.tsx` 是整个 app 的核心入口之一。它不是简单的 `return <div />`，它是一个大型组合式 UI：

- layout：sidebar、editor pane、tabs、command palette
- navigation：hash 路由、open tabs、document transitions
- config：`ConfigProvider`、`useConfigContext`
- document lifecycle：`DocumentProvider` 与 `useDocumentContext`
- overlays：dialog、settings、skill dialogs

你可以把它理解为：

> App 层负责把“文档状态、导航状态、配置、UI Shell”组织成一个工作台。

## 3. 关键状态：DocumentContext

在 OpenKnowledge 里，前端的复杂度并不只是组件嵌套，而是状态很重。DocumentContext 很关键：

- active doc / active tab
- open tabs
- navigation target
- sync status
- document transition

这意味着：

- 当前文档不是简单的 `useState` 变量
- tab 生命周期、hash 路由和文件导航是耦合的
- 某些特性如“skill tabs / preview tabs / large-file guard”都走这条状态链

你要重点阅读：

- `packages/app/src/editor/DocumentContext.tsx`
- `packages/app/src/components/EditorPane.tsx`
- `packages/app/src/editor/editor-tabs.ts`
- `packages/app/src/components/navigation-targets.ts`

## 4. 前端的核心设计：hash 导航而非 page router

OpenKnowledge 的前端不是传统的 `react-router` 页面路由，而是更像“文档工作台路由”。它依赖 hash 来表达当前目标：

- 文档页
- folder index
- skill 文件
- asset
- settings
- dialog overlay

关键文件：

- `packages/app/src/lib/doc-hash.ts`
- `packages/app/src/components/navigation-targets.ts`

这里非常适合你去理解：

- “打开一个文档”是不是简单 push route，而是同步给 tab state / 文档状态机
- “activate document / switch tab / history back” 全部在同一机制内

这类设计非常典型：OpenKnowledge 把“编辑器本体”当成中心，而不是把页面当成导航目标。

## 5. UI 层重要的几个入口组件

### `FileSidebar`

负责文件树、导航、 open state、目录关系。

### `EditorPane`

负责编辑区核心渲染与编辑器状态。

### `CommandPalette`

这个项目中命令面板不是简单快捷键面板，它是一个很重要的 command-dispatch 层。

### `SystemDocSubscriber`

用于 system doc、agent-driven state、skills 和自动同步的订阅。

### `TerminalLaunchProvider` / `handoff`

说明前端不仅是 markdown 编辑器，还有与 terminal / agent / tool path 的连接逻辑。

## 6. 前端如何和后端对接

前端通过多个层次和 `server` 连接：

- fetch / API requests
- WebSocket / collab routes
- local bridges / desktop IPC
- config provider and resource discovery

关键点：

- 本地应用和桌面应用共享一个前端构建
- 运行环境可能是 browser / electron / dev server
- App 层本身不仅和 API 对接，还要知道 desktop bridge / config mode / single-file mode 等上下文

## 7. 你应该关注的设计能力

对于 React 开发者，OpenKnowledge 的前端最值得学习的，不是“某个按钮怎么写”，而是：

- how to build a document-centric app shell
- how to manage open tabs and navigation without classic router
- how to treat files as first-class state
- how to support preview / skills / config overlay / agent subscriptions within one shell

这几处是该项目最有代表性的 UI 设计。

## 8. 阅读建议：以“状态机”视角看前端

OpenKnowledge 的 App 层比较难的地方，不在于组件很多，而在于状态很复杂。建议你在读代码时用一个问题：

> 这段代码到底改变了哪个状态？

例如：

- 切换 tab 改变什么 state？
- hash 导航影响哪些存储和 open target？
- `DocumentContext` 里 active document 与 preview 是否分离？
- `ConfigProvider` 里的 merge config 对哪些 UI 产生影响？

你如果能把状态来源和状态传播链条看明白，就基本理解 App 层了。

## 9. 最小阅读清单

建议先看下面这些：

- `packages/app/src/App.tsx`
- `packages/app/src/main.tsx`
- `packages/app/src/editor/DocumentContext.tsx`
- `packages/app/src/lib/doc-hash.ts`
- `packages/app/src/components/EditorPane.tsx`
- `packages/app/src/components/FileSidebar.tsx`

如果你还想继续深入：

- `packages/app/src/editor/editor-tabs.ts`
- `packages/app/src/components/navigation-targets.ts`
- `packages/app/src/lib/config-provider.tsx`
- `packages/app/src/lib/single-file-mode.tsx`

## 10. 结论

React UI 层是最容易进入的门，但它并不是“只是页面”，而是一个高度状态驱动的编辑器工作台。它是整个 OpenKnowledge 的“体验接口”，而真正难的一部分，往往在底层 server / watcher / persistence / git 层。

下一步你应该去看：

- [03-core-server-runtime.md](./03-core-server-runtime.md)
