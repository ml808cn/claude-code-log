# Claude Code Log

> 本文是 [README.md](README.md) 的中文译本。

一个 Python 命令行工具，用于将 Claude Code 转录记录（JSONL 文件）转换为可读的 HTML 和 Markdown 格式。

浏览器日志演示：

[Browser log](https://github.com/user-attachments/assets/12d94faf-6901-4429-b4e6-ea5f102d0c1c)

TUI 演示：

[TUI](https://github.com/user-attachments/assets/75718e2b-3b02-4e17-8f3d-366e2c40dcc2)

## 项目概览

📋 **[查看更新日志](CHANGELOG.md)** - 了解每个版本的新增内容

该工具生成简洁、极简的 HTML 页面，按时间顺序展示用户的提问与助手的回复。它旨在为你的 Claude Code 交互创建一份可读的日志，同时支持单个文件和整个项目目录层级。

> [!NOTE]
> 实验性 provider 支持现已可用，可从 Antigravity CLI（`agy`，**alpha** 阶段）和 Codex CLI（`codex`，**beta** 阶段）导出单会话。用法为
> `--provider agy|codex --session-id <id>`；随着上游转录格式的演变，这些集成可能会发生变化。

📄 **[查看示例 HTML 输出](https://daaain.github.io/claude-code-log/example/)** - 一个取自本项目开发样例的真实示例，每次文档构建时都会重新生成

## 快速开始

太长不看：运行下面的命令，然后浏览从你整个 Claude Code 存档生成的页面：

```sh
uvx claude-code-log@latest --open-browser
```

## 主要特性

- **交互式 TUI（终端用户界面）**：通过实时导航、摘要以及导出 HTML 和恢复会话的快捷操作来浏览和管理 Claude Code 会话
- **监听模式（Watch Mode）**：`claude-code-log watch` 会在会话写入的同时重新转换，让编辑器或 Obsidian 中的 Markdown 保持最新；`claude-code-log serve --watch` 会让打开的会话页面随着消息到达而不断增长，同时保持你的滚动位置和折叠的章节
- **项目目录层级处理**：处理整个 `~/.claude/projects/` 目录，并生成带链接的索引页
- **单独会话文件**：为每个会话生成独立的 HTML 文件，并带有导航链接
- **单个文件或目录处理**：转换单个 JSONL 文件或指定的目录
- **会话导航**：带会话摘要和快捷导航的交互式目录
- **Token 用量跟踪**：显示单条消息和会话总量的 token 消耗
- **运行时消息过滤**：由 JavaScript 驱动的过滤功能，用于显示/隐藏消息类型（用户、助手、系统、工具调用等）
- **按时间排序**：所有消息跨会话按时间戳排序
- **交互式时间线**：生成按消息时间分组的可缩放交互式时间线，以便直观地浏览对话
- **跨会话摘要匹配**：正确地将异步生成的摘要匹配到其原始会话
- **日期范围过滤**：使用自然语言按日期范围过滤消息（例如 "today"、"yesterday"、"last week"）
- **丰富的消息类型**：支持用户/助手消息、工具调用/结果、思考内容、图片
- **系统命令可见性**：以可展开的详情展示系统命令（如 `init`），并进行结构化解析
- **Markdown 渲染**：使用 wenmode 进行服务端 Markdown 渲染并带语法高亮
- **详情级别与紧凑模式**：`--detail full|high|low|minimal|user-only` 按详细程度过滤，`--compact` 合并重复出现的章节标题——与 `--format md` 搭配使用，可将过往对话回传给 LLM 进行分析或构建经验
- **悬浮导航**：始终可用的返回顶部按钮和过滤控件
- **CLI 接口**：使用 Click 构建的简洁命令行工具

## 该工具解决了哪些问题？

该工具可帮助你回答如下问题：

- **"我该如何回顾我所有的 Claude Code 对话？"**
- **"我昨天/上周和 Claude 一起做了什么？"**
- **"我的 Claude Code 会话花了多少成本？"**
- **"我该如何搜索我整个 Claude Code 历史记录？"**
- **"Claude 在这个项目中使用了哪些工具？"**
- **"我该如何与他人分享我的 Claude Code 对话？"**
- **"我的项目开发时间线是怎样的？"**
- **"我该如何分析我的 Claude Code 使用模式？"**
- **"我该如何将过去的会话回传给 LLM 进行分析或构建经验？"**

## 使用方法

### 交互式 TUI（终端用户界面）

TUI 提供了一个交互式界面，用于浏览和管理 Claude Code 会话，支持实时导航、会话摘要和快捷操作。

```bash
# 为所有项目启动 TUI（默认行为）
claude-code-log --tui

# 为指定项目目录启动 TUI
claude-code-log /path/to/project --tui

# 为指定的 Claude 项目启动 TUI
claude-code-log my-project --tui  # 自动转换为 ~/.claude/projects/-path-to-my-project
```

**TUI 功能：**

- **会话列表**：交互式表格，显示会话 ID、摘要、时间戳、消息数量和 token 用量
- **智能摘要**：优先使用 Claude 生成的摘要而非首条用户消息，以便更好地识别会话
- **工作目录匹配**：自动查找并打开与当前工作目录匹配的项目
- **快捷操作**：
  - `h`：生成会话 HTML 并在浏览器中打开
  - `m`：生成会话 Markdown 并在浏览器中打开
  - `v`：在嵌入式查看器中查看会话 Markdown（带目录）
  - `c`：使用 `claude -r <sessionId>` 在 Claude Code 中恢复会话
  - `r`：从文件重新加载会话数据
  - `p`：切换到项目选择器视图
  - `H`/`M`/`V`：强制重新生成 HTML/Markdown（用于开发的隐藏快捷键）
- **项目统计**：实时显示会话总数、消息数、token 数和日期范围
- **缓存集成**：利用现有缓存系统实现快速加载，并自动校验缓存
- **键盘导航**：方向键导航，Enter 展开行详情，`q` 退出
- **行展开**：按 Enter 展开所选行，显示完整摘要、首条用户消息、工作目录和详细的 token 用量

### 默认行为（处理所有项目）

```bash
# 处理 ~/.claude/projects/ 中的所有项目（默认行为）
claude-code-log

# 显式处理所有项目
claude-code-log --all-projects

# 处理所有项目并在浏览器中打开
claude-code-log --open-browser

# 带日期过滤地处理所有项目
claude-code-log --from-date "yesterday" --to-date "today"
claude-code-log --from-date "last week"

# 跳过单独的会话文件（仅创建合并后的转录记录）
claude-code-log --no-individual-sessions
```

这会生成：

- `~/.claude/projects/index.html` - 顶层索引，带项目卡片与统计信息
- `~/.claude/projects/project-name/combined_transcripts.html` - 各项目合并转录页（这些文件可能达到数 MB）
- `~/.claude/projects/project-name/session-{session-id}.html` - 单独会话页面
- `~/.claude/projects/project-name/session-{session-id}.md` - Markdown 版本（通过 TUI 按需生成）

### 单个文件或目录处理

```bash
# 单个文件
claude-code-log transcript.jsonl

# 指定目录
claude-code-log /path/to/transcript/directory

# 自定义输出位置
claude-code-log /path/to/directory -o combined_transcripts.html

# 转换后在浏览器中打开
claude-code-log /path/to/directory --open-browser

# 按日期范围过滤（支持自然语言）
claude-code-log /path/to/directory --from-date "yesterday" --to-date "today"
claude-code-log /path/to/directory --from-date "3 days ago" --to-date "yesterday"
```

### 将过往对话回传给 LLM

`--detail low --format md --compact` 这一组合会生成适合作为 LLM 上下文的精简 Markdown，用于审阅或从过往工作中提炼模式：

```bash
# 会话 → 供 LLM 审阅的精简 Markdown
claude-code-log transcript.jsonl --detail low --format md --compact -o session.md

# 整个项目历史
claude-code-log /path/to/project --detail low --format md --compact
```

`--detail` 级别（输出由小到大）：

- `user-only` — 仅用户提问和引导（适合作为下游 agent 的输入，例如构建需求文档）
- `minimal` — 仅用户 + 助手文本
- `low` — 以交互为主；保留 WebSearch、WebFetch 和 Task（agent 委派）作为关键信号
- `high` — 详细但已清理；丢弃系统/钩子噪音
- `full` — 全部内容（默认）

`--compact` 会合并 Markdown 中连续的同类章节，使连续的助手回复共用一个标题，而不是每条都重复 `### 🤖 Assistant:`。

### 关联提交 SHA

转录正文中形如 `7c2e6f6` 的普通 token，若该 SHA 能从本地远程跟踪分支到达，则会被转为可点击的提交链接。**github.com**、**gitlab.com** 和 **bitbucket.org** 开箱即用。对于自托管代码托管平台（自建 GitLab、Gitea、Forgejo 等），可通过 `--git-link` 提供 URL 模板：

```bash
# 自托管 GitLab
claude-code-log /path/to/transcript --git-link 'https://{host}/{path}/-/commit/{sha}'

# 通过环境变量实现同样效果（适用于 TUI / 重复调用）
export CLAUDE_CODE_LOG_GIT_LINK='https://{host}/{path}/-/commit/{sha}'
claude-code-log --tui
```

占位符：`{host}`、`{path}`、`{sha}`。只有当静态映射表尚未识别该 host 时模板才会生效，因此混合使用 GitHub 仓库与自托管 GitLab 时会从两者得到正确的链接。无法从任何本地远程跟踪 ref 到达的 SHA 会渲染为纯文本——仅本地存在的工作进度提交永远不会产生坏链。

## 项目目录层级输出

处理所有项目时，该工具会生成：

```sh
~/.claude/projects/
├── index.html                           # 顶层索引，带项目卡片
├── project1/
│   ├── combined_transcripts.html        # 合并的项目页面
│   ├── session-{session-id}.html        # 单独的会话页面
│   ├── session-{session-id}.md          # Markdown 版本（通过 TUI 按需生成）
│   └── session-{session-id2}.html       # 更多会话页面……
├── project2/
│   ├── combined_transcripts.html
│   └── session-{session-id}.html
└── ...
```

### 索引页功能

- **项目卡片**：每个项目显示为带有统计信息的可点击卡片
- **会话导航**：可展开的会话列表，带摘要和对单独会话文件的快捷访问
- **汇总统计**：项目总数、转录文件数和消息数量，以及 token 用量
- **最近活动**：按最后修改日期排序的项目
- **快捷导航**：一键访问合并转录页或单独会话
- **简洁的 URL**：由目录名转换而来的可读项目名称

## 支持的消息类型

- **用户消息**：常规用户输入和提问
- **助手消息**：Claude 的回复，带 token 用量显示
- **摘要消息**：带跨会话匹配的会话摘要
- **系统命令**：如 `init` 之类的命令，以可展开详情展示，并进行结构化解析
- **工具调用**：带可折叠详情的工具调用，以及特殊的 TodoWrite 渲染
- **工具结果**：带错误处理的工具执行结果
- **思考内容**：Claude 的内部推理过程
- **图片**：粘贴的图片和截图

## HTML 输出特性

- **响应式设计**：适用于桌面端和移动端
- **运行时消息过滤**：JavaScript 控件显示/隐藏消息类型，并带实时计数
- **会话导航**：带会话摘要和时间戳范围的交互式目录
- **Token 用量显示**：单条消息和会话级别的 token 消耗跟踪
- **语法高亮**：代码块经 Markdown 渲染后得到正确格式化
- **Markdown 支持**：使用 wenmode 进行服务端渲染，包括：
  - 标题、列表、强调、删除线
  - 代码块和行内代码
  - 链接、图片和表格
  - GitHub Flavored Markdown 特性
- **可折叠内容**：工具调用、系统命令和长内容位于可展开章节中
- **悬浮控件**：始终可用的过滤按钮、详情开关和返回顶部导航
- **跨会话特性**：摘要正确匹配到各个异步会话

## Markdown 输出特性

Markdown 导出为 HTML 提供了一种轻量、可移植的替代方案：

- **GitHub-Flavored Markdown**：兼容 GitHub、GitLab 和其他 Markdown 渲染器
- **层级结构**：会话以标题和可折叠详情组织
- **消息摘录**：章节标题包含消息预览，便于快速导航
- **代码保留**：通过围栏代码块提供语法高亮提示
- **嵌入式查看器**：TUI 内置带目录的 Markdown 查看器
- **图片支持**：可配置的图片处理方式（占位符、内嵌 base64 或引用的文件）
- **`--compact` 模式**：合并连续的同类章节标题——与 `--detail low` 或 `minimal` 搭配最有用，因为工具剥离会产生连续的助手或用户章节

## 安装

推荐的方式是作为 `uv` 工具安装：

```bash
uv tool install claude-code-log
```

这会创建一个可直接调用的 `claude-code-log` 二进制文件。

或者直接用 `uvx` 运行（无需单独的安装步骤）：

```bash
uvx claude-code-log@latest
```

另外，如果你愿意，也可以使用 `pip` 安装：

```bash
pip install claude-code-log
```

或者从源码安装：

```bash
git clone https://github.com/daaain/claude-code-log.git
cd claude-code-log
uv sync
uv run claude-code-log
# or if you want `claude-code-log` binary that works everywhere:
uv tool install --editable .
```

## 贡献

请参阅 [CONTRIBUTING.md](CONTRIBUTING.md)，了解开发环境搭建、测试和架构文档。

## 社区扩展

基于 `claude-code-log` 构建的项目：

- **[archive-session](https://github.com/lifeinchords/claude-code-skills#archive-session-skill--slash-command--optional-hook)**，作者 [@lifeinchords](https://github.com/lifeinchords)。将 CLI 封装为三个集成界面：
  - 一个 Claude Code [Skill](https://github.com/lifeinchords/claude-code-skills/blob/main/.claude/skills/archive-session/SKILL.md)
  - 一个 Claude Code 斜杠 [命令](https://github.com/lifeinchords/claude-code-skills/blob/main/.claude/commands/archive-session.md) `/archive-session`，用于在聊天中显式调用
  - 一个 Claude Code PreCompact [Hook](https://github.com/lifeinchords/claude-code-skills/blob/main/.claude/hooks/pre-compact-archive.sh)，在上下文压缩之前自动存档转录记录和子代理日志

跨平台（macOS 和 Windows/MSYS）。

## TODO

- 教程覆盖层
- 如果存在，是否集成 `claude-trace` 请求日志？
- 将图片转换为 WebP，因为截图往往是巨大的 PNG——反复重做可能很耗时（因此也需要某种缓存），并且需要带编译依赖的重型库（除非有快速的纯 Python 转换库？或者 WASM？）
- 为内置工具添加特殊格式：Glob、Grep、LS、MultiEdit、NotebookRead、NotebookEdit、WebFetch、TodoRead、WebSearch
- 添加类似 `ccusage` 的每日摘要，或许还可以基于 Claude 生成的会话摘要再添加一些文本摘要？
- 从 @claude GitHub Actions 导入日志
- 从 @claude GitHub Actions 流式传输日志，参见 [octotail](https://github.com/getbettr/octotail)
- 将 CLI 封装为 GitHub Action，在 Claude GitHub Action 之后运行并处理其[输出](https://github.com/anthropics/claude-code-base-action?tab=readme-ov-file#outputs)
- 将过滤后的用户消息回传给无头（headless）claude CLI，以从会话中提炼用户意图
- 在 Python（CLI）侧也进行消息类型过滤，而不仅仅在 UI 中
- 添加极简主题并使其支持浅色 + 深色；在花哨主题中动画化渐变背景
- 合并 git worktree 目录
