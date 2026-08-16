# 04. Desktop + CLI：Electron 壳与操作入口

## 1. 为什么这两层很重要

如果你已经看到了 UI 和 server 层，那么 OpenKnowledge 的“工程外壳”就很容易理解：

- `packages/desktop` 负责本地桌面应用运行时
- `packages/cli` 负责命令行启动与初始化

这两层做的事情不一样：

- Desktop 负责“原生启动流程、窗口、进程、native 集成”
- CLI 负责“人类操作入口、初始化、脚本式启动、agent/desktop dispatch”

## 2. Desktop：Electron main process 的核心

`packages/desktop/src/main/index.ts` 是桌面端最核心的入口。

这里负责：

- app 启动生命周期
- BrowserWindow 创建与管理
- project window / navigator window / note window 等各种类型窗口
- server 的 spawn / attach
- IPC 通信
- OS-level 配置和桌面 integration

它最关键的设计是：

> 一个 project window 对应一个 contentDir / 一个 server / 一个 utilityProcess。

这意味着：

- 每个项目都可以有自己的运行时
- app 可以创建多个项目窗口
- lifecycle 有明确的管理边界
- 需要处理嵌入式 collab server 和 app render window 的关系

## 3. `window-manager.ts`：桌面 shell 的中枢

`packages/desktop/src/main/window-manager.ts` 是另一个非常关键的文件。它控制：

- 窗口创建
- attach 模式和 spawn 模式
- project attach vs new spawn
- window focus / show / lifecycle
- project reuse / existing open guard

这一层很重要的概念是：

### attach 模式

如果一个项目的 server 已经存在，就不必重复启动，而是直接 attach 到已有实例。

### spawn 模式

如果没有现成 server，就启动新的 utility process / server，并为该 window 绑定其生命周期。

这类设计非常典型，体现了“桌面应用程序部件和本地 server 之间的边界分工”。

## 4. 为什么 Electron 这里很关键

对前端开发者来说，Electron 让人容易理解为“一个桌面壳 + 一个浏览器 renderer”。但 OpenKnowledge 的实现远不止如此：

- 它管理多个 project window
- 处理 native menu / project lifecycle
- 处理 local file system / project context
- 启动服务器进程
- bridge renderer 和 main process

因此，从架构上看，desktop 很像：

> 一个“运行时调度器 + 局部 native shell + App bootstrap 层”。

## 5. CLI：`ok` 是产品入口

`packages/cli/src/index.ts` 和 `packages/cli/src/commands/start.ts` 是 CLI 的关键入口。

`ok start` 负责：

- resolve project config
- identify bind host / port / runtime options
- decide if UI should be served
- boot the server
- maybe open browser
- onboard local integrations

这里非常值得注意的是：

- CLI 不是“单独的脚本”，而是和 desktop / server 共用一套底层能力
- 它的启动策略和 Desktop 的启动策略是相同的概念，只是入口不同

## 6. `ok` 与 Desktop 的关系

从系统设计看：

- Desktop = 本地桌面应用入口
- CLI = 开发者/本机运行入口
- 两者共享 server / core / app 能力

所以你会看到一类设计：

- same server runtime
- same project model
- same config semantics
- different shell surfaces

这也是 OpenKnowledge 能同时兼容“桌面端”和“本地 web UI”的关键原因。

## 7. 它为什么说“Bun / Node / Electron”都可能出现

OpenKnowledge 在不同场景中会面对：

- Bun-based development / tests
- Node runtime for CLI / server
- Electron runtime for desktop

你作为学习者不需要一开始掌握所有 runtime 细节，但要认识到：

- 这个项目不是只依赖一种 runtime
- 它把 runtime 视作“环境适配层”，而非业务逻辑本体
- 业务核心仍然在 `server` / `core` / `app` 这些层

## 8. 你需要从这个层获得什么理解

从 desktop + cli 层，你应该能理解：

- 应用如何从“用户操作”变成“server + window + project runtime”
- 为什么项目实例会有 spawn / attach 的区分
- 为什么它可以同时维护本地文件处理与桌面壳能力
- 为什么 CLI 和 desktop 是两个不同入口，但共享一套核心逻辑

## 9. 建议的最小阅读清单

重点看：

- `packages/desktop/src/main/index.ts`
- `packages/desktop/src/main/window-manager.ts`
- `packages/desktop/src/main/bootstrap.ts`
- `packages/cli/src/index.ts`
- `packages/cli/src/commands/start.ts`

你可以再补充：

- `packages/desktop/src/main/navigator-window.ts`
- `packages/desktop/src/main/create-new-project.ts`
- `packages/cli/src/commands/init.ts`

## 10. 总结

Desktop 和 CLI 让 OpenKnowledge 从“纯编辑器”变成“可落地的工具”。如果没有这两层：

- 没有桌面窗口
- 没有本地项目启动
- 没有 CLI 初始化过程
- 没有 OS-native bridge

那么它就只剩一个前端 demo，而不是一个真正的本地知识工作台。

下一步建议你深入看：

- [05-learning-plan.md](./05-learning-plan.md)
