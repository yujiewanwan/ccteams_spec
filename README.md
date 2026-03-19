# Flow Spec Suite

Spec-driven development workflow for Claude Code. Four commands from idea to merge.

> **中文版说明见下方** / Chinese version below

---

## Why not open-spec / spec-kit?

| | open-spec / spec-kit | flow-spec |
|---|---|---|
| Era | Early 2023 | 2024+ |
| Core Assumption | AI needs strong constraints | AI is capable, needs freedom |
| Alignment | Template-based, one-way | Three agents debate in parallel |
| Development | Human writes code | Agent-teams fully automated |
| Best For | Conservative enterprises | Efficiency-focused teams |

## Quick Start

```bash
# 1. Create spec (human alignment)
/flow:new
# → PM + Architect + QA agents ask questions in parallel
# → Generates docs/specs/SPEC-001/spec.yaml

# 2. Develop (fully automated)
/flow:continue        # Start agent-teams (FE/BE/QA/Review)
/flow:continue        # Auto test, then human verification
/flow:continue        # Confirm human testing done

# 3. Finish
/flow:sync            # Update docs, output PR checklist
```

**Status flow:** `draft → approved → in-progress → human-testing → done`

## Commands

| Command | What it does |
|---------|--------------|
| `/flow:new` | Three agents align in parallel → generate spec |
| `/flow:continue` | Auto-execute next step based on spec.yaml status |
| `/flow:sync` | Update docs, generate PR checklist |
| `/flow:ff` | Fast-forward mode (when all ACs are automated) |

## Artifacts

```
docs/specs/<SPEC-ID>/
├── requirement.md    # Requirements (human-readable)
├── test-cases.md     # Test cases (human-readable)
└── spec.yaml         # Machine contract (drives workflow)
```

## Installation

```bash
# Claude Code Plugin
/plugin marketplace add your-username/flow-spec-suite
/plugin install flow-spec-suite@marketplace-name

# Or manual
cp -r skills/* .claude/skills/
```

---

## 中文说明

面向 Claude Code 的 spec-driven 开发工作流。四个命令从需求到合并。

### 与 open-spec / spec-kit 的区别

| | open-spec / spec-kit | flow-spec |
|---|---|---|
| 设计年代 | 2023 早期 | 2024+ |
| 核心假设 | AI 需要强约束才能做对 | AI 能力足够，需要释放空间 |
| 对齐方式 | 模板填空，单向输出 | 三角色 Agent 并行辩论 |
| 开发方式 | 人类写代码 | Agent-teams 全自动 |
| 适用场景 | 保守型企业 | 追求效率的小团队 |

### 快速开始

```bash
# 1. 创建 spec（人工对齐）
/flow:new
# → PM + 架构师 + QA 三个 Agent 并行提问
# → 生成 docs/specs/SPEC-001/spec.yaml

# 2. 开发（全自动）
/flow:continue        # 启动 agent-teams（FE/BE/QA/Review）
/flow:continue        # 自动测试，通过后进入人工验收
/flow:continue        # 完成人工测试确认

# 3. 收尾
/flow:sync            # 更新文档，输出 PR checklist
```

**状态流转：** `draft → approved → in-progress → human-testing → done`

### 命令说明

| 命令 | 作用 |
|---------|--------------|
| `/flow:new` | 三角色并行对齐 → 生成 spec |
| `/flow:continue` | 根据 spec.yaml 状态自动执行下一步 |
| `/flow:sync` | 更新文档，生成 PR checklist |
| `/flow:ff` | 快进模式（全自动化时一键到底） |

### 产物

```
docs/specs/<SPEC-ID>/
├── requirement.md    # 需求文档（人工可读）
├── test-cases.md     # 测试用例（人工可读）
└── spec.yaml         # 机器契约（驱动工作流）
```

### 安装

```bash
# Claude Code 插件
/plugin marketplace add your-username/flow-spec-suite
/plugin install flow-spec-suite@marketplace-name

# 或手动安装
cp -r skills/* .claude/skills/
```

## License

MIT
