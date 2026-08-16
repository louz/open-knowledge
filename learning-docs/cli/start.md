# ok start：启动本地协作服务与 Web UI

## 1. 命令定位

`ok start` 是 OpenKnowledge 的“运行入口”。它不是简单地打开一个页面，而是启动项目级的本地协作服务器，负责：

- 提供 HTTP / WebSocket 服务
- 承载项目内容访问
- 管理 UI shell
- 维护本地文档状态与数据流
- 让编辑器、浏览器、MCP 客户端都能连接到同一运行时

对应实现入口：

- `packages/cli/src/commands/start.ts`
- `packages/cli/src/commands/start.test.ts`
- `packages/cli/src/index.ts`

---

## 2. 它解决的核心问题

`ok init` 把项目准备好了，但项目真正“可用”还需要一个正在运行的服务实例。

`ok start` 的核心职责是：

- 读取项目配置
- 验证当前目录已经是已初始化项目
- 启动 OpenKnowledge 的 project server
- 提供 UI 和 API 服务
- 复用已有 server，避免重复启动
- 处理本地、远程、嵌入式启动场景

所以它更偏“运行时启动器”，而不是配置脚本。

---

## 3. 主要作用

### 3.1 先检查是否已初始化

`runStartCommand()` 的第一步并不是直接启动，而是先进行一系列前置校验：

- 当前目录是否已有 `.ok/`
- config 是否存在
- 若缺少初始化，抛出 `OkDirMissingError`
- 输出明确提示：`Run ok init first`

这很关键，因为 OpenKnowledge 设计上要求：

- 项目需要先初始化
- `start` 依赖初始化后的配置与目录结构
- 启动动作并不是走“自动新建项目”的策略

也就是说，`ok start` 假定项目已经具备运行条件。

---

### 3.2 解析运行时参数和配置叠加

`start.ts` 里有大量“配置解析”逻辑，例如：

- bind / host / port
- remote / externalUrl
- idle shutdown
- open browser
- only server / only ui
- single-file mode
- projectDir
- react shell dist dir

这里的设计重点是：

- CLI 参数、环境变量、配置文件会按优先级合并
- 远程访问和本地访问需要不同的安全策略
- 它不是简单地读一个字段，而是统一做“runtime config resolution”

尤其值得注意的是：

- `--remote` 是 deprecated alias，但它仍然会展开成 `server.externalUrl` 等正式配置项
- `allowExternal`、远程 bind、idleShutdown 等语义都受到严格控制
- 有些参数的优先级和合并顺序是明确设计过的

---

### 3.3 启动 project server

真正启动的核心是 `bootStartServer()`。

这个函数负责：

- 解析 server runtime config
- 确定本地监听地址和端口
- 建立 HTTP / WebSocket 服务
- 绑定 shell（React UI）与 API 路由
- 处理 `--only server` / `--only ui` 等分流模式
- 维持项目的 lock / 生命周期

这里的关键点是：

- 这是一个“组合式启动器”，而不是单一模块启动
- 它会把 UI、API、MCP、content 访问逻辑一起装配起来
- 启动后，OpenKnowledge 的多个入口会连接同一个 project runtime

---

### 3.4 复用已有 server，避免冲突

`ok start` 有一个关键行为：

- 如果同一个项目已经有一个 server 在运行，第二次 `ok start` 不会直接崩掉
- 它会尝试复用已有实例（spawn-or-reuse）
- 返回已有 server 的 URL 和状态

这主要依赖：

- `server.lock`
- `resolveServerReuse()`
- `isServerLockCollision()`

因此 `ok start` 是一个“启动/重用”命令，而不是“绝对强制重启”。

这对桌面应用、Node 服务热恢复，以及多个入口共存都很重要。

---

### 3.5 支持 UI / API / MCP 的统一访问模型

`ok start` 默认启动的不是仅仅一个编辑器 shell，而是一个完整的 runtime：

- `/`：提供 React UI
- `/api/*`：项目 API
- `/mcp`：MCP 入口
- `/collab`：协作服务
- content assets：本地文档和静态资源

也就是说，OpenKnowledge 把“编辑器界面”放在同一个 server 进程里，并让多个协议共享相同 runtime。对理解系统架构很关键：

- UI 不是独立进程
- API 不只是前端调后端
- MCP/协作能力与 core runtime 紧耦合

---

### 3.6 远程模式和外部暴露逻辑

`start.ts` 中有大量关于远程部署的逻辑，体现出 OpenKnowledge 不是只做本地桌面应用，而是支持外部访问场景。

例如：

- `--remote`
- `--external-url`
- `server.allowExternal`
- `OK_ALLOW_EXTERNAL`
- loopback / non-loopback bind 控制

这些代码的核心主题是：

- 远程访问必须显式打开
- 外部暴露默认不被静默接受
- 任何能够访问到的外部地址都被视为“有权限级影响”
- 因为它不仅暴露界面，还可能暴露 GitHub、发布、知识库操作能力

---

## 4. 处理流程概览

`ok start` 的大致执行顺序如下：

1. 读取当前环境和配置
2. 确认当前目录是已初始化项目
3. 解析 server bind / port / remote / shell / idle shutdown 等配置
4. 计算运行时配置并检查安全性约束
5. 调用 `bootStartServer()` 启动项目运行时
6. 如果已有 server 存在，复用已有实例
7. 启动 UI 和 API 接口
8. 绑定退出信号和 shutdown 逻辑
9. 维持服务器生命周期

---

## 5. 关键理解

如果说 `ok init` 是“项目装配”，那么 `ok start` 就是“运行时发动机”。

它的核心价值不只是“跑起来”，而是：

- 统一一套服务入口
- 连接 editor、browser、agent、MCP
- 提供本地协作和知识库运行时
- 让多个调用入口都落在同一个 project server 上

---

## 6. 适合学习时的切入点

最值得重点看的位置：

- `runStartCommand()`：总入口
- `bootStartServer()`：启动核心
- `resolveServerRuntimeConfig()`：配置决定
- `resolveCollabPort()`：端口解析
- `shouldOpenBrowser()`：浏览器打开策略
- `isServerLockCollision()` / `resolveServerReuse()`：复用逻辑

如果你继续深入学习，这些函数会比单纯看 UI 更有帮助，因为它们定义了系统的实际运行模型。

---

## 7. 一句话总结

`ok start` 是 OpenKnowledge 的“运行与协作入口”：它负责启动项目级服务器，把 UI、API、MCP、协作和文档运行时统一起来，并在必要时复用已有实例而不是重复启动。
