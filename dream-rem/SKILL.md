---
name: dream-rem
version: 1.0.1
description: "定时深度整合记忆，自动合并新信息删除过时内容 / 触发词：深度整理记忆、梦境整理、整合记忆 / 命令：/dream-rem"
license: MIT
triggers:
  - 深度整理记忆
  - 梦境整理
  - 整合记忆
  - dream-rem
  - "/dream-rem"
---

# dream (REM) — 睡梦中的记忆整理

定时触发深度整合记忆（默认：≥ 24 小时 + ≥ 5 个新会话），执行四阶段整理：
1. **Orient** → 扫描现有记忆文件，建立当前记忆索引
2. **Gather** → 从 daily 文件中收集新信息，发现与旧记忆矛盾的地方
- **Consolidate** → 合并新信息到主题文件，删除过时，修正矛盾
4. **Prune & Index** → 精简入口索引，保持文件大小合理

## 触发条件

- 自动触发：满足 `≥24 小时` + `≥5 个新会话` → 后台自动运行
- 手动触发：输入 `/dream` 强制触发一次深度整理

## 工作流程

### 前置条件
- 需要 `extract-memories` + `memory-review` 正常工作
- 需要分布式锁 `.consolidate-lock` 防止多设备同时整理冲突

### 四阶段执行

#### 1. Phase 1 — Orient
- `ls memory/` 列出所有记忆文件
- 读取入口索引 `memory/ENTRYPATH.md`
- 扫描所有主题文件，理解当前记忆结构

#### 2. Phase 2 — Gather
- 扫描 `memory/YYYY-MM-DD.md`  daily 文件
- 查找：新信息、与旧记忆矛盾的信息、需要合并的信息
- 不做全量读取，只找怀疑有变化的地方

#### 3. Phase 3 — Consolidate
- 将新信息合并到对应主题文件
- 删除被推翻的旧结论
- 相对日期转绝对日期（yesterday → 2026-04-02）
- 如果是新增主题，创建新文件

#### 4. Phase 4 — Prune & Index
- 更新入口索引 `ENTRYPATH.md`
- 保持索引 ≤ 25KB，条目精简，每个条目一行，只放标题和链接
- 删除指向过时记忆的指针

## 输出

整理完成后输出统计：
- 合并更新 × 条
- 删除过时 × 条
- 新增文件 × 个
- 索引现在 × 条，大小 × KB

## 权限要求
- 需要 `FileRead`、`FileEdit`、`FileWrite`
- 需要 `sessions_spawn` fork 子 Agent

## 参考
本 Skill 基于 CC 源码 `autoDream` 模块设计，适配 OpenClaw 记忆系统。
