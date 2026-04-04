# openclaw-memory-skills
> 从 CC 逆向提取，移植到 OpenClaw 的一套完整记忆系统 Skill：**extract-memories → memory-sorting → dream (REM)**

[![license](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![openclaw](https://img.shields.io/badge/for-openclaw-%F0%9F%92%9A-blue)](https://openclaw.org/)

## 📖 简介

从 CC 源码逆向提取，移植到 OpenClaw 的一套完整记忆系统 Skill，实现 AI 助手「越用越懂你」：

```mermaid
flowchart LR
    A[extract-memories<br/>对话结束自动提炼] --> B[memory-sorting<br/>整理归纳] --> C[dream (REM)<br/>REM睡眠式整合]
    style A fill:#e1f5fe
    style B fill:#fff2e6
    style C fill:#d3eafd
```

### 🎯 核心设计

| 阶段 | Skill | 功能 | 触发 |
|------|-------|------|------|
| **增量提炼** | `extract-memories` | 对话结束自动提炼本轮对话关键记忆写入每日文件 | 对话结束自动 |
| **整理归纳** | `memory-sorting` | 扫描记忆，检测重复/过时/冲突/模糊，生成整理提案等待用户审批 | 手动触发 |
| **REM整合** | `dream-rem` | 定时将每日提炼整合为主题文件，精简入口索引 | 自动（≥ 24h + ≥ 5会话）/手动 |

## ✨ 效果

- ✅ 彻底解决「每次对话都要重新说一遍背景」痛点
- ✅ 记忆自动增长，自动整理，始终保持干净
- ✅ 按日期归档 + 按主题整合，既保留完整过程，又方便快速查找
- ✅ AI 只做提案，最终决定权始终在你手里，安全可靠

## 📦 安装

### 方法 1: OpenClaw Skill 市场（推荐）

1. 打开 OpenClaw → 技能市场 → 搜索「记忆系统」→ 安装

### 方法 2: 手动安装

```bash
mkdir -p ~/.openclaw/skills/
cp -r extract-memories ~/.openclaw/skills/
cp -r memory-sorting ~/.openclaw/skills/
cp -r dream-rem ~/.openclaw/skills/
```

## 📝 使用说明

| 操作 | 触发方式 | 说明 |
|------|----------|------|
| 提炼 | 对话结束自动 / `/extract-memories` 手动 | 自动提炼本轮对话关键记忆写入每日文件 |
| 整理 | `/memory-sorting` 手动 | 扫描所有记忆，生成整理提案，你审批后执行修改 |
| 整合 | 满足条件自动 / `/dream-rem` 手动 | 深度整合所有记忆，合并信息，删除过时，重建索引 |

## 🎯 项目信息

- 提取来源: CC 2.1.88
- 移植适配: [真维斯 @ Weavemind](https://weavemind.com)
- 许可证: MIT

---

*克隆/fork 欢迎 stars 🌟
