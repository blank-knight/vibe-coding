# Vibe Coding 完整工作流指南

> 五步法：从 idea 到可维护的 AI 辅助项目。

---

## Step 1: 程序初步设计文档

**目标：** 把脑子里的想法变成 AI 能理解的文字。

1. 把 idea 交给 AI，让其生成初步的程序设计文档
2. 命名为 `program-design-document.md`
3. 可以简陋，但**目标要准确**——之后朝着目标不断迭代
4. 同时用 Canvas 白板生成架构：让 AI 根据 `program-design-document.md` 生成 Obsidian Canvas 架构图 `architecture.canvas`，作为单一真相源存入 memory-bank

**提示词示例：**
```
我想做一个 [你的项目描述]。
请帮我生成程序设计文档，包含：
1. 项目目标（一句话）
2. 核心功能列表
3. 用户流程
4. 数据模型
5. 非功能需求（性能、安全等）
```

---

## Step 2: 确定技术栈和 Agent 代码生成规则

**目标：** 选定技术栈，建立 AI 生成代码的规则。

### 2.1 胶水编程选型

1. 让 AI 结合**胶水编程**思路，推荐可复用的 GitHub 组件
2. 确定最简单但健壮的技术栈，命名为 `tech-stack.md`

**胶水编程的核心原则：**
- 优先找成熟轮子，不要从零造
- 让 AI 搜索 GitHub Topics 找开源实现
- 对比质量、维护活跃度、许可证
- AI 只负责写组件之间的"连接代码"

**常用 GitHub Topics 参考：**

| 需求 | 推荐 Topic |
|------|----------|
| Telegram Bot | telegram-bot |
| 数据分析 | data-analysis |
| AI Agent | ai-agent |
| CLI 工具 | cli |
| Web 爬虫 | web-scraping |
| Web 框架 | web-framework |

### 2.2 生成 Agent 代码生成规则

使用 AI 工具的 `/init` 或项目初始化功能，让它阅读设计文档和技术栈，生成代码生成规则。

### ⚠️ 关键：审查生成的规则

确保规则强调**模块化（多文件）**和**禁止单体巨文件（monolith）**。可能需要手动修改或补充规则。

### 🔴 极其重要：Always 规则

某些规则必须设为 "Always"（始终应用），确保 AI 在生成任何代码前都强制阅读。例如：

```
# 重要提示：
# 写任何代码前必须完整阅读 memory-bank/architecture.md（包含完整数据库结构）
# 写任何代码前必须完整阅读 memory-bank/program-design-document.md
# 生成代码前，先用生成器 G 根据当前任务意图生成初始提示词，
# 再用优化器 O 按预设标准 Ω 优化提示词，
# 最后让元生成器 M 根据优化结果更新生成器 G 的规则到 memory-bank/g-updated-rules.md 中。
# 对于每次重新生成的提示词 P，将其内容更新到 memory-bank/p-history.md
# 每完成一个重大功能或里程碑后，必须更新 memory-bank/architecture.md 和 memory-bank/architecture.canvas
```

### 其他规则

其他（非 Always）规则要引导 AI 遵循技术栈的最佳实践（如网络、状态管理等）。

> 如果想要代码最干净、项目最优化，这一整套规则设置是**强制性的**。

---

## Step 3: 实施计划

**目标：** 把大任务拆成 AI 能一步步执行的小任务。

根据 `tech-stack.md` 和 `program-design-document.md`，让 AI 生成详细的实施计划 `implementation-plan.md`：

**核心原则：**
- 包含一系列给 AI 开发者的**分步指令**
- 每一步要**小而具体**
- 同时用优化后的提示词生成代码
- 检查 Canvas 白板架构的模块连接，用胶水代码拼接组件
- 每一步都必须包含**验证正确性的测试**
- **严禁包含代码**——只写清晰、具体的指令
- 先聚焦于**基础代码**，完整功能后面再加
- 若任务复杂，计划中明确启动 tmux 多窗格，让不同 AI Agent 按白板分工协作，共享 memory-bank 状态文件

**提示词示例：**
```
阅读 memory-bank/program-design-document.md 和 memory-bank/tech-stack.md，
生成详细的实施计划 implementation-plan.md：
- 分成 5-15 个步骤
- 每步只做一件小事
- 每步包含验证测试
- 不要在计划里写代码
- 先做核心功能，高级功能后面加
```

---

## Step 4: 记忆库（Memory Bank）

**目标：** 建立项目的外部记忆系统。

### 新建项目文件夹

在项目根目录下创建子文件夹 `memory-bank/`，将以下文件放入：

| 文件 | 说明 | 初始状态 |
|------|------|---------|
| `program-design-document.md` | 程序设计文档 | Step 1 产出 |
| `tech-stack.md` | 技术栈 | Step 2 产出 |
| `implementation-plan.md` | 实施计划 | Step 3 产出 |
| `progress.md` | 已完成步骤 | 空文件 |
| `architecture.md` | 每个文件的作用 | 空文件 |
| `architecture.canvas` | Obsidian Canvas 架构图 | Step 1 产出 |
| `vibe-coding-core.md` | 项目规则 | Step 2 产出 |
| `g-updated-rules.md` | 生成器 G 的每次迭代 | 初始规则 |
| `p-history.md` | 提示词 P 的每次迭代 | 空文件 |

**为什么 Memory Bank 有效？**

AI 工具的对话窗口有限（通常 100K-200K tokens）。一个复杂项目聊几十轮对话后，早期的重要上下文会被压缩或遗忘。Memory Bank 把关键信息持久化到文件里：

- 新对话开始时，AI 只需要读一遍 `memory-bank/` 即可恢复完整上下文
- 不依赖特定 AI 工具的"记忆功能"——换工具也不丢
- 文件就是文档，人也能读

---

## Step 5: 开始实施

**目标：** 按计划逐步执行，每步验证。

### 5.1 确保计划清晰

启动你的 AI 编程工具，对实施计划进行完善：

```
阅读 memory-bank/ 里所有文档，implementation-plan.md 是否完全清晰？
你有哪些问题需要我澄清，让它对你来说 100% 明确？
```

### 5.2 实施提示词

> ⚠️ 永远先用 "Ask" 模式或 "Plan Mode"（Claude Code 中按 shift+tab），确认满意后再让 AI 执行该步骤。

```
阅读 memory-bank/ 所有文档，然后执行实施计划的第 1 步。
我会负责跑测试。在我验证测试通过前，不要开始第 2 步。
验证通过后，打开 progress.md 记录你做了什么供后续开发者参考，
再把新的架构洞察添加到 architecture.md 中解释每个文件的作用。
```

### 5.3 后续工作（每次新对话）

每次重新开始、新建聊天时，让 AI 阅读对应的 md 文件即可：

```
阅读 memory-bank/ 所有文件，阅读 progress.md 了解之前的工作进度，
然后继续实施计划第 2 步。在我验证测试前不要开始第 3 步。
```

这就是 Memory Bank 的威力——**你永远不需要重复解释上下文。**

---

## 实用技巧

### 代码回退
- Claude Code 用 `/rewind` 回退
- 其他工具：用 git 管理，`git checkout` 回退

### 提升提示词效果
- 加一句：**"慢慢想，不着急，重要的是严格按我说的做，执行完美。如果我表达不够精确请提问。"**
- Claude Code 中触发深度思考的关键词强度：`think` < `think hard` < `think harder` < `ultrathink`

### 清理上下文
- 经常用 `/clear` 或 `/compact`（保留历史对话）来管理上下文窗口

### 功能迭代
- 每增加一个主要功能，就新建一个 `feature-implementation.md`，写短步骤+测试

### Git 版本管理
- 每完成一个步骤并通过测试 → 立即 commit
- 打 tag 标记里程碑，方便回退
- message 格式建议：`[step-N] 描述`

---

## 元方法论闭环（进阶）

每次让 AI 写代码前，执行以下循环：

```
意图 I → 生成器 G 生成提示词 P → 优化器 O 优化 P* → 元生成器 M 更新 G'
         ↑                                                        |
         └────────────────────────────────────────────────────────┘
```

1. **生成器 G** 接收意图，输出提示词 P
2. **优化器 O** 接收 P 和理想标准 Ω，输出优化后 P\*
3. **元生成器 M** 接收当前 G 和 P\*，输出更强的 G'
4. G' 替换旧 G，循环继续

详见 [METHODOLOGY.md](METHODOLOGY.md) 的"元提示词递归闭环"章节。
