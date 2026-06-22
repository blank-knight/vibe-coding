# {项目名} — Agent 代码生成规则

> Claude Code 自动加载此文件。Codex/Hermes 请复制为 AGENTS.md。

---

## ⚠️ ALWAYS（每次生成代码前强制执行）

1. **先读项目规则**: `memory-bank/vibe-coding-core.md`（本项目专属规则）
2. **读进度**: `memory-bank/progress.md`（了解当前进度）
3. **读架构**: `memory-bank/architecture.md`（了解项目结构）
4. **读设计文档**: `memory-bank/program-design-document.md`（了解项目目标）
5. 每完成一步 → 更新 `memory-bank/progress.md`
6. 每完成一个功能 → 更新 `memory-bank/architecture.md`

---

## 🔴 铁律

- **禁止假数据**: Mock/Stub/Demo 数据不得进入任何代码路径。先查真实数据源。
- **先读再写**: 没读 `memory-bank/` 就开始写代码 = 违规。
- **模块化**: 单文件不超过 300 行。
- **每步验证**: 代码必须测试通过才标记完成。
- **胶水优先**: 优先复用成熟开源库，不要从零造轮子。

---

## 元方法论闭环

写代码前，执行提示词进化循环：
1. 生成器 G 根据当前意图生成提示词 P
2. 优化器 O 按标准 Ω 优化为 P\*
3. 元生成器 M 更新 G 的规则到 `memory-bank/g-updated-rules.md`
4. 提示词记录到 `memory-bank/p-history.md`

---

## 项目技术栈

<!-- 在 memory-bank/tech-stack.md 中定义 -->

## 项目设计文档

<!-- 在 memory-bank/program-design-document.md 中定义 -->

---

*本文件同时充当 CLAUDE.md（Claude Code）和 AGENTS.md（Codex/Hermes）的入口。*
*Vibe Coding 方法论框架: https://github.com/axibawa/vibe-coding*
