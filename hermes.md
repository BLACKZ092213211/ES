# 1. **Hermes Agent入门**

## 1.1. **Hermes Agent介绍**

Hermes Agent 是由 Nous Research （也是 Hermes 系列大模型背后的实验室）推出的具备自我进化能力的通用AI Agent框架，它不仅能自主使用工具、编写并执行代码，还能在任务完成后自动总结经验，将工作流沉淀为可复用的“技能”，并在后续使用中持续优化这些技能。配合跨会话的持久化记忆与用户画像建模，Hermes 能够随着使用时间的增长变得越来越懂用户，真正实现了从“被动执行”到“主动进化”的跨越。

Hermes Agent 特点如下：

* 可灵活配置多种模型厂商，支持200+主流模型（从 OpenAI 到国产 Kimi、MiMo 等）
* 可以通过CLI访问，通过统一的网关设计，也可以 Telegram、QQ、微信等多平台访问。
* 自我进化能力，Skill可以自动生成、自我优化并写入长期记忆，后续可直接使用
* 空闲时成本接近为零，适合长期运行

Hermes Agent文档地址：[https://hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs)

## 1.2. **Hermes Agent安装**

Hermes Agent 的本质是一个基于 Python 和 Node.js 运行的智能体框架。支持 Linux、macOS、WSL2 和 Android (Termux)。安装程序会自动处理平台特定的配置。

**Linux / macOS / WSL2/Android**

```
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

**Windows（原生，PowerShell）** 

```
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

**桌面端(Linux/windows/macOS)**

```
https://hermes-agent.nousresearch.com/
```



## 1.3. **Hermes Agent 配置**

### 1.3.1. **选择Provider**

这是最重要的配置步骤。使用 `hermes model` 以交互方式完成选择：

```
hermes model
```

或者通过CLI设置

```
hermes config set model anthropic/claude-opus-4.6
hermes config set OPENROUTER_API_KEY sk-or-...
```

### 1.3.2. **目录结构**

安装完成Hermes Agent后，所有设置均存储在~/.hermes/目录中，以便访问。该目录下的结构如下：

```
~/.hermes/
├── config.yaml     # 配置项（模型、终端、TTS、压缩等）
├── .env            # API 密钥与敏感信息
├── auth.json       # OAuth 提供商凭证（Nous Portal 等）
├── SOUL.md         # 主 Agent 身份（系统提示中的第 1 槽位）
├── memories/       # 持久记忆（MEMORY.md、USER.md）
├── skills/         # Agent 创建的技能（由 skill_manage 工具管理）
├── cron/           # 定时任务
├── sessions/       # 网关会话
└── logs/           # 日志（errors.log、gateway.log——自动脱敏）
```

配置优先级按照如下顺序进行解析（优先级从高到低）：

1) CLI 参数 —— 例如 hermes chat --model anthropic/claude-sonnet-4（每次调用的覆盖）
2) ~/.hermes/config.yaml —— 用于所有非敏感设置的主要配置文件
3) ~/.hermes/.env —— 环境变量的备用位置；必须用于敏感信息（API 密钥、令牌、密码）
4) 内置默认值 —— 当其他设置均未配置时使用的硬编码安全默认值



## 1.4. **Hermes Agent 简单使用**

* **对话**

可以在终端输入“hermes”与Hermes Agent进行对话

```
hermes
```

![截屏2026-06-29 23.59.08](/Users/zzp/Desktop/截屏2026-06-29 23.59.08.png)

* **多行输入**
  * macOs：option + enter
  * Windows：alt + enter	


* **中断Agent**

与Hermes Agent进行对话过程中，可以按“Ctrl+C”中断Agent运行并切换到新的指令。2秒内再次按下“Ctrl+C”会退出 Hermes Agent。

退出对话后，可以看到可以通过“hermes --resume xxx”来恢复本次对话。

* **/model**

在对话中输入“/model”可以列出已有模型并切换

## 1.5. **Hermes Agent 命令**

以下命令可以在安装Hermes Agent完成后执行。

```
#开始进入聊天
hermes

#选择大语言模型提供商和模型
hermes model

#配置启动哪些工具
hermes tools

#完整设置向导（一次性完成全部配置）
hermes setup

#查看当前配置
hermes config

#重启hermes gateway服务(如果安装后，通过此命令启动)
hermes gateway restart

#配置连接消息平台
hermes gateway setup

#恢复上次对话
hermes --continue

#诊断问题
hermes doctor

#更新至最新版本
hermes update
```

更多命令可以通过“hermes --help”查看。

# 2. **Hermes Agent 记忆与上下文**

## 2.1. **持久记忆**

在传统的 AI 对话应用中，上下文（Context）通常局限于当前会话窗口，一旦会话结束，Agent 就会“失忆”。Hermes Agent 的持久化记忆（Persistent Memory）旨在打破这一限制，通过文件系统与系统提示词（System Prompt）的巧妙结合，让 Agent 能够跨会话记住你的开发环境、项目架构以及个人偏好及所学知识。

Hermes Agent 的持久记忆保存在本地客户端的文件系统中，具体位于 ~/.hermes/memories/目录下。系统通过两个核心文件来划分记忆的边界：

**1)** [**MEMORY.md**](http://MEMORY.md)**(Agent 的个人笔记)**

用途：用于记录 Agent 需要记住的环境、工作流和经验教训信息，例如：环境事实(操作系统、工具、项目结构)、项目配置、代码规范、踩坑经验等“客观信息”。

容量限制：2,200 字符（约 1500-3000 tokens，取决于中英文比例）。

**2)** [**USER.md**](http://USER.md)**(用户画像)**

用途：用于记录关于用户身份、偏好和沟通风格的信息，例如用户姓名、角色、时区、沟通风格、偏好设置、工作习惯等“主观信息”。

容量限制：1,375 字符（约 1000-2000 tokens，取决于中英文比例）。

* **持久记忆加载机制：**

以上两个文件在会话启动时，系统会将 [MEMORY.md](http://MEMORY.md)和 [USER.md](http://USER.md)的内容一次性读取，并格式化后注入到 System Prompt 中（位于 SOUL.md 之后、项目上下文之前）。模型在处理后续对话时，无需重新计算这部分静态内存的 Token，从而大幅降低了首字响应时间（TTFT）并减少了算力成本。如果在会话中你修改了项目配置，Agent 会通过工具更新文件，但本次会话的 System Prompt 不会动态刷新。这种设计避免了频繁变动的 System Prompt 破坏 LLM 的缓存命中率。直到下一次新会话开启，Agent 才会加载最新的记忆。

* **持久记忆写入与删除：**

Agent 通过内置的 memory工具来对记忆文件进行 CRUD（增删改查）操作。开发者无需手动编辑 Markdown 文件，Agent 会根据对话上下文自动触发保存行为。当记忆容量接近上限时，Agent 会被引导去合并碎片的记忆，例如将三条分散的记录（“项目用 Java”、“项目用 MyBatis”、“项目用 MySQL”）合并为一条紧凑的条目（“项目 ~/code/api 使用 Java + MyBatis + MySQL”）。

## 2.2. **上下文系统**

### **2.2.1. 上下文种类**

在 Hermes Agent中，上下文主要分为三类：持久化记忆（Memory）、项目上下文（Project Context）和人格设定（[SOUL.md](http://SOUL.md)）。

**1) 持久化记忆**

这是 Agent 跨会话保存的“笔记本”，位于 ~/.hermes/memories/。[包含MEMORY.md和USER.md](http://包含MEMORY.md和USER.md)。

* [MEMORY.md](http://MEMORY.md):Agent 的“工作日志”。记录环境信息（如 OS 版本、工具链）、项目架构、踩过的坑。
* [USER.md](http://USER.md):Agent 的“用户档案”。记录你的偏好（如“我喜欢简洁回复”、“我不用 sudo”）。

以上两个文件由 Agent 在对话过程中通过 memory工具动态创建。

**2) 项目上下文**

这是针对当前代码库的“说明书”，位于你的项目目录中。Hermes 只会加载其中一种（按优先级）：

* .[hermes.md](http://hermes.md)：Hermes 原生的项目指令（最高优先级）。
* [AGENTS.md](http://AGENTS.md)：通用的项目规范文件（推荐）。
* [CLAUDE.md](http://CLAUDE.md)：兼容 Claude Code 的上下文文件。
* .cursorrules：兼容 Cursor IDE 的规则文件。

以上这些文件是使用对应工具后，在项目目录中自动/手动创建，如果项目是从 Cursor 或 Claude Code 迁移过来，直接使用现有的 .cursorrules或 [CLAUDE.md](http://CLAUDE.md)即可，Hermes 会自动识别，无需额外操作，Hermes加载时，默认支持20000字符，如果过长，保留首尾（头70%，尾20%），截断中间。

**3)** [**SOUL.md**](http://SOUL.md)

Hermes Agent 的灵魂，该文件是全局唯一文件，位于“~/.hermes/[SOUL.md](http://SOUL.md)”，该文件决定“Hermes 应该以什么样的姿态与你对话”。当第一次运行 Hermes 时，如果检测到 $HERMES\_HOME下没有 [SOUL.md](http://SOUL.md)，系统会自动生成一个默认版本。后续只需要按需修改即可。Hermes加载时，默认支持20000字符，如果过长，保留首尾（头70%，尾20%），截断中间。

[SOUL.md](http://SOUL.md)编写示例如下：

```
# 身份：务实的高级工程师

你是一位资深经验的后端工程师。你的核心目标是交付简单、可靠且可维护的解决方案。

## 沟通风格
- **直接且简洁**：直击要点，避免客套话和过度寒暄。默认使用中文回复。
- **敢于质疑**：如果用户的请求在技术上是错误的或危险的，请明确指出风险，不要盲目执行。
- **透明坦诚**：如果不确定答案，请直接说“我不知道”，不要编造事实（幻觉）。

## 工作准则
- **代码优先**：能用代码说明的，不要用长篇大论解释。
- **安全第一**：在编写脚本或修改配置时，始终优先考虑安全性和权限最小化。
- **实用主义**：反对过度设计。选择最成熟的方案，而不是最炫酷的方案。

## 禁忌
- 不要使用“作为一个AI...”这类免责声明。
- 不要在回复中加入表情符号（Emoji）。
- 不要猜测文件路径，使用前先用工具确认。
```

### **2.2.2. 上下文加载流程与对比**

当在终端输入 hermes启动会话时，按照如下步骤加载上下文：

1) [加载SOUL.md](http://加载SOUL.md)：读取 $HERMES\_HOME/[SOUL.md](http://SOUL.md)\-> 安全扫描 -> 加载。
2) 加载记忆：读取 ~/.hermes/memories/[MEMORY.md](http://MEMORY.md)和 [USER.md](http://USER.md)\-> 注入系统提示。
3) 探测项目：从当前目录递归向上遍历，寻找 .[hermes.md](http://hermes.md)\-> [AGENTS.md](http://AGENTS.md)\-> [CLAUDE.md](http://CLAUDE.md)\-> .cursorrules（优先级递减，第一个命中的胜出）。
4) 截断处理：如果文件超过 20,000 字符，保留前 70% 和后 20%，中间插入截断标记。
5) 最终组装：将上述所有内容组合成最终的 System Prompt，并在此后冻结。

通过这种分层、分级、动静结合的上下文设计，Hermes 既保证了对话的高效性（缓存），又满足了复杂项目的灵活性（渐进式发现）。

三种上下文对比如下：

| **上下文文件**    | **特点**                     | **作用域** | **生命周期**       |
| ----------------- | ---------------------------- | ---------- | ------------------ |
| MEMORY.md/USER.md | 自主学习，当前记住了什么内容 | 跨会话实例 | 动态变化，随时更新 |
| Agents.md等       | 定义项目规范                 | 跟随项目   | 随项目迭代而更新   |
| SOUL.md           | 定义Agent角色                | 全局       | 长期稳定，很少改动 |







# 3\. **Hermes Agent Session会话**

## 3.1. **Session会话工作原理**

Hermes Agent 的核心特性之一是将每一次交互自动持久化为“会话”（Session）。这不仅是为了断点续传，更是为了支持跨平台的历史回溯与全文检索。

从 v0.17.0 开始，Hermes 采用 SQLite 作为 Session 的主存储方案，替代了旧版的 JSONL 文件方案。这带来了两大好处：FTS5 全文搜索引擎让历史检索更快，结构化存储让查询更灵活。

* **SQLite 数据库（~/.hermes/state.db）**：这是 Session 的**主存储**。负责存储结构化元数据，包括 Session ID、来源平台（CLI/Telegram 等）、用户 ID、会话标题、模型配置、Token 消耗统计及完整消息历史。内置 FTS5 全文搜索索引，支持 `session_search` 工具跨会话检索。
* **JSONL 导出文件**：`hermes sessions export` 命令可以从 SQLite 导出为 JSONL 格式，用于审计、备份或数据迁移。导出的是**按需生成**的快照，不是实时同步的转录副本。
* **遗留文件（~/.hermes/sessions/）**：该目录中的 `session_*.json` 是 v0.17.0 之前旧版 JSONL 方案的遗留文件，新版不再往里写入。`request_dump_*.json` 是调试用的原始 API 请求 dump，不是 Session 主存储。

Hermes Agent中虽保存了完整历史，但在每一轮对话中，并不会将过往所有的媒体文件（图片、音频）字节重新注入模型上下文。

* 对于媒体文件，图像仅在调用视觉模型时被临时处理，随后转为文本描述或本地缓存路径存入数据库；音频则转为文本，未来的对话轮次仅继承这些文本结论，而非原始二进制文件。
* 对于文本内容，导致 Token 耗尽的主因通常是直接粘贴的长文本（如日志、Diff）。官方建议使用 /compress压缩摘要或通过文件路径引用，而非全量输入。

## 3.2. **Session会话相关命令**

* **列出会话命令-hermes sessions list**

会话首次交互后，后台线程会自动调用轻量级模型生成 3-7 词的描述性标题，无感知且不阻塞主流程。可以使用“hermes sessions list”查看自动生成的标题，“hermes sessions list”就是列出近期会话的命令。

```
hermes sessions list
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/20/1779606186063/02adf6930f134e719c21fcc2d24f9dc9.jpg)

* **手动设置会话标题-/title**

我们也可以在任意对话Cli中使用“/title”斜杠命令手动设置标题：

```
/title myfirst_chat
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/20/1779606186063/31be03c7012446039ab48cc262d6c6c3.jpg)

退出Hermes 再次查看生成的标题,可以看到最近对话名称为自己设置的内容。

关于会话名称注意点如下：

1) 自动命名仅在每个会话中触发一次，如果已手动设置标题，则会跳过自动命名。
2) 两个不同的会话不能共享相同的标题，标题最多100个字符
3) /title 命令在所有的网关平台（Telegram、Discord、Slack、WhatsApp）中均有效

* **恢复对话**

可以使用--continue 或 --resume从 CLI 恢复之前的对话。

```
# 恢复最近的CLI Session一次会话
hermes --continue  或者 hermes -c

# 按照名称恢复会话
hermes -c "myfirst_chat"

#通过ID 恢复特定的session会话
hermes --resume  20260520_142041_660118 等价于 hermes -r  20260520_142041_660118
#按照标题恢复session会话
hermes --resume  myfirst  等价于 hermes -r myfirst
```

注意：session id 需要通过执行“hermes sessions list”查看。

* **导出会话-hermes session export xxx**

可以执行“hermes sessions export backup.jsonl”命令将所有 sessions 导出到 JSONL 文件。

```
#导出所有sessions 会话到文件
hermes sessions export backup.jsonl
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/20/1779606186063/20835a8f0920494d9fb3bd007556c4af.jpg)

也可以执行如下命令导出指定的会话内容：

```
# 导出单个session
hermes sessions export session.jsonl --session-id 20260520_142041_660118
```

注意：session\_id 需要通过执行“hermes sessions list”命令查看确定。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/20/1779606186063/157d19ecc826444ca7c5524bbb52804b.jpg)

* **删除会话-hermes sessions delete xxx**

执行如下命令删除指定会话，需要传入会话ID：

```
# 删除特定 session（需要确认）
hermes sessions delete 20260520_142041_660118

# 跳过确认直接删除
hermes sessions delete 20260520_142041_660118 --yes
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/20/1779606186063/007fc3c4b05b41e89b998ba7b7b50115.jpg)

* **清理旧会话-hermes session prune**

```
# 删除超过 90 天的结束 sessions（默认）
hermes sessions prune

# 自定义年龄阈值，超过30天的会话被删除
hermes sessions prune --older-than 30
```

注意：清理操作仅删除已结束的会话（即显式结束或自动重置的会话）。活跃会话永远不会被清理。

# 4. **Hermes Agent Profiles**

## 4.1. 什么是Profile

Profile（配置文件）是 Hermes Agent 的一项核心设计，它允许你在同一台机器上运行多个**完全独立**的 Agent 实例。每个 Profile 拥有自己的一套配置、人格、技能、记忆和会话记录，彼此之间互不干扰。

**Profile 的目录结构：**

每个 Profile 位于 `~/.hermes/profiles/<name>/`，目录布局与默认实例完全一致：

```
~/.hermes/profiles/<name>/
├── config.yaml          # 独立配置（模型、终端、工具集等）
├── .env                 # 独立的 API 密钥（可选，默认继承全局 .env）
├── SOUL.md              # 该 Profile 专属的人格定义
├── skills/              # Profile 专属技能（默认从全局复制74个内置技能）
├── sessions/            # 独立的会话历史
├── state.db             # 独立的 SQLite 会话数据库
├── memories/            # 独立的持久记忆（MEMORY.md、USER.md）
├── logs/                # 独立日志
└── auth.json            # 独立的认证凭证
```

**Profile 解决了什么问题？**

在实际使用中，一个 Agent 很难同时胜任所有角色。例如：

* 作为**学习助手**时，你希望它懂 Java、用中文写笔记、把文件保存到 Obsidian Vault。
* 作为**代码审查者**时，你希望它严格检查代码规范、使用英文回复、专注技术细节。
* 作为**日常助手**时，你希望它随意聊天、帮你查天气、控制智能家居。

如果只用一个 Agent 承载所有这些需求，你需要在每次对话前反复强调上下文（"请用中文回复"、"请保存到 Obsidian"），不仅繁琐，还会消耗大量 Token。Profile 让你为每个角色创建一个**专属 Agent**，一次配置，永久生效。

**Profile 的三大核心组件：**

Profile 的设计遵循"身份 + 流程 + 环境"三层分离原则：

| 层级 | 文件 | 作用 | 示例 |
|------|------|------|------|
| **SOUL.md** | 定义 Agent **是谁** | 人格、语气、沟通风格 | "你是 Java 程序员，正在学 Python，用中文回复" |
| **Skill** | 定义 Agent **怎么做** | 工作流程、模板、规范 | "笔记必须包含 YAML frontmatter，使用 `[[WikiLink]]`" |
| **config.yaml** | 定义 Agent **在哪工作** | 模型、工作目录、工具集 | `terminal.cwd: /Users/you/obsidian-vault` |

三者配合：SOUL.md（身份）+ Skill（流程）+ config.yaml（环境）= 一个完整的专职 Agent。

## 4.2. Profile 创建与使用

### 4.2.1. 创建 Profile

使用 `hermes profile create` 命令创建新 Profile。最常用的方式是从默认 Profile 克隆：

```
# 从默认 profile 克隆，创建一个名为 "note" 的新 profile
hermes profile create note --clone-from default
```

执行后，系统会自动完成以下操作：

1) 在 `~/.hermes/profiles/note/` 创建完整的目录结构。
2) 从默认 Profile 复制 config.yaml、74 个内置技能。
3) 在 `~/.local/bin/note` 创建快捷别名（shell alias），之后直接在终端输入 `note` 即可启动该 Profile。

创建时的常用选项：

```
# 从指定 profile 克隆
hermes profile create code-reviewer --clone-from note

# 创建空白 profile（不克隆任何配置）
hermes profile create minimal --blank

# 克隆全部（包括记忆和会话）
hermes profile create full-copy --clone-from default --clone-all
```

### 4.2.2. 配置 Profile 的 SOUL.md

创建 Profile 后，第一步是编辑 `~/.hermes/profiles/<name>/SOUL.md`，定义这个 Agent 的身份。以下是一个学习笔记助手的 SOUL.md 示例：

```
# 学习笔记助手

你是一个专注于学习笔记整理的助手。你的核心任务是帮助用户将学到的知识转化为高质量、结构化的 Obsidian 笔记。

## 你的背景
- 你理解 Java 程序员的学习视角
- 你熟悉 Python 和 AI/ML 领域的技术栈
- 你的学习路径是：NumPy → PyTorch → Transformer → LLM 微调 → RAG/Agent

## 语言与风格
- 默认使用中文撰写笔记
- 技术术语保留英文原名（如 "Backpropagation" 而非 "反向传播"）
- 按 "是什么 → 为什么 → 怎么用" 的逻辑组织内容
- 在合适的地方用 Java 类比帮助理解

## 输出要求
- 每篇笔记必须包含 YAML frontmatter（title、date、tags）
- 使用 [[WikiLink]] 实现笔记间的双向链接
- 代码示例必须可运行，包含关键注释
- 每篇笔记结尾列出与已有知识的关联
```

### 4.2.3. 配置工作目录

编辑 Profile 的 `config.yaml`，设置 Agent 的工作目录：

```yaml
# ~/.hermes/profiles/note/config.yaml
terminal:
  cwd: /Users/you/obsidian-vault   # Agent 启动后的默认工作目录
```

这样当该 Profile 启动时，所有文件操作都默认在这个目录下进行。

### 4.2.4. 共享全局技能

默认情况下，每个 Profile 只能看到自己 `skills/` 目录下的技能。如果想共享全局安装的技能（位于 `~/.hermes/skills/`），需要在 config.yaml 中添加：

```yaml
# ~/.hermes/profiles/note/config.yaml
skills:
  external_dirs:
    - /Users/you/.hermes/skills
```

这样 Profile 既能使用自己的专属技能，也能访问全局技能库。

### 4.2.5. Profile 相关命令

```
# 列出所有 profile
hermes profile list

# 查看某个 profile 的详细信息
hermes profile show note

# 切换默认 profile（设置后 hermes 命令默认使用该 profile）
hermes profile use note

# 重命名 profile
hermes profile rename note study-notes

# 删除 profile（不可恢复）
hermes profile delete old-profile

# 导出 profile 为压缩包（用于迁移或备份）
hermes profile export note -o note-profile.tar.gz

# 从压缩包导入 profile
hermes profile import note-profile.tar.gz

# 管理别名（创建/删除快捷命令）
hermes profile alias note    # 查看别名
hermes profile alias note --remove   # 删除别名
```

### 4.2.6. 启动 Profile 的两种方式

**方式一：使用别名（推荐日常使用）**

```
note          # 直接输入别名启动，cwd 固定为 config.yaml 中的配置
```

**方式二：使用 -p 参数**

```
hermes -p note     # cwd 跟随当前终端目录，适合在不同项目中临时使用
```

例如，你有一个固定指向 Obsidian Vault 的 `note` Profile，当你执行 `note` 时，Agent 始终在 Obsidian Vault 中工作。但如果你想临时用同一个 Profile 处理另一个项目，可以使用 `cd /other/project && hermes -p note`，此时 Agent 会在 `/other/project` 中工作，并自动加载该项目的 AGENTS.md。








## 4.3. Profile 分发：共享完整 Agent

Profile 分发是 Hermes 官方提供的 Agent 共享机制。它将一个完整的 Agent——人格（SOUL.md）、技能（skills）、定时任务（cron）、MCP 连接、配置（config.yaml）——打包为一个 **git 仓库**。任何有权限访问该仓库的人，都可以用一条命令安装整个 Agent，后续还可以一键更新，而自己的记忆、会话和 API 密钥完全不受影响。

如果说 Profile 是本地 Agent，那么分发就是让 Agent 变成可共享、可版本管理的产品。

### 4.3.1. 分发解决了什么问题

在没有分发功能之前，共享一个 Hermes Agent 意味着要分别发送：

1) 你的 SOUL.md
2) 需要安装的技能列表
3) 去掉密钥的 config.yaml
4) MCP 服务器连接说明
5) 所有 cron 任务配置
6) 环境变量设置说明

……然后祈祷对方能正确组装。每次版本升级或修复 Bug 都意味着重复这一过程。

有了分发功能，这一切都存放在一个 git 仓库中：

```
my-research-agent/
├── distribution.yaml    # 清单文件：名称、版本、所需环境变量
├── SOUL.md              # Agent 人格 / 系统提示词
├── config.yaml          # 模型、temperature、推理、工具默认值
├── skills/              # 随 Agent 捆绑的技能
├── cron/                # Agent 运行的定时任务
└── mcp.json             # Agent 连接的 MCP 服务器
```

接收方只需要一条命令：

```
hermes profile install github.com/you/my-research-agent --alias
```

填入自己的 API 密钥（`.env.EXAMPLE` → `.env`），即可运行 `my-research-agent chat`。当作者推送新版本时，接收方运行 `hermes profile update my-research-agent` 即可拉取更改——记忆和会话保持不变。



# 5. **Hermes Agent 其他**

## 5.1. **Hermes Agent 语音模式**

```
9.43 03/06 NWz:/ :6pm G@I.ip Hermes实测：对着麦克风说话，AI直接语音回复你 # Hermes # AI助手 # 语音交互 # AI工具 # 效率工具  https://v.douyin.com/h1fYl9TxHpk/ 复制此链接，打开Dou音搜索，直接观看视频！
```

## 5.2. **Hermes Agent WebUI**

```
hermes dashboard
```

![image-20260701214529180](/Users/zzp/Library/Application Support/typora-user-images/image-20260701214529180.png)

## 5.2. **Hermes Agent 消息平台**

3.30 :7pm O@K.JV 06/08 GVY:/ Hermes接入企业微信飞书钉钉微信QQ全攻略 5个平台一次性全讲清楚！ 企业微信/飞书/钉钉/微信/QQ 全部官方支持，扫码就能接 跟着做，10分钟搞定全部配置 # HermesAgent# AI机器人# 企业微信机器人# 自动化办公# 效率工具  https://v.douyin.com/ncrZo47DHWA/ 复制此链接，打开Dou音搜索，直接观看视频！
