# 🎵 Vibe Coding — AI 辅助开发方法论

> 一套经过实战检验的 AI 辅助软件开发方法论框架。
> Memory Bank + 元提示词闭环 + 胶水编程 + Canvas 白板驱动 + AI 蜂群协作。

## 这是什么？

Vibe Coding（由 Andrej Karpathy 在 2025 年提出）是一种 AI 辅助的软件开发范式——开发者用自然语言描述意图，AI 生成代码。但"纯靠感觉写代码"会导致项目失控：代码质量不可控、技术债务堆积、AI 上下文丢失。

**本项目解决的问题：** 给 Vibe Coding 装上"护栏"。通过结构化的记忆库、分步实施计划、提示词自我进化闭环，让 AI 编程从"碰运气"变成"可复现的工程流程"。

## 核心理念

| 方法 | 解决什么问题 |
|------|-------------|
| **Memory Bank** | AI 没有记忆——每次新对话都忘了之前做了什么。用文件系统充当外部记忆。 |
| **元提示词闭环 (G/O/M)** | 提示词质量不稳定——用"生成器→优化器→元更新器"的递归循环让提示词自我进化。 |
| **胶水编程** | AI 容易幻觉出不存在的 API——优先复用成熟开源库，AI 只负责写"连接代码"。 |
| **Canvas 白板驱动** | 架构只存在于脑子里——用 Obsidian Canvas 作为可视化的单一真相源。 |
| **AI 蜂群协作** | 单 Agent 瓶颈——用 tmux 让多个 AI Agent 分工协作、互相感知。 |

## 快速开始

### 方式一：复制模板到新项目

```bash
# 1. 克隆本仓库
git clone https://github.com/axibawa/vibe-coding.git

# 2. 复制模板到你的新项目
cp -r vibe-coding/template/ /path/to/your-new-project/

# 3. 按步骤执行（详见 GUIDE.md）
```

### 方式二：手动搭建

1. 在项目根目录创建 `memory-bank/` 文件夹
2. 按顺序创建以下文件（模板见 `template/` 目录）：
   - `program-design-document.md` — 程序设计文档
   - `tech-stack.md` — 技术栈选型
   - `implementation-plan.md` — 分步实施计划
   - `progress.md` — 进度记录（空文件起步）
   - `architecture.md` — 架构说明（空文件起步）
   - `vibe-coding-core.md` — 项目规则
   - `g-updated-rules.md` — 提示词生成器迭代记录
   - `p-history.md` — 提示词历史
3. 创建 `CLAUDE.md`（或 `AGENTS.md`）作为 AI 代码生成规则入口

### 工作流一览

```
idea → program-design-document.md + architecture.canvas
  ↓
胶水编程找轮子 → tech-stack.md
  ↓
Agent 代码生成规则（含 Always 规则 + 元方法论闭环）
  ↓
implementation-plan.md（分步指令 + 每步测试）
  ↓
memory-bank/ 全套文件
  ↓
AI 工具逐步执行（Claude Code / Codex / Cursor / 任何 AI 助手）
  ↓
每步验证 → 更新 progress.md + architecture.md
  ↓
下一个功能 / 下一个里程碑
```

## 目录结构

```
vibe-coding/
├── README.md                ← 你在看这个
├── GUIDE.md                 ← 完整工作流指南（五步法）
├── METHODOLOGY.md           ← 方法论深度解析（元提示词/胶水编程/Canvas/蜂群）
├── LICENSE                  ← MIT
├── template/                ← 可直接复用的项目模板
│   ├── CLAUDE.md            ← AI 代码生成规则（通用版）
│   ├── AGENTS.md            ← 同上（给 Codex/Hermes 等用）
│   └── memory-bank/
│       ├── program-design-document.md   ← 程序设计文档模板
│       ├── tech-stack.md                ← 技术栈模板
│       ├── implementation-plan.md       ← 实施计划模板
│       ├── progress.md                  ← 进度记录模板
│       ├── architecture.md              ← 架构文档模板
│       ├── vibe-coding-core.md          ← 项目规则模板
│       ├── g-updated-rules.md           ← 生成器 G 初始规则
│       └── p-history.md                 ← 提示词历史初始模板
```

## 兼容工具

本方法论**不绑定特定工具**。已在以下 AI 编程助手上验证：

- ✅ **Claude Code**（Anthropic CLI）— `CLAUDE.md` 自动加载
- ✅ **OpenAI Codex CLI** — 使用 `AGENTS.md`
- ✅ **Cursor** — 支持自定义规则文件
- ✅ **Hermes Agent** — 使用 `AGENTS.md`
- ✅ 任何支持上下文文件的 AI 助手

## 为什么有效？

### 传统 Vibe Coding 的问题

```
想法 → AI 生成代码 → 跑通了 → 继续加功能 → AI 忘了之前的上下文 → 代码越来越乱 → 推倒重来
```

### 加上 Memory Bank 后

```
想法 → 写进设计文档 → AI 读文档生成计划 → 每步生成代码+测试 → 验证通过写进 progress.md
→ 新对话开始 → AI 先读 memory-bank/ 所有文件 → 完全恢复上下文 → 继续下一步
```

**关键：** 把 AI 的"记忆"从"对话窗口"转移到"文件系统"。对话可以清空，文件不会丢失。

## License

MIT — 随便用，改了也行，标个出处就行。
