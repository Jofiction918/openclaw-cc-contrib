---
name: dream-rem
version: 2.2.0
description: "深度整合记忆，将 daily 日记提炼到 topic 文件，清理过时内容（定时Cron检测）/ 触发词：深度整合、梦境整理 / 命令：/dream-rem"
license: MIT
triggers:
  - 深度整合记忆
  - 梦境整理
  - 整合记忆
  - dream-rem
  - "/dream-rem"
---

# dream-rem v2.1.0 — 睡梦式记忆深度整合

定时深度整合：将分散的 daily 日记提炼合并到 topic 文件，删除过时内容，保持 `MEMORY.md` 简洁可用。

## 触发机制（v2.1.0 已修复为真正自动）

### 触发方式一：Cron 高频检测（主要）

系统有一个每 30 分钟执行的 cron 任务（ID: `c38df937-1194-4cf0-babd-0d0bd94abc7a`），该任务：
1. 读取 `memory/heartbeat-state.json`，自增 `sessionCount`（每次心跳代表一个新会话）
2. 检测是否满足整合条件：
   - `sessionCount >= 5` **且** 距 `lastDreamAt` 已过 **24 小时**，或
   - 距 `lastDreamAt` 已过 **72 小时**（强制整合，防止记忆碎片化）
3. 满足条件 → 执行 dream-rem 四阶段整合
4. 更新 `heartbeat-state.json` 的 `lastDreamAt`，重置 `sessionCount`

### 触发方式二：Heartbeat 检测（辅助）

每次 heartbeat 时，若 sessionCount 已达 5 且超过 24 小时，也触发整合。

### 触发方式三：手动触发

- 命令：`/dream-rem`

---

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

**ENTRYPATH 检查**：若 `MEMORY.md` 超过 200 行或超过 25KB，触发精简警告。

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

### Phase 4 — Prune & Index（精简索引）

1. 重写 `MEMORY.md`：
   - 每行一个指针：`- [名称](topics/文件名.md) — 一句话 hook`（≤150 字符）
   - 总行数 ≤200，总大小 ≤25KB
   - 删除过时 topic 的指针，补充新增 topic 的指针
2. 验证修改后文件可读
3. 更新 `heartbeat-state.json` 的 `lastDreamAt`

---

## 输出模板

整合完成后静默执行（cron/heartbeat），手动触发时输出：

```markdown
## 🌙 Dream 完成 · {{YYYY-MM-DD HH:MM}}

**整合范围**：{{N}}个 daily 文件
**本次耗时**：{{N}} 分钟

### 整合结果

| 类型 | 数量 | 说明 |
|------|------|------|
| 🌟 新增/更新 topic | {{N}}个 | - |
| 🗑 清理过时记忆 | {{N}}条 | - |
| 📋 MEMORY.md | {{N}}行（之前 {{M}}行） | ✅ 精简 |

### 本次主要变化
- **新增**：`topics/xxx.md`
- **更新**：`topics/ccc.md`
- **删除**：`topics/ddd.md`（过时）

### 下次整合预计
{{YYYY-MM-DD HH:MM}}（≥5会话 + ≥24小时后自动触发）
```

---

## 写入规范

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

## 状态记录

`memory/heartbeat-state.json`：
```json
{
  "lastExtraction": 1703275200,
  "lastDreamAt": 1703188800,
  "sessionCount": 7
}
```

---

## 权限要求

- `FileRead`：读取 MEMORY.md、topics/、daily 文件
- `FileWrite` / `FileEdit`：修改 topics/、MEMORY.md、`memory/heartbeat-state.json`
- `sessions_spawn`：（可选）fork 子 Agent 并行处理

## 触发词

- 自动：Cron 高频检测（每 30 分钟）
- 自动：Heartbeat 检测（辅助）
- 手动：`/dream-rem`

---

*本 Skill 基于 CC 记忆系统 autoDream 设计，适配 OpenClaw v2.1.0*
