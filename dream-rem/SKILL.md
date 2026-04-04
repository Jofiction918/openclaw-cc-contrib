---
name: dream-rem
version: 2.0.0
description: "深度整合记忆，将 daily 日记提炼到 topic 文件，清理过时内容 / 触发词：深度整合、梦境整理 / 命令：/dream-rem"
license: MIT
triggers:
  - 深度整合记忆
  - 梦境整理
  - 整合记忆
  - dream-rem
  - "/dream-rem"
---

# dream-rem v2.0.0 — 睡梦式记忆深度整合

定时深度整合（≥24小时 + ≥5个新会话）或手动触发 `/dream-rem`，将分散的 daily 日记提炼合并到 topic 文件，删除过时内容，保持 `MEMORY.md` 简洁可用。

## 何时使用

- **自动触发**：满足 ≥24小时 + ≥5个新会话时后台自动运行
- **手动触发**：`/dream-rem`

## 核心原则

1. **MEMORY.md = 纯索引**——不是记忆文件，每行一个指针，不超过 150 字符
2. **topic 文件 = 真实记忆**——所有记忆内容存在 `topics/` 下
3. **删除被推翻的**——不保留矛盾的两个版本，保留正确那个
4. **相对日期 → 绝对日期**——`"昨天"` → `"2026-04-04"`

---

## 四阶段执行流程

### Phase 1 — Orient（建立视野）

1. 读取 `MEMORY.md` 索引，了解当前主题覆盖情况
2. 扫描 `topics/` 目录，建立已有 topic 清单
3. 确认 `daily logs` 位置（`memory/logs/` 或 `memory/*.md`）

**ENTRYPATH 检查**：若 `MEMORY.md` 超过 200 行或超过 25KB，触发精简警告：
```
> WARNING: MEMORY.md is {{N}} lines (limit: 200). Only part was loaded.
> Keep index entries to one line under ~150 characters; move detail into topic files.
```

### Phase 2 — Gather（收集信号）

1. 扫描最近 14 天的 daily 文件（或 `logs/` 目录）
2. 识别值得提炼的新信息（对照已有 topic 清单，避免重复）
3. 识别已被推翻的旧记忆（对比 daily 新结论和 topic 旧内容）
4. 识别矛盾（同一事实在不同文件说法不一致）

### Phase 3 — Consolidate（整合执行）

对于每条值得提炼的信息：
- **有对应 topic 文件** → 合并进去（追加新内容，更新过时内容）
- **没有对应 topic** → 创建新 topic 文件（含 frontmatter）

对于被推翻的旧记忆：
- **直接删除**旧段落/旧文件，不保留矛盾版本
- 在执行报告中注明"已删除过时记忆 ×N 条"

### Phase 4 — Prune & Index（精简索引）

1. 重写 `MEMORY.md`：
   - 每行一个指针：``- [名称](topics/文件名.md) — 一句话 hook`（≤150 字符）
   - 总行数 ≤200，总大小 ≤25KB
   - 删除指向已删除/过时 topic 的指针
   - 补充新增 topic 的指针
2. 验证修改后文件可读
3. 清理 `.consolidate-lock`

---

## 输出模板（固定格式）

整合完成后，在主会话输出：

```markdown
## 🌙 Dream 完成 · {{YYYY-MM-DD HH:MM}}

**整合范围**：{{N}}个 daily 文件 · {{N}}个会话
**本次耗时**：{{N}} 分钟

### 整合结果

| 类型 | 数量 | 说明 |
|------|------|------|
| 🌟 新增/更新 topic | {{N}}个 | 新增 N 个 topic 文件，更新 M 个 |
| 🗑 清理过时记忆 | {{N}}条 | 删除被推翻/过期的记忆 |
| 📋 MEMORY.md | {{N}}行（之前 {{M}}行） | ✅ 精简 / ⚠️ 需关注 |

### 本次主要变化

- **新增**：`topics/xxx.md`、`topics/yyy.md`
- **合并**：`topics/aaa.md` → `topics/bbb.md`（内容合并）
- **更新**：`topics/ccc.md`（补充最新结论）
- **删除**：`topics/ddd.md`（过时，已被新结论推翻）

### 警告（如有）
> WARNING: MEMORY.md is {{N}} lines (limit: 200).
> 或：MEMORY.md is {{size}} (limit: 25KB).

### 下次整合预计
{{YYYY-MM-DD HH:MM}}（≥24小时 + ≥5个新会话后自动触发）
```

---

## 写入规范

**topic 文件命名**：
```
memory/topics/user_role.md
memory/topics/feedback_concise.md
memory/topics/project_deadline.md
memory/topics/reference_linear.md
```

**topic 文件格式**：
```yaml
---
name: 名称
description: 一句话描述（用于判断 relevance）
type: user / feedback / project / reference
---
正文内容

**Why:** 原因（feedback/project 必须）
**How to apply:** 适用场景（feedback/project 必须）
```

**MEMORY.md 指针格式**：
```
- [名称](topics/文件名.md) — 一句话 hook（≤150字符）
```

---

## What NOT to Save（6条禁止）

1. 代码结构/架构/文件路径——可从源码重新读取
2. Git 历史——`git log` 是权威来源
3. 调试方案——修复在代码里
4. CLAUDE.md 已有的内容——不要重复
5. 临时任务状态——属于 plan
6. PR 列表/活动摘要——改为提炼"有什么非 очевидный 值得记忆"

---

## 分布式锁

- 获取锁：创建 `.consolidate-lock`（主机名 + 时间戳）
- 锁存在：退出，报告"另有整合在进行"
- 释放：完成或中断时删除 `.consolidate-lock`

---

## 权限要求

- `FileRead`：读取 MEMORY.md、topics/、daily 文件
- `FileWrite` / `FileEdit`：修改 topics/、MEMORY.md
- `sessions_spawn`：（可选）fork 子 Agent 并行处理

## 触发词

- 自动：≥24小时 + ≥5个新会话
- 手动：`/dream-rem`

---

*本 Skill 基于 Claude Code 记忆系统 autoDream 设计，适配 OpenClaw v2.0.0*
