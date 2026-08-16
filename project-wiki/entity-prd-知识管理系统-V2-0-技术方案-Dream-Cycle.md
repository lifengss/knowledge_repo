---
title: "Dream Cycle"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "流程"
uploadedAt: "2026-08-14T02:50:31.941Z"
---

# Dream Cycle

**类型**：流程

## 定义


## 属性
- 脚本:scripts/dream-cycle-cron.sh
- 步骤:git pull(可选)→ gbrain sync/embed → alert_monitor → cleanup_stale_drafts
- 容错原则:git pull 失败仅告警不中断
- 依赖前置:不依赖 git pull 在线
## 关联关系
- git_adapter.py（依赖）
- GBrain CLI（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（3. Dream Cycle 复用 Git 能力（失败不中断））
