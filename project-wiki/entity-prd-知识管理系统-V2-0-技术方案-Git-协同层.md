---
title: "Git 协同层"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "模块"
uploadedAt: "2026-08-14T02:50:31.936Z"
---

# Git 协同层

**类型**：模块

## 定义

V2.0 新增的核心模块，为 brains/ 知识库提供 Git 版本化、团队协同、回溯与审计能力
## 属性
- 新增文件:git/git_adapter.py
- 接口前缀:/api/git
- 安全约束:仅作用于brains/目录
- 依赖:系统git命令(shell:false, PYTHONUTF8=1)
## 关联关系
- 知识管理系统(KS)（属于）
- git_adapter.py（包含）
- API/git 接口集（包含）
- Dream Cycle（依赖）
- .gitignore 反转规则（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（2. Git 协同层（V2.0 核心新增））
