# ok init：项目初始化与编辑器接入

## 1. 命令定位

`ok init` 是 OpenKnowledge 的“初始化入口”，它的职责不是启动编辑器，而是把一个目录变成一个已准备好的 OpenKnowledge 项目。

从实现上看，它主要负责两大类工作：

1. 让当前目录具备项目结构，例如 `.ok/`、配置、内容目录等。
2. 把 OpenKnowledge 的 MCP/skill 配置写入本机已检测到的编辑器中，让 Claude、Cursor、Codex 等工具能够识别和调用本地的 OpenKnowledge 服务。

对应代码入口：

- `packages/cli/src/commands/init.ts`
- `packages/cli/src/integrations/resolve-project-root.ts`
- `packages/cli/src/integrations/write-project-skill.ts`
- `packages/cli/src/commands/editors.ts`

---

## 2. 它解决的核心问题

在 OpenKnowledge 里，很多功能并不是“裸打开一个目录就能用”，而是依赖：

- `.ok/` 目录存在
- `.ok/config.yml` 配置已生成
- 内容目录已确认
- Git 工作区已准备
- 编辑器的 MCP 配置已写入
- project skill / agent integration 也已安装

因此 `ok init` 更像是“项目装配”命令，而不是单纯的启动命令。

---

## 3. 主要作用

### 3.1 确认和解析项目根目录

`runInit()` 一开始会做项目根解析：

- 先根据当前工作目录解析项目根目录
- 如果当前目录在 Git 仓库下，可能会向上查找或提升到 Git root
- 如果是在子目录中运行，也会处理 “promote to repo root” 的场景

也就是说，`ok init` 会尽量把一个子目录视为该 Git 仓库的一部分，而不是生成多个独立的 `.ok/`。

这部分逻辑很重要：

- 一个 Git 仓库通常对应一个 OpenKnowledge 项目根
- `.ok/` 的位置通常是 repo 级别的，而不是任意嵌套目录级别
- 这样后续 `ok start`、`ok sync`、MCP 访问等行为都能统一到同一项目语义上

---

### 3.2 生成项目结构

`initContent()` 会在项目根下创建 `.ok/` 和必要的配置文件，如：

- `.ok/config.yml`
- 内容目录配置
- 共享的 project metadata

这里的关键点是：

- 这是“幂等”的：重复执行不会无意义地覆盖已有配置
- 它会处理 `.ok/` 是否已存在、内容目录是否已设定、是否需要升级/补全
- 它也会根据项目路径和 Git 状态决定 `content.dir` 或内容范围

---

### 3.3 确保 Git 环境就绪

`ok init` 会调用 `ensureProjectGit(projectRoot)`，保证项目具备 Git 可用性：

- 若目录还没初始化 Git，就执行 `git init`
- 若 Git 不可用或版本过低，会抛出异常并中断初始化

这说明 OpenKnowledge 把 Git 当成系统底层能力，而不是可选附加品。

它并不是为了“编辑器用一下”，而是为了后续：

- 文档同步
- GitHub 共享
- 版本管理
- shadow repo / project state
- 备份和恢复

---

### 3.4 处理 project skill 与 gitignore

在初始化的中段，它还会：

- 写入 project skill 的 `.gitignore` 规则
- 去掉已经跟踪的 project skill projection
- 把工程内的 skill 运行产物保持为本地-only，而不是被误提交

这块设计很典型：

- OpenKnowledge 把 AI skill bundle 看成“本地构建产物”
- 它不希望这些内容被提交到仓库里
- 因而初始化过程会做清理和屏蔽，不让开发者把机器相关产物混入源代码

---

### 3.5 写入编辑器 MCP 配置

这是 `ok init` 最有代表性的能力：

它会扫描当前机器上已经安装的编辑器和 AI harness，并把 OpenKnowledge 的 MCP 配置写入到目标配置文件中。

对应逻辑：

- `detectInstalledEditors()`
- `resolveEditorTargets()`
- `writeEditorMcpConfig()`
- `writeUserMcpConfigs()`

它写入的目标包括：

- Claude
- Claude Desktop
- Cursor
- Codex
- GitHub Copilot
- OpenCode
- 其他受支持的编辑器

它的设计特点是：

- 只写“已检测到且支持的编辑器”
- 不强制为所有工具创建新配置目录
- 能保持既有配置结构，不破坏用户自己的本地编辑器配置
- 对 JSON/TOML/YAML 配置做了“surgical upsert”，尽量只修改 OK 自己的条目

这也体现了它作为“接入层写入器”的特征：

- 不是仅仅创建一个目录
- 而是让各个 AI 编辑器认识 OpenKnowledge，并把调用链接到本地服务

---

## 4. 处理流程概览

`ok init` 的整体执行顺序可以概括为：

1. 解析当前目录和项目根
2. 确定内容目录的作用域
3. 确认/初始化 Git
4. 扫描并创建 `.ok/`/配置
5. 设置 `.gitignore` 和 project skill 隔离
6. 判断要安装到哪个 editor target
7. 写入 MCP 配置和 skill bundles
8. 输出初始化摘要，告诉用户当前已准备好的内容和编辑器结果

这是一条典型的“项目准备 → 环境接线 → AI 能力接入”的流程。

---

## 5. 关键理解

如果你要理解 `ok init`，最重要的一点是：

它不是“启动程序”，而是“把当前目录变成一个可用的 OpenKnowledge 项目”。

它的核心价值在于：

- 让项目具备 `.ok/` 语义
- 让配置和内容目录清晰化
- 让编辑器知道 OpenKnowledge 这个本地服务
- 让 AI 客户端能通过 MCP/skill 访问知识库

---

## 6. 适合学习时的切入点

最值得看的几个位置：

- `runInit()`：总入口
- `initContent()`：项目结构创建
- `ensureProjectGit()`：Git 初始化
- `writeEditorMcpConfig()`：编辑器接入写入
- `resolveProjectRoot()`：项目根解析

如果你想继续深入，下一步最自然的阅读顺序是：

1. 先读 `runInit()`
2. 再看 `initContent()`
3. 然后读 `writeEditorMcpConfig()`
4. 最后看 `resolveProjectRoot()` 和 `detectInstalledEditors()`

---

## 7. 一句话总结

`ok init` 是“项目装配命令”：它负责把一个普通目录变成一个具备 OpenKnowledge 配置、Git 语义、编辑器接入和 AI toolchain 的工作区。
