---
layout: default
title: Reasonix Code 用户指南
---

# Reasonix Code 用户指南

> **Reasonix Code** — 一个基于工具的证据驱动型 AI 编程助手。不是聊天封装器，不是一个全栈 Agent 运行时。它是一个精确、严谨、审计友好的代码助手，帮你读文件、改代码、追 bug。

---

## 目录

- [快速开始](#快速开始)
- [心智模型](#心智模型)
- [核心能力速查](#核心能力速查)
- [编辑系统（SEARCH/REPLACE）](#编辑系统searchreplace)
- [规划与任务管理](#规划与任务管理)
- [记忆系统](#记忆系统)
- [技能系统（Skills）](#技能系统skills)
- [子 Agent 系统](#子-agent-系统)
- [MCP 集成](#mcp-集成)
- [身份自定义](#身份自定义)
- [编辑纪律（六条铁律）](#编辑纪律六条铁律)
- [常见问题 / 故障排除](#常见问题--故障排除)

---

## 快速开始

### 启动 Reasonix Code

```bash
# 在项目目录下启动
cd your-project
reasonix code
```

启动后，Reasonix 进入交互式对话循环。你可以：

- **读取文件** — `"帮我看看 src/main.ts"`
- **搜索代码** — `"所有用到 auth 的地方"`
- **修改代码** — `"把这个函数改成 async"`
- **探索架构** — `"这个项目的认证流程是怎样的"`
- **Web 调研** — `"查一下这个 API 的最新文档"`

### 快速上手示例

```text
你：这个项目用的什么测试框架？
Reasonix：通过 glob + read_file 查找配置文件...

你：把 utils.ts 里的 parseData 改成异步函数
Reasonix：read_file → edit_file（SEARCH/REPLACE）→ 完成

你：记住测试命令是 npm test
Reasonix：remember → 存入跨会话记忆

你：重构这个模块，需要动多个文件
Reasonix：提交 submit_plan → 你审批 → 按步骤执行
```

### 验证运行状态

```bash
# 检查工具是否可用
# 所有工具列在对话顶部的系统提示词中

# 查看可用技能列表
# 也在系统提示词的 Skills 索引中
```

---

## 心智模型

Reasonix Code 围绕一个**单次对话循环**构建，每次你发消息就触发一次。

```
你发送消息
     │
     ▼
┌──────────────────────────────────────┐
│ 1. 构建上下文                          │
│    系统提示词 + 持久记忆 + Skills      │
│    + 工具描述 → 形成完整 prompt        │
└──────────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ 2. 调用工具                           │
│    read_file / search_content /       │
│    run_command / web_search / ...     │
│    每个工具返回带证据的结果            │
└──────────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ 3. 生成响应                           │
│    基于工具证据 → 答案 + SEARCH/REPLACE│
│    + 行级引用（Cite or shut up）       │
└──────────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────┐
│ 4. 持久化（可选）                      │
│    remember → 存入跨会话长期记忆       │
└──────────────────────────────────────┘
     │
     ▼
    等待下一条消息
```

**核心原则**：**每个事实性声明必须有行级引用**。没有引用的断言会被拒绝。

---

## 核心能力速查

| 能力 | 工具 | 说明 |
|------|------|------|
| **读文件** | `read_file` | 全文读取（≤64KB）或自动大纲模式（更大文件） |
| **写文件** | `write_file` | 创建新文件或完全覆盖 |
| **编辑文件** | `edit_file` / `multi_edit` | SEARCH/REPLACE 模式，精确匹配 |
| **按内容搜索** | `search_content` | grep 文件内容，支持上下文行 |
| **按文件名搜索** | `search_files` | 文件名匹配（子串/正则） |
| **Glob 搜索** | `glob` | 按模式匹配路径，按修改时间排序 |
| **目录查看** | `list_directory` | 单层目录 |
| **目录树** | `directory_tree` | 递归树状结构（自动忽略依赖目录） |
| **文件元信息** | `get_file_info` | 大小、类型、修改时间 |
| **符号分析** | `get_symbols` | AST 驱动的文件结构解析 |
| **代码定位** | `find_in_code` | 文件内符号的 AST 过滤定位 |
| **Shell 命令** | `run_command` | 执行命令，禁止链式/后台操作 |
| **后台进程** | `run_background` | 启动开发服务器/长任务 |
| **作业管理** | `job_output` / `wait_for_job` / `stop_job` / `list_jobs` | 后台作业控制 |
| **网页搜索** | `web_search` / `web_fetch` | 搜索 + 抓取 URL 内容 |
| **持久记忆** | `remember` / `forget` / `recall_memory` | 跨会话存储偏好和事实 |
| **技能安装** | `install_skill` / `create_skill` | 创建可复用 playbook |
| **技能调用** | `run_skill` | 执行已安装的 playbook |
| **计划提交** | `submit_plan` | 多步修改的审批关卡 |
| **进度标记** | `mark_step_complete` | 计划步骤完成后标记 |
| **计划修订** | `revise_plan` | 调整剩余步骤 |
| **选择器** | `ask_choice` | 当需要用户选择时弹出菜单 |
| **任务跟踪** | `todo_write` | 会话内 3+ 步任务列表 |
| **MCP 注册** | `add_mcp_server` | 注册外部 MCP 服务器 |
| **子 Agent** | `explore` / `research` / `review` / `security_review` | 隔离只读调研 |

---

## 编辑系统（SEARCH/REPLACE）

Reasonix 的核心编辑模式是 **SEARCH/REPLACE**，这是一种精确匹配的替换机制：

### 编辑块格式

```
path/to/file.ts
<<<<<<< SEARCH
这行文本必须精确匹配文件中的内容
包括缩进和空格
=======
替换后的新内容
>>>>>>> REPLACE
```

### 规则

1. **读前编辑禁令** — 编辑前必须 `read_file` 目标文件，否则工具拒绝
2. **唯一匹配** — `SEARCH` 文本必须在文件中唯一存在
3. **批量编辑** — 跨文件修改用 `multi_edit`，全部验证后才写入
4. **创建新文件** — 用空 SEARCH 块：

```
path/to/new.ts
<<<<<<< SEARCH
=======
新文件全部内容
>>>>>>> REPLACE
```

5. **失败回滚** — `multi_edit` 验证失败则不写任何文件

### 为什么不用 sed / 直接覆盖？

SEARCH/REPLACE 让你可以看到**改了哪里、改了什么**，每处改动都可审计、可撤销。

---

## 规划与任务管理

Reasonix 提供三个层级的任务编排：

### submit_plan — 审批关卡（多步修改）

用于跨文件重构、架构改动、任何代价高的操作。

> **对话示例**：
>
> **你**：重构这个模块的认证流程
>
> **Reasonix**：提交 Plan，包含 step-1 提取 auth 中间件 → step-2 更新路由文件 → step-3 添加测试
>
> **你**：审批通过
>
> **Reasonix**：按步骤执行，每步调用 mark_step_complete

### todo_write — 会话内任务跟踪

用于 3 步以上但不需要审批的工作。

> **你**：帮我重构这个文件，涉及多个函数
> **Reasonix**：创建 todo 列表，逐项完成

### ask_choice — 分支选择

当你有多个选项需要决策时。

> **Reasonix**：你想怎么处理？
> - A) 删除这个弃用函数
> - B) 保留但加弃用标记
> - C) 提取到单独文件
>
> **你**：选 A

---

## 记忆系统

Reasonix 支持**跨会话持久记忆**，通过 `remember` / `forget` / `recall_memory` 管理。

### 记忆类型

| 类型 | 用途 | 示例 |
|------|------|------|
| `user` | 用户偏好/角色/技能 | "用户熟悉 Python 和 TypeScript" |
| `feedback` | 修正和确认的规则 | "不要用 var，用 const/let" |
| `project` | 项目的事实和决策 | "测试命令是 npm test" |
| `reference` | 外部系统指针 | "部署到 AWS ECS" |

### 作用域

| 作用域 | 有效期 | 说明 |
|--------|--------|------|
| `global` | 所有项目生效 | 个人偏好、通用规则 |
| `project` | 当前项目目录有效 | 项目特定约定 |

### 优先级

| 优先级 | 行为 |
|--------|------|
| `high` | 注入到 HIGH PRIORITY 块，始终加载 |
| `medium`（默认） | 正常加载到系统提示词 |
| `low` | 按需通过 `recall_memory` 读取 |

### 示例

```text
你：记住这个项目的代码风格是 2 空格缩进
→ remember({ type: "project", scope: "project", name: "indent-style",
     content: "2空格缩进，不使用 tab" })

下次启动 Reasonix：
→ 自动加载到系统提示词，无需重新说明
```

```text
你：忘记那个缩进规则
→ forget({ name: "indent-style", scope: "project" })
```

---

## 技能系统（Skills）

Skills 是可复用的 playbook，就像函数一样调用。有两种执行模式：

### 两种模式

| 模式 | 标签 | 说明 |
|------|------|------|
| **Inline（内联）** | 无标签 | playbook 直接注入到当前 agent 的上下文，工具调用和推理可见 |
| **Subagent（子 Agent）** | [🧬 subagent] | 在隔离的子 loop 中执行，只返回最终结论，上下文不污染主对话 |

### 内置技能

| 技能名称 | 模式 | 说明 |
|----------|------|------|
| `explore` | 🧬 subagent | 隔离子 agent 做代码库只读调查 |
| `research` | 🧬 subagent | 结合代码阅读和网络搜索做综合调研 |
| `review` | 🧬 subagent | 审查当前分支变更的正确性/安全/测试 |
| `security_review` | 🧬 subagent | 安全专项审查 |
| `test` | inline | 运行测试并诊断失败 |

### 安装自定义技能

```text
你：帮我写一个"检查日志"的技能
Reasonix：
→ install_skill({
    name: "log-checker",
    description: "扫描日志文件中的异常",
    body: "使用 grep 搜索 ERROR/WARNING 模式..."
  })
→ 以后可以通过 run_skill log-checker 调用
```

### 何时用子 Agent

- ✅ 需要并行独立调研
- ✅ 会读取 10+ 个文件的深度调查
- ❌ 简单的单次 grep 或跨文件检查
- ❌ 需要你交互决策的工作

---

## 子 Agent 系统

子 Agent（Subagent）是一个**完全隔离的 AI 对话循环**：

```text
主 Agent  ── spawn ──▶  子 Agent（独立上下文）
    │                       │
    │                   只读工具
    │                   read_file / search_content /
    │                   web_search / run_command
    │                       │
    ◀────── 返回最终结论 ────┘
```

### 内置子 Agent 技能

| 技能 | 用途 | 典型场景 |
|------|------|---------|
| `explore` | 代码库只读调查 | "找出发送邮件相关的所有代码路径" |
| `research` | 外部 + 内部综合调研 | "查一下 React 19 的新特性，对比我们项目中的用法" |
| `review` | 分支变更审查 | "审查我当前分支的改动有没有问题" |
| `security_review` | 安全专项审查 | "检查这些改动有没有安全漏洞" |

### 优势

- 工具调用不进入主 Agent 上下文
- 可并行运行多个子 Agent
- 只返回精炼的最终结论

---

## MCP 集成

Reasonix 支持通过 **Model Context Protocol（MCP）** 注册外部工具服务器，扩展自己的能力边界。

### 注册 MCP 服务器

```text
# 从内置目录注册（如 filesystem / memory / github）
→ add_mcp_server({
    name: "my-fs",
    from_catalog: "filesystem",
    args: ["/path/to/allowed/dir"]
  })

# 注册自定义 stdio 服务器
→ add_mcp_server({
    name: "custom-tool",
    transport: "stdio",
    command: "npx",
    args: ["-y", "@org/mcp-server"]
  })

# 注册远程 SSE 服务器
→ add_mcp_server({
    name: "remote-api",
    transport: "sse",
    url: "https://api.example.com/mcp"
  })
```

### 使用场景

- 连接数据库 MCP 做 SQL 查询
- 注册文件系统 MCP 访问隔离目录
- 连接 GitHub MCP 做 PR 管理
- 自定义工作流集成

---

## 身份自定义

Reasonix 的身份由**系统提示词（System Prompt）**定义。要修改身份和行为风格，需要编辑加载 Reasonix 时的系统提示词模板。

### 可以自定义的内容

| 项目 | 默认 | 可改为 |
|------|------|--------|
| **名称** | Reasonix Code | 任何名称 |
| **角色描述** | 编码助手 | 如"代码评审专家"、"全栈架构师" |
| **行为规则** | Cite or shut up / 编辑纪律 | 可放松或增强 |
| **风格偏好** | 先改后解释 | 如"先解释再改"、"极简回复" |
| **语言偏好** | 自动跟随用户 | 固定中文/英文 |
| **限制边界** | 不做未经要求的工作 | 可加更多限制 |

### 修改方法

取决于你的 Reasonix 部署方式：

- **CLI 版**：查找启动脚本或配置文件中 `--system-prompt` 或类似的参数
- **VS Code 插件版**：在插件设置中找 "System Prompt" / "自定义提示词"
- **自定义部署**：在加载系统提示词的入口处修改

**示例修改**：如果你希望 Reasonix 更简洁：

```text
你是 Reasonix Code，一个极简编码助手。
回答始终保持简短，不要解释为什么要做修改，
直接给出 SEARCH/REPLACE 块。
只在用户要求时提供解释。
```

> ⚠️ **注意**：修改系统提示词会影响所有行为规则，包括编辑纪律、证据要求等。如果移除了 "Cite or shut up" 规则，Reasonix 可能产生缺乏证据的断言。

---

## 编辑纪律（六条铁律）

Reasonix 的编辑行为受到六条硬性约束，确保精确性和可审计性：

### 1. 自动预览不是审计

`read_file` 的大文件模式返回首尾 + 符号大纲，中间部分被省略。**不要根据省略部分做结论**——有疑问就指定 `range:"A-B"` 重新读取。

### 2. Flag → 追查消费者

看到某个布尔字段（如 `parallelSafe?: boolean`），**不等于知道它的行为**。必须 `search_content` 找到消费这个 flag 的分支代码来确认它实际做什么。

### 3. 不编造百分比

没有亲手算过，就不能说"节省 40-60% token"。要么给出引用的实测数据，要么用模糊语言。

### 4. Schema 成本是真实的

每个工具的**描述文本**在每次请求中都会发送。提议新工具时，必须说明：
- 现有工具组合为什么做不到
- 大概的 token 成本
- 为什么不能通过修改 prompt 或工具描述来达到同样效果

### 5. MEMORY.md 是设计空间的一部分

`high` 优先级的记忆块是用户反馈过的内容。**推荐方案如果和 MEMORY.md 矛盾，就是错的**。必须在推荐前交叉检查。

### 6. 用户面向 ≠ 模型面向 ≠ 库面向

Reasonix 有四个交互面：
- **斜杠命令**（用户面）
- **工具**（模型面）
- **UI**（用户面）
- **库导出的 API**（`src/index.ts`，嵌入者面）

把用户功能提升为模型工具会破坏用户控制约束。把库导出当作"死代码"会误解设计——嵌入者直接消费 `src/index.ts`。

---

## 常见问题 / 故障排除

### 编辑被拒绝

**Q**: 我收到 "SEARCH 不匹配" 错误

**A**: 文件可能在你 `read_file` 后被修改了（或你直接 `write_file` 覆盖了）。重新 `read_file` 获取最新内容，再试编辑。

### 工具调用失败

**Q**: 提示 "unavailable in plan mode"

**A**: 你在 plan mode 下。调用 `submit_plan` 提交计划，审批通过后即可执行。

### 模型能力不足

**Q**: 任务太复杂，Reasonix 给出模糊的回答

**A**: Reasonix 会检测到这种情况并自动抛出 `<<<NEEDS_PRO>>>` 标记，自动升级到更强模型重试。你也可以直接说明"这个任务比较复杂"。

### 记忆不生效

**Q**: 我记得了某个事实，但下次会话没有出现

**A**: 检查记忆的作用域：`project` 作用域只在当前项目目录下生效；`global` 在所有项目生效。另外 `low` 优先级的记忆需要显式 `recall_memory` 读取。

### 技能找不到

**Q**: `run_skill` 提示技能不存在

**A**: 
```bash
# 查看已安装的所有技能
list_directory .reasonix/skills/
```

### 需要查看后台作业状态

```text
→ list_jobs  # 查看所有运行中和已退出的作业
→ job_output({"jobId": 1})  # 查看指定作业的输出
```

### 如何中断后台进程

```text
→ stop_job({"jobId": 1})  # 发送 SIGTERM，强行退出时 SIGKILL
```

---

> **Reasonix Code** — 精确、严谨、可审计的代码助手。
> 让每次编辑都有据可查，让每个断言都有源可追。
