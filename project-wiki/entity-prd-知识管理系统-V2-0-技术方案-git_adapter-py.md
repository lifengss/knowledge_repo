---
title: "git_adapter.py"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "模块"
uploadedAt: "2026-08-14T02:50:31.937Z"
---

# git_adapter.py

**类型**：模块

## 定义

Git 操作的薄封装模块，统一通过 subprocess 调用系统 git 命令，对外暴露 init/status/add_commit/pull/push/log/diff/branch/clone 等函数
## 属性
- 执行方式:subprocess
- 平台兼容:Windows shell:false
- 封装函数:git_init/git_status/git_add_commit/git_pull/git_push/git_log/git_diff/git_branch/git_clone
## 关联关系
- Git 协同层（实现）
- Dream Cycle（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（2.2 模块拆分（后端））
