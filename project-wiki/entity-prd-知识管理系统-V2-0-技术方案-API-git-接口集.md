---
title: "API/git 接口集"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "接口"
uploadedAt: "2026-08-14T02:50:31.939Z"
---

# API/git 接口集

**类型**：接口

## 定义

统一前缀 /api/git 的 REST 接口集合，透传 project 参数实现多项目隔离，全部返回 {success,data:{...}} 信封
## 属性
- 端点:/api/git/config|init|commit|pull|push|status|log|conflicts|diff
- 分页参数:pageSize(默认20),不认limit
- 信封装约:{success,data:{items,total}}
## 关联关系
- git_adapter.py（依赖）
- script-check-skill 契约（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（2.3 新增接口）
