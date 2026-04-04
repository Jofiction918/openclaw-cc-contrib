---
name: extract-memories
version: 2.0.0
description: "对话结束后自动提炼关键记忆到 topic 文件 / 触发词：提炼记忆、提取记忆 / 命令：/extract-memories"
license: MIT
triggers:
  - 提炼记忆
  - 提取记忆
  - extract-memories
  - "/extract-memories"
---

# extract-memories v2.0.0 — 对话记忆提炼

对话结束后自动分析本轮对话，将值得持久化的信息写入 `memory/topics/` 下的独立 topic 文件，同时更新 `MEMORY.md` 索引。

## 何时使用

- **自动触发**：每次对话结束自动后台执行
- 手动触发：`/extract-memories`

## 核心概念

**MEMORY.md = 索引，不是记忆文件。**

```
memory/
├── MEMORY.md              ← 纯索引（一行一个指针，不含记忆内容）
└── topics/               ← 所有记忆文件
    ├── user_role.md
    ├── feedback_concise.md
    ├── project_deadline.md
    └── reference_xxx.md
```

## 四种记忆类型

| 类型 | 什么时候存 | 正文结构 |
|------|-----------|---------|
| `user` | 学到用户角色/偏好/知识时 | 一段文字即可，无强制结构 |
| `feedback` | 用户纠正你或确认你做对了 | **规则本身 → Why:（原因）→ How to apply:（何时适用）** |
| `project` | 学到项目截止/动机/约束时 | **事实 → Why:（动机）→ How to apply:（如何影响工作）** |
| `reference` | 学到外部系统指针时 | URL/路径 + 用途说明 |

### body 结构示例

**feedback 示例**：
```markdown
集成测试必须用真实数据库，不能用 mock。
**Why:** 上次 mock 测试通过了，但 prod 迁移时才发现行为不一致，导致故障。
**How to apply:** 任何涉及数据库的测试优先用真实实例而非 mock 对象。
```

**project 示例**：
```markdown
非关键 PR 合并冻结：2026-04-05 起，所有非关键 PR 暂停合并。
**Why:** 移动端团队需要从 release 分支切出，周五前完成代码冻结。
**How to apply:** 评估任何计划中的 PR 是否属于"非关键"，若是则延后。
```

---

## 输出模板（固定格式）

提炼完成后，在主会话输出：

```markdown
## {{YYYY-MM-DD}} 记忆提炼

提炼结果：N条
写入位置：memory/topics/

### user
- [名称]: description（用途：用于 relevance 匹配）
  body

### feedback
- [名称]: description
  body
  **Why:** 原因
  **How to apply:** 何时适用

### project
- [名称]: description
  body（含绝对日期）
  **Why:** 动机
  **How to apply:** 如何影响工作

### reference
- [名称]: description
  URL/路径 + 用途说明

---

**本次提炼规则**：
- 只用最近 N 条对话，不额外调查/grep/查源码
- 优先更新已有 topic 文件，不新建重复记忆
- 按主题组织，不按时间顺序
- 不保存代码结构/文件路径/Git历史/调试方案/CLAUDE.md已有内容/临时状态
- 使用记忆时：引用文件/函数/配置前必须验证当前仍存在
- 记忆是时间点观察，不是实时状态——引用前需验证是否仍准确
```

---

## 写入规范（两步写入）

**Step 1**：写 topic 文件到 `memory/topics/`（APPEND 模式，不覆盖已有内容）

文件命名：`memory/topics/[type]_[slug].md`

**Step 2**：在 `MEMORY.md` 末尾追加一行指针

格式：`- [名称](topics/文件名.md) — 一句话 hook`（不超过 150 字符）

---

## frontmatter 格式（必须包含）

```yaml
---
name: 一句话名称
description: 一行描述（用于判断 relevance，决定未来是否调取这条记忆）
type: user / feedback / project / reference
---
正文内容
```

**name 命名规范**：
- `user_role.md` — 用户身份/角色
- `feedback_[主题].md` — 用户偏好和纠正
- `project_[项目/主题].md` — 项目上下文
- `reference_[系统名].md` — 外部系统指针

---

## What NOT to Save（6条禁止）

即使用户明确要求保存以下内容，也要先询问"这个值得保存吗"：

1. **代码结构/架构/文件路径**——可从源码重新读取
2. **Git 历史**——`git log` / `git blame` 是权威来源
3. **调试方案/fix 配方**——修复方案在代码里，commit message 有上下文
4. **CLAUDE.md / AGENTS.md 已有的内容**——不要重复
5. **临时任务状态**——属于 plan，不属于记忆
6. **即使用户要求也不保存**：PR 列表、活动摘要——改为问"有什么非 очевидный（不明显的）值得记忆"

---

## 文件结构

```
memory/
├── MEMORY.md              ← 纯索引（一行一个指针）
└── topics/
    ├── user_role.md
    ├── feedback_concise.md
    ├── project_deadline.md
    ├── reference_linear.md
    └── ...
```

topics/ 目录初始为空，由本 skill 和 dream-rem 逐步填充。

---

## 权限要求

- `FileRead`：读取对话上下文
- `FileWrite` / `FileEdit`：写入 `memory/topics/` 和 `MEMORY.md`
- `sessions_spawn`：（可选）fork 子 Agent

## 触发词

- 自动：对话结束（skill 内置触发逻辑）
- 手动：`/extract-memories`

---

*本 Skill 基于 Claude Code 记忆系统 extractMemories 设计，适配 OpenClaw v2.0.0*
