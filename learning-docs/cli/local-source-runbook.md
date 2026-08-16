# 本地源码项目中的 CLI 初始化与启动

## 1. 目标

本篇整理的是：在源码仓库中，不依赖全局安装包，而是直接用本地源码里的 CLI 入口来初始化项目并启动 OpenKnowledge。重点是把“源码运行”的步骤整理成一套可复现的流程，方便后续继续阅读 `ok init` 和 `ok start` 的实现。

这里的核心思路是：

- `ok init`：把普通目录变成 OpenKnowledge 项目，并写入编辑器 MCP 配置。
- `ok start`：启动项目级 Web UI + API + MCP + 协作服务。
- 源码模式：直接执行 `packages/cli/src/cli.ts`，而不是调用全局 `ok` 命令。

---

## 2. 先决条件

### 2.1 Node.js 24+

这个仓库要求 Node 24 或更高版本，且使用 pnpm 10 以上。

在 Windows 下最稳妥的方式是使用 `fnm` 或 `corepack`：

```powershell
# 先确认版本
node -v

# 如果还没启用 pnpm
corepack enable
corepack pnpm -v
```

如果你是用 `fnm`：

```powershell
fnm install 24
fnm use 24
fnm env --use-on-cd --shell cmd
```

> 关键点：`pnpm` 本身可能在当前 shell 中不可见，`corepack enable` 后通常就能正常工作。

### 2.2 Windows 环境：MSVC + Windows SDK

如果是在 Windows 上本地源码构建，尤其是 `native-config` 这类 Rust/N-API 模块，必须有：

- Visual Studio 2022 Build Tools
- MSVC C/C++ toolchain
- Windows 10/11 SDK
- `cl.exe` / `link.exe` 在 PATH 中

常见表现：

- `link.exe` not found
- `fatal error LNK1104` / `cannot open file 'kernel32.lib'`
- Rust 编译阶段失败

解决方法：在同一个终端中打开 “Developer Command Prompt for VS 2022”，或者将 MSVC 环境变量注入当前 shell。

---

## 3. 最稳妥的源码运行顺序

### 第一步：安装依赖

```powershell
cd C:/source-learning/open-knowledge
corepack enable
corepack pnpm install
```

这一步会安装工作区依赖和所有 package 的 node_modules。

### 第二步：必要时先构建

在很多源码运行场景中，最好先跑一次构建，确保 CLI 和前端 bundle 都已生成：

```powershell
cd C:/source-learning/open-knowledge
corepack pnpm run build
```

这个 repo 的 `build` 会走 turbo，生成各个 package 的产物。对于本地调试，`build` 通常比单独跑 `start` 更稳，因为 `ok start` 会依赖 bundled React shell / app dist 等资源。

> 如果你只是想快速启动并验证流程，CLI 本身也能直接用源码入口启动；但一个完整构建更稳妥。

---

## 4. 直接用源码 CLI 初始化项目

### 4.1 进入源码仓库

```powershell
cd C:/source-learning/open-knowledge
```

### 4.2 准备一个空目录作为 demo

```powershell
New-Item -ItemType Directory -Force -Path C:/tmp/ok-demo | Out-Null
```

### 4.3 运行源码里的 `init`

```powershell
corepack pnpm exec tsx .\packages\cli\src\cli.ts init --cwd C:/tmp/ok-demo
```

这条命令的关键点：

- 不是全局 `ok init`
- 而是直接调用当前源码下的 CLI 入口
- `--cwd` 指定要初始化的目标目录

### 4.4 可能出现的交互

运行 `init` 时，程序可能提示：

- “Where should the MCP server be configured?”
- 选项包括：
  - User-level
  - Project-level

这是正常行为。它在询问要把 MCP 配置写到：

- 当前用户级配置（适合给本机工具统一接入）
- 当前项目级配置（适合按项目隔离）

这就是 `ok init` 最核心的一步：把 OpenKnowledge 接入各个编辑器。

### 4.5 `init` 做了什么

从代码上看，`ok init` 不仅创建 `.ok/` 目录，还会：

- 解析项目根目录
- 初始化 Git（如果目录还没有 `git`）
- 创建 `.ok/config.yml` 等配置
- 安装项目技能和 `.gitignore` 处理
- 扫描本地已安装的编辑器
- 把 OpenKnowledge 的 MCP 配置写入这些编辑器

因此它更像是“项目装配”，而不是“打开 app”。

---

## 5. 直接用源码 CLI 启动项目

初始化完成后，就可以启动项目：

```powershell
corepack pnpm exec tsx .\packages\cli\src\cli.ts start --cwd C:/tmp/ok-demo --open
```

这里的含义是：

- `start`：启动项目服务器
- `--cwd`：指向已初始化的项目目录
- `--open`：在本地浏览器中自动打开 UI

如果启动成功，浏览器会打开编辑器界面；你就可以在页面中：

- 新建 `.md` 文件
- 编辑 markdown 内容
- 侧边栏浏览目录
- 以网页方式直接写文档

这正是 OpenKnowledge 作为“本地知识库 + 浏览器编辑器”的使用方式。

---

## 6. 源码模式和正式安装模式的区别

### 6.1 正式安装方式

用户通常是：

```bash
npm install -g @inkeep/open-knowledge
cd your-project
ok init
ok start --open
```

这是“已发布包”的使用方式。

### 6.2 源码运行方式

在本地源码仓库中，通常是：

```powershell
cd C:/source-learning/open-knowledge
corepack pnpm install
corepack pnpm run build
corepack pnpm exec tsx .\packages\cli\src\cli.ts init --cwd C:/tmp/ok-demo
corepack pnpm exec tsx .\packages\cli\src\cli.ts start --cwd C:/tmp/ok-demo --open
```

两者的逻辑是一致的，但源码模式更适合：

- 阅读 CLI 实现
- 调试本地行为
- 修改 package 逻辑后立即验证

---

## 7. 需要注意的环境细节

### 7.1 先启用 Corepack

在 Windows 环境中，很多人会遇到：

```powershell
pnpm: not recognized
```

这个通常意味着当前 shell 里没装好 pnpm；最直接的办法是：

```powershell
corepack enable
```

然后再用：

```powershell
corepack pnpm ...
```

### 7.2 需要 Node 24

如果 Node 版本太低，安装和运行都会失败，尤其是工作区脚本和 CI 约束会直接拒绝。

### 7.3 Windows 的 native build 依赖

在源码运行中，很多“看起来像 Node 问题”的错误，往往最终其实是：

- Rust toolchain 没装好
- Visual Studio 构建环境未加载
- Windows SDK 缺失

因此，真正的启动顺序应该是：

1. 安装 Node 24
2. 启用 pnpm/corepack
3. 安装 VS Build Tools + Windows SDK
4. 进入正确 shell / Developer Prompt
5. `pnpm install`
6. `pnpm run build`
7. `ok init`
8. `ok start --open`

---

## 8. 一句话总结

在本地源码项目中，最直接的运行方式不是“装一个全局包”，而是：

- 先用 `corepack pnpm install`
- 再用 `corepack pnpm exec tsx packages/cli/src/cli.ts init --cwd <project>` 初始化项目
- 然后再用 `corepack pnpm exec tsx packages/cli/src/cli.ts start --cwd <project> --open` 启动服务

这样就能在本地源码环境中直接跑出 OpenKnowledge 的浏览器编辑器，并在页面中创建和编辑 markdown 文件。

---

## 9. 适合继续阅读的代码入口

如果你要继续深入学习，建议按顺序看这些实现：

- `packages/cli/src/cli.ts`：CLI 入口和命令注册
- `packages/cli/src/commands/init.ts`：`ok init` 的核心逻辑
- `packages/cli/src/commands/start.ts`：`ok start` 的核心逻辑
- `packages/cli/src/integrations/resolve-project-root.ts`：项目根解析
- `packages/cli/src/integrations/write-project-skill.ts`：项目 skill 和 editor 接入

这样可以把“命令语义”与“代码实现”对应起来。
