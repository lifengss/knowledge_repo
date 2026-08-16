---
title: ".gitignore 反转规则"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "规则"
uploadedAt: "2026-08-14T02:50:31.940Z"
---

# .gitignore 反转规则

**类型**：规则

## 定义

V2.0 关键改动：.gitignore 从排除 brains/ 改为仅跟踪 brains/，使源码/运行产物不入版本
## 属性
- 反转前:第11行排除 brains/
- 反转后:* + !brains/ + !brains/** + !*.md + !.gitignore
- 目标:仅版本化知识库Markdown
## 关联关系
- Git 协同层（实现）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（2.4 .gitignore 反转（关键））
