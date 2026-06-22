---
name: ai-coding-memory-bank-sdd-workflow
description: Use when starting or iterating an AI coding project and you need a repeatable workflow combining Memory Bank, SDD reviews, and G/O/M prompt evolution.
version: 1.1.0
license: MIT
metadata:
  tags: [ai-coding, workflow, memory-bank, sdd, gom, planning, execution]
  related_skills: [writing-plans, subagent-driven-development, test-driven-development]
---

# AI Coding Workflow: Memory Bank + SDD + G/O/M

## Overview

This skill defines a practical execution protocol for AI coding projects:

1) Create CLAUDE.md + Memory Bank first
2) Choose execution mode by risk (fast MVP vs reviewed delivery)
3) Run implementation with structured verification gates
4) Update rules and docs after each milestone (G/O/M loop)

Core principle: **No Memory Bank, no coding.**

## 🚀 Quick Start — 3 步上手（不用理解 G/O/M）

> 新手或小项目？从这里开始。理解了再去看后面的完整流程。

```bash
# Step 1: 初始化 memory-bank（一键复制）
#   从公开 repo 获取模板
git clone https://github.com/blank-knight/vibe-coding.git /tmp/vc
cp -r /tmp/vc/template/* 你的项目/

# Step 2: 编辑 CLAUDE.md 写入项目规则
# 最少填这三块：
#   - 技术栈（用什么框架/语言/数据库）
#   - 禁止行为（不准碰哪些文件、不准用什么库）
#   - 常用命令（怎么启动、怎么测试）

# Step 3: 启动你的 AI 编程工具执行
cd 你的项目
# 用 Claude Code / Codex / Cursor / 任何 AI 助手
# 先让它读 memory-bank/ 所有文件，再按计划执行
```

**就这三步。** 后面的 G/O/M、SDD、双路径是进阶内容，等遇到痛点再看。

### 什么时候需要看完整流程？

- 项目会长期维护 → 看跨会话恢复协议
- 代码涉及钱（交易系统、支付） → 看 Path B 质量优先
- 每次重写 AI prompt 都很痛苦 → 看 G/O/M 闭环
- 多个 AI Agent 协作 → 看 Skill 生态整合

## CLAUDE.md 规则体系

CLAUDE.md 和 memory-bank 是配合关系，不是替代关系：

- **CLAUDE.md** = 规则约束（禁止假数据、先读再写、模块化、每步验证）— AI"应该"遵守
- **memory-bank/** = 项目状态（进度、架构、计划）— AI 读写的工作区

### 加载机制

| 层级 | 文件 | 何时加载 |
|------|------|---------|
| 通用铁律 | `~/CLAUDE.md`（或工作区根目录） | 会话启动时加载到 system prompt |
| 项目规则 | `项目/CLAUDE.md` | 访问项目文件时按需加载 |
| 跨会话记忆 | MEMORY.md | 每次会话注入 |

### CLAUDE.md / AGENTS.md / .cursorrules 关系

**竞争关系，不是配合。** 同一目录下优先级链 first found wins：
`.hermes.md > AGENTS.md > CLAUDE.md > .cursorrules`

**只用 CLAUDE.md，不要同时放 AGENTS.md。**

## When to Use

Use when:
- Starting a new project with Claude Code/Codex/Cursor/any AI coding assistant
- Rebooting work after context loss or session change
- Moving from prototype to production quality
- You want stable, repeatable project delivery

Do not use for:
- Tiny throwaway scripts that will not be maintained
- One-command experiments with no follow-up

## Stage 0: Requirement Framing

Before generating code, pin down:
- Objective: what outcome must exist
- Assets: existing repos/APIs/data/docs
- Constraints: budget, timeline, stack, compliance
- Exclusions: what is explicitly out of scope

Output: short scope note (can be merged into program-design-document.md).

## Stage 1: Memory Bank Bootstrapping (Mandatory)

### 快速初始化（推荐）

```bash
git clone https://github.com/blank-knight/vibe-coding.git /tmp/vc
cp -r /tmp/vc/template/* 你的项目/
```

模板会生成：
```
你的项目/
├── CLAUDE.md                      # 项目专属规则（编辑这个）
└── memory-bank/
    ├── progress.md                # 进度追踪
    ├── vibe-coding-core.md        # G/O/M 框架定义
    ├── program-design-document.md # 设计文档
    ├── tech-stack.md              # 技术栈
    ├── implementation-plan.md     # 实施计划
    ├── architecture.md            # 架构
    ├── g-updated-rules.md         # 生成器迭代
    └── p-history.md               # 提示词历史
```

Rules:
- `implementation-plan.md` contains steps only, no code
- Step granularity: typically 2-5 minutes of focused work
- Every step has explicit input/output paths and a verification command

## 📚 Documentation Freshness Check（防废弃 API）

AI 模型的训练数据有截止日期，写出来的 API 用法可能已经过时。在每个 Phase 开始前：

1. **检查框架版本**：确认 package.json / pyproject.toml 里的版本是当前稳定版
2. **查最新文档**：如果涉及框架核心 API（React hooks、FastAPI 路由、TailwindCSS 类名等），让 AI 先查官方文档再写代码
3. **在计划里加一行**：
   ```
   写代码前先确认框架版本，如果不确定 API 是否最新，
   用 web_search 查官方文档确认。
   ```
4. **CLAUDE.md 里固化**：在技术栈部分标注版本号，如 `FastAPI >=0.104` 而非 `FastAPI`，防止 AI 用旧版写法

**高频踩坑**：Next.js App Router vs Pages Router、TailwindCSS v3 vs v4 配置、FastAPI 0.100+ 的 lifespan 变更。

## Stage 2: Choose Execution Path

### Shared Controller: Mother Prompt Template

Both Path A and Path B use the same mother prompt as the top-level controller.

**Mother prompt template location:** `templates/mother-prompt-template.md`

Role of mother prompt:
- Defines project principles and guardrails
- Anchors memory-bank discipline
- Controls phase outputs and reporting format

### Path A: Fast MVP (Big-Bang)

Use when speed matters more than strict controls.

Flow:
1. Fill the mother prompt template with project specifics
2. Ask coding agent to read all memory-bank docs first
3. Execute phased build quickly
4. Post-implementation: reconcile docs with actual code

Tradeoff: fast output, higher risk of hidden defects.

### Path B: Quality-First (SDD + Mother Prompt)

Use when reliability matters (production systems, money-impacting bots, long-lived code).

Path B still uses the mother prompt, but adds strict review gates per task.

Per task pipeline:
1. Implementer subagent executes task (under mother-prompt principles)
2. Spec reviewer checks exact plan compliance (PASS required first)
3. Quality reviewer checks maintainability/security/tests
4. If issues: fix -> re-review loop
5. Mark task complete

## Stage 3: Milestone G/O/M Loop

After each phase, choose your G/O/M depth based on project needs:

### G/O/M 三级模式（按需选择）

| 级别 | 适用场景 | 做什么 | 时间开销 |
|------|---------|--------|---------|
| **Fast（跳过）** | 脚手架、一次性实验 | 不走 G/O/M，直接写 prompt 执行 | 0 |
| **Standard（推荐默认）** | 大部分项目 | G 生成 prompt → O 检查是否精确、有无歧义 → 执行。M 记录好用的 prompt 模式到 p-history.md | +5分钟/Phase |
| **Strict（完整 Ω 审计）** | 涉及钱、安全、生产系统 | G → O 按 6 维 Ω 清单逐条审计 → M 更新 g-updated-rules.md → 写 p-history.md | +15分钟/Phase |

**80% 的项目用 Standard 就够。** Strict 留给真正需要零缺陷的场景。

### Standard 模式操作（推荐）

```
1. G：从 implementation-plan.md 取当前步骤，生成执行 prompt
2. O：检查 prompt 是否——
   - 意图明确（不是"做个登录页面"而是"实现JWT登录，用/auth/login路由"）
   - 有验证标准（怎么算完成）
   - 标注了不准碰的文件
3. 执行 → 验证 → 更新 progress.md
4. M：如果这次 prompt 效果好，把模式记到 p-history.md
```

### Strict 模式 — 完整 Ω Checklist

1. **G (Generator):** produce task prompt from plan + context
2. **O (Optimizer):** improve prompt quality against Ω criteria
3. **M (Meta-generator):** update generator rules in `g-updated-rules.md`

Update files every milestone:
- `progress.md`: what is done, blockers, verification status
- `architecture.md`: actual file/module responsibilities
- `p-history.md`: final optimized prompts used for execution

## Ω Prompt Quality Checklist

A prompt is acceptable only if it is:
- Precise: no ambiguous intent
- Complete: explicit inputs/outputs/constraints
- Verifiable: has concrete pass/fail checks
- Scoped: states what must not be changed
- Context-anchored: references required memory-bank docs
- Anti-regression: includes checks for existing behavior

## Stage 4: Release Gate

Before claiming completion:
- Run full test suite
- Run lint/type checks as applicable
- Validate critical user path manually
- **Ensure no mock/stub/demo logic leaked into production path**

## 🔗 Skill 生态整合（多 Skill 协作）

本 skill 不孤立运行。根据执行路径，串联其他工具：

| 阶段 | 用什么 | 怎么触发 |
|------|-------|---------|
| 写计划 | writing-plans / plan skill | 大需求拆解成 bite-sized tasks |
| 写代码（Path B） | TDD skill | 先写测试再实现（RED-GREEN-REFACTOR） |
| 调 Bug | systematic-debugging | 4 阶段根因分析，不要盲改 |
| 代码审查 | code-review skill | 提交前安全扫描+质量门禁 |
| 提交代码 | github-pr-workflow | 分支→commit→PR→CI→merge |
| 探索性验证 | spike skill | 不确定方案可行性？先做个 throwaway 实验验证 |
| 简化重构 | simplify-code skill | 3-agent 并行清理近期代码变更 |

### 典型串联流程（Path B 完整版）

```
writing-plans (拆计划)
  → test-driven-development (写测试)
    → AI Agent (实现)
      → systematic-debugging (修Bug)
        → code-review (审查)
          → github-pr-workflow (提交PR)
```

**在 implementation-plan.md 的每个步骤里标注用哪个 skill**，AI 执行时会自动加载。

## 📦 Git 工作流集成

Release Gate 通过后，规范提交：

```bash
# 1. 规范化提交
git add -A
git commit -m "feat: [Phase X] 具体功能描述

- 实现了 XXX
- 修复了 XXX
- 通过测试: pytest tests/"

# 2. 推送+PR（如果用 GitHub）
git push origin feature/phase-X

# 3. 打 Tag（里程碑用）
git tag -a v0.X.0 -m "Phase X complete"
```

### Git 提交规范

| 前缀 | 含义 |
|------|------|
| `feat:` | 新功能 |
| `fix:` | 修 Bug |
| `refactor:` | 重构（不改行为） |
| `docs:` | 只改文档 |
| `test:` | 只改测试 |
| `chore:` | 杂项（依赖、配置） |

**每个 Phase 结束后必须 commit + 更新 progress.md。** 不要攒一堆改动一次性提交。

## Decision Heuristic (A vs B)

Prefer Path A when:
- Idea validation / prototype
- Low blast radius if broken
- Need speed this week
- Review gates: post-implementation reconciliation only

Prefer Path B when:
- Production deployment
- Financial or user-impacting behavior
- Multi-person maintenance expected
- Review gates: per-task spec + quality checks

## Cross-Session Handoff Protocol (Mandatory)

The #1 pain point in AI coding is losing context between sessions. This protocol eliminates it:

**On session start** — when you say "continue XXX" / "继续做 XXX":
1. Read `XXX/memory-bank/progress.md` first (current status, what's done, what's next)
2. Read `XXX/CLAUDE.md` for project rules
3. Read `XXX/memory-bank/implementation-plan.md` for the plan
4. If no `memory-bank/` exists, read `XXX/MEMORY.md` instead
5. Summarize the current state back before doing anything — confirm you're picking up where things left off

**On session end** — when you say "save" / "存档" / "update memory":
1. Update `memory-bank/progress.md` with: what changed this session, where code stopped, known issues, next steps
2. If no `memory-bank/` exists, update `XXX/MEMORY.md` instead
3. Always include a **📌 Current Status** section at the top

**This is not optional. A session that ends without updating progress.md guarantees the next session starts blind.**

## Executing Phases via AI Agent (Standard Pattern)

For each phase, write the phase prompt to a temp file and pipe it in:

```bash
# 1. Write phase prompt to file (avoids shell quoting issues)
cat > /tmp/phase-N-prompt.txt << 'EOF'
Read all memory-bank/ files first.
Then implement Phase N: <description>
...
EOF

# 2. Execute with your AI tool (example: Claude Code)
cat /tmp/phase-N-prompt.txt | claude --max-turns 50 2>&1 | tee /tmp/phase-N-output.log

# 3. Verify — check files exist FIRST
ls -la src/   # confirm files were created

# 4. Update progress.md
```

**After each phase:** update `memory-bank/progress.md` to mark it complete before starting the next.

## Common Pitfalls

1. Starting code before CLAUDE.md + Memory Bank is complete
2. Turning plan steps into broad vague tasks
3. Running quality review before spec-compliance review
4. Skipping re-review after fixes
5. Leaving docs stale after big-bang execution
6. Treating mother prompt as one-shot instead of phased controller
7. Incorrectly assuming Path B replaces the mother prompt — it does not; it adds review gates on top of it
8. **Assuming pre-flight hang = task failed** — AI tools may hang for minutes while the actual task runs fine. Always check whether output files exist before concluding failure.
9. **Not pre-installing venv deps** — Pre-create a venv and install known deps before AI runs
10. **Naming mismatches between generated files** — Always run full import verification after each phase
11. **CSS `h-full` on flex containers squeezes expandable children** — use `min-h-0` on the outer div
12. **Python dict.get("key", default) returns None when value IS None** — `float(item.get("confidence") or 0.0)`
13. **在同一目录同时放 AGENTS.md 和 CLAUDE.md** — 优先级链 first found wins，只会读一个。只用 CLAUDE.md。
14. **编造 Mock 数据** — CLAUDE.md 铁律 #1 明确禁止。必须查数据库/API 取真实数据。没有数据就显示空状态。
15. **用废弃 API 写代码** — AI 训练数据可能过期。写框架核心 API 前必须确认版本。
16. **从不 commit** — 每完成一个 Phase 必须 commit + 更新 progress.md。攒 5 个 Phase 一次性提交 = 无法回滚。
17. **G/O/M 强迫症** — 不是每个项目都需要完整 Ω 审计。Fast 模式跳过 G/O/M 是完全合理的。
18. **孤立使用本 skill** — 不串联 TDD / debugging / code-review / git = 浪费生态。

## Verification Checklist

- [ ] CLAUDE.md exists at project root
- [ ] Memory Bank files exist (至少 progress.md)
- [ ] Implementation plan has verifiable step-level tasks
- [ ] Chosen path (A/B) documented with reason
- [ ] For B path, each task has spec + quality review evidence
- [ ] Milestone updates applied to progress/architecture/p-history/rules
- [ ] Final release gate passed
- [ ] Git commit made for each completed Phase
- [ ] Framework versions checked against latest stable
- [ ] G/O/M depth explicitly chosen (Fast/Standard/Strict)

## Support Files

- `templates/mother-prompt-template.md` — generic mother prompt template
- `templates/saas-mother-prompt-zh.md` — Chinese SaaS variant (订阅制, payment integration, PWA)
- `references/china-saas-stack.md` — Chinese market stack: payment, data APIs, PWA tradeoffs, pricing
- `references/data-pipeline-audit-trail.md` — Audit trail pattern for crawler/LLM pipelines
- `references/integration-testing-pitfalls.md` — Real bugs caught during integration testing
