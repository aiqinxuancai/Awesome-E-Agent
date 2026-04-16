# 易语言 × AI Agent 实践白皮书

> **AI 不懂易语言？让它懂。不想学？让 AI 替你写。不想用了？让 AI 帮你跑路。**

---

## 前言：被 AI 时代遗忘的语言？

易语言，一门用中文写代码的 Windows 编程语言，承载了无数开发者最初的编程记忆，也沉淀着大量真实跑在生产环境的存量代码。

但 AI 编程的浪潮滚滚而来——GitHub Copilot、Claude Code、Codex、Gemini CLI……这些工具几乎对易语言一无所知。它们不认识"子程序"，看不懂".ec 模块"，更别提帮你修 bug 或写新功能。

**易语言开发者，仿佛站在了 AI 时代的门外。**

直到现在。

本文提供三条路径，覆盖「继续用易语言」到「彻底离开易语言」的完整谱系。你可以选择任何一条，也可以逐步演进。

---

## 路径一：拥抱 AI，留在 IDE — AutoLinker 方案

> 适合：**不想改变工作流，只想让 AI 帮我写代码的开发者**

### 核心理念：让 AI 住进易语言 IDE

[AutoLinker](https://github.com/aiqinxuancai/AutoLinker) 是一个易语言支持库插件。安装后，它从两个方向同时打通了易语言与 AI 的通道：

- **向内**：在 IDE 侧边栏植入一个 AI 对话面板，直接与大模型对话，让 AI 实时读写你的工程代码
- **向外**：启动本地 MCP 服务，让 Claude Code、Codex、Gemini CLI 等终端 Agent 接入易语言 IDE，像操作普通代码仓库一样操作 `.e` 工程

**安装方法**：编译后将 `AutoLinker.fne` 放入易语言 `lib` 目录，在 IDE 中启用支持库即可。

---

### 内嵌 AI 对话面板：不离开 IDE 的完整 Agent 体验

插件加载后，易语言 IDE 左侧会出现一个新的 **AI 对话页签**。这不是简单的补全工具，而是一个具备工程感知能力的 Agent 界面——它知道你当前打开的是哪个页面、工程里有哪些子程序、引用了哪些支持库。

**配置方式**：在插件设置中填入以下信息即可接入任意兼容 OpenAI 接口的模型：

| 配置项 | 说明 | 示例 |
|--------|------|------|
| Base URL | 模型服务的 API 地址 | `https://api.anthropic.com` |
| API Key | 对应服务的密钥 | `sk-ant-...` |
| 模型 ID | 使用的具体模型 | `claude-opus-4-5` / `gpt-4o` / `gemini-2.5-pro` |

国内中转、自建代理、或任何兼容 OpenAI 格式的第三方服务均可接入。

**能做什么**：在对话框里用自然语言描述需求，AI 会调用内置工具读取你的工程源码，直接完成修改并写回 IDE，无需你手动复制粘贴任何内容。

> 💬 "帮我给所有网络请求子程序加上超时处理，编译通过为止。"

**第二步：接入外部 AI Agent（可选）**

如果你更习惯在终端里使用 Claude Code 或 Codex，AutoLinker 同时提供本地 MCP 服务。启动后，输出窗口会打印：

```
[AutoLinker][LocalMCP] listening on http://127.0.0.1:19207/mcp
```

<table>
<tr><th>工具</th><th>配置文件</th><th>配置内容</th></tr>
<tr>
<td>Claude Code</td>
<td><code>~/.claude.json</code></td>
<td>

```json
{
  "mcpServers": {
    "AutoLinker": {
      "transport": "streamable_http",
      "url": "http://127.0.0.1:19207/mcp"
    }
  }
}
```

</td>
</tr>
<tr>
<td>Codex</td>
<td><code>~/.codex/config.toml</code></td>
<td>

```toml
[mcp_servers.AutoLinker]
url = "http://127.0.0.1:19207/mcp"
```

</td>
</tr>
<tr>
<td>Gemini CLI</td>
<td><code>~/.gemini/settings.json</code></td>
<td>

```json
{
  "mcpServers": {
    "AutoLinker": {
      "transport": "streamable_http",
      "url": "http://127.0.0.1:19207/mcp"
    }
  }
}
```

</td>
</tr>
</table>

配置完成后，在终端运行 Claude Code 或 Codex，即可直接读取易语言工程源码、搜索支持库，并将修改写回 IDE。

### MCP 工具集：AI Agent 能做什么？

AutoLinker 向外部 Agent 暴露了一套完整的工具集：

| 能力 | 工具 |
|------|------|
| 读取当前页完整源码 | `get_current_page_code` |
| 搜索整个工程源码 | `search_public_code` |
| 列出并搜索支持库声明 | `search_support_library_public_code` |
| 搜索模块公开接口 | `search_available_module_public_code` |
| 精确编辑任意程序树页面 | `edit_program_item_code` |
| 批量修改多个页面 | `multi_edit_program_item_code` |
| 触发编译并获取结果 | `compile_with_output_path` |
| 执行 PowerShell 命令 | `run_powershell_command` |
| 联网搜索参考资料 | `search_web_tavily` |

这意味着，你可以在终端对 Claude Code 说：

> "帮我给这个工程所有子程序添加错误处理，编译通过为止。"

然后去喝杯茶。

### 项目规范文件：让 AI 了解你的项目

在 `.e` 文件同目录创建同名 `.AGENTS.md`（如 `MyApp.AGENTS.md`），AutoLinker 会自动将其注入所有 AI 功能的系统提示词。

```markdown
# 项目规范

## 技术背景
本项目运行于 Windows XP+，使用 VC2017 链接器。

## 命名约定
- 子程序名使用中文，格式：`动词_名词`
- 局部变量使用小驼峰英文命名

## 禁止事项
- 不得引入新的第三方支持库
- 不得修改 `_启动子程序`
```

这一机制与 Claude Code 的 `CLAUDE.md`、Codex 的 `AGENTS.md` 如出一辙——**你的 AI，从此懂你的项目**。

### 内嵌右键菜单 AI 功能

AutoLinker 还在 IDE 右键菜单直接提供 AI 能力，适合快速的单次操作，无需打开对话面板：

- **AI 优化函数**：对当前函数做等价重构与性能优化
- **AI 添加注释**：一键为函数补全文档注释
- **AI 翻译变量名**：将中文函数名、参数名重命名为英文 `lowerCamelCase`
- **AI 按上下文添加代码**：根据当前页类型与你的描述，直接生成并插入代码

---

## 路径二：抛弃 IDE，AI 直接开发 .e 项目 — e-packager 方案

> 适合：**易语言存量项目，希望用最强 AI Agent 全力接管开发的团队**

### 核心理念：把 .e 文件"解封印"

易语言的 `.e` 文件是二进制专有格式，对任何外部工具都是黑盒。[e-packager](https://github.com/aiqinxuancai/e-packager) 打破了这个封印——

**将 `.e` 文件解包为标准可读目录，让 Git、Diff、AI Agent 直接操作源码文件，修改完成后再回包为 `.e`。**

```
MyApp.e  →  e-packager unpack  →  MyApp/
                                  ├── src/          ← 源码 .txt 文件 + 窗口 .xml
                                  ├── project/      ← 元数据（勿删）
                                  ├── image/        ← 图片资源
                                  ├── audio/        ← 音频资源
                                  ├── AGENTS.md     ← AI 专属项目说明（自动生成）
                                  └── info.json     ← 文件元信息
```

### 操作流程

**解包**

```bash
# 方式一：命令行
e-packager unpack MyApp.e MyApp/

# 方式二：直接拖拽 .e 文件到 e-packager.exe，自动解包到同名目录
```

**用 AI Agent 开发**

解包后，用任意 AI Agent 直接操作 `src/` 下的文本源码：

```bash
# 进入项目目录，召唤 Claude Code
cd MyApp
claude

# 或者 Codex
codex

# 告诉 AI 你要做什么——它会读 AGENTS.md 了解项目结构，然后直接改文件
```

**回包**

```bash
# 方式一：命令行
e-packager pack MyApp/ MyApp_new.e

# 方式二：在项目目录内直接运行，自动输出到 pack/ 目录
cd MyApp
e-packager
```

### 这条路径的意义

| 传统方式 | e-packager 方式 |
|----------|----------------|
| 易语言 IDE 独占，无法用 Git 管理 | 源码变为文本，完整 Git 版本控制 |
| AI 看不懂 .e 文件 | AI 直接读写 .txt 源码文件 |
| Code Review 靠截图 | `git diff` 清晰展示每一行变更 |
| 多人协作靠复制文件 | 标准 Git 分支与合并工作流 |

**你保留了易语言的全部技术积累，同时获得了现代工程的全部红利。**

---

## 路径三：彻底告别易语言 — AI 驱动的语言迁移方案

> 适合：**不再维护易语言，希望将存量代码迁移至现代语言的团队**

### 核心理念：用 AI 做翻译官

你不需要手动重写。你只需要：

1. **用 e-packager 解包**，获得可读的文本源码
2. **把源码交给 AI**，让它理解逻辑
3. **让 AI 重写**为 C#、C++、Python 或任何你想要的语言

### 操作示例

**第一步：解包源码**

```bash
e-packager unpack LegacyApp.e LegacyApp/
```

**第二步：召唤 AI，开始迁移**

```bash
cd LegacyApp
claude  # 或 codex
```

然后给出指令：

> "这是一个易语言项目，AGENTS.md 里有完整的项目结构说明。请将 src/ 下的所有业务逻辑完整重写为 C# (.NET 10)，保留所有功能，重写为符合现代 C# 规范的项目结构。"

Claude Code 会：
- 读取 `AGENTS.md` 理解项目整体结构
- 逐一读取 `src/` 下的源码文件理解每个模块的逻辑
- 生成完整的 C# 项目，包括类结构、方法、异常处理
- 遇到不确定的地方主动询问

**第三步：Review + 修正**

迁移后的代码是标准现代语言，可以用 VSCode、Visual Studio、Rider 等任何工具进行 Code Review 和修正，也可以继续让 AI 迭代完善。

### 为什么 AI 能做好这件事？

e-packager 解包后的易语言源码是**结构化的文本格式**，具备清晰的子程序边界、变量声明和控制流。对于现代大语言模型而言，理解并翻译这类代码是其最擅长的任务之一——

**它可能是你见过的最便宜的遗留代码迁移项目。**

---

## 三条路径对比

| | 路径一：AutoLinker | 路径二：e-packager + AI 开发 | 路径三：AI 语言迁移 |
|--|--------------------------|------------------------------|-------------------|
| **目标** | 在易语言 IDE 内用 AI 开发 | 抛弃 IDE，AI 直接维护 .e 项目 | 彻底迁移至现代语言 |
| **保留易语言** | ✅ 是 | ✅ 是（源码格式变化） | ❌ 否 |
| **所需工具** | AutoLinker.fne + Claude Code 等 | e-packager + 任意 AI Agent | e-packager + 任意 AI Agent |
| **适合场景** | 日常开发提效 | 团队协作、大型项目 | 技术栈升级、摆脱历史包袱 |
| **上手难度** | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **收益幅度** | 开发效率大幅提升 | 工程能力质的飞跃 | 完全现代化 |

---

## 结语：时代的门，从来没有关上

易语言的开发者从不缺乏实干精神——用一门"小众"语言，造出了无数真实运行的程序。

现在，AI 的浪潮不是威胁，而是机遇。

**你可以让 AI 懂易语言，也可以让易语言的遗产在 AI 的帮助下涅槃重生。**

三条路，总有一条适合你。

---

## 相关资源

- 🔧 [AutoLinker — 易语言 AI Agent IDE 插件](https://github.com/aiqinxuancai/AutoLinker)
- 📦 [e-packager — 易语言 .e 文件解包/封包工具](https://github.com/aiqinxuancai/e-packager)
