---
title: "多项目知识库隔离/共享"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "规则"
uploadedAt: "2026-08-14T02:50:31.951Z"
---

# 多项目知识库隔离/共享

**类型**：规则

## 定义


## 属性
- 配置:projects.json
- 数据库:drafts.db
- 私有库路径:brains/<project>/
- 共享库路径:_shared_brain/
- 实现文件:api/projects.py, cache/draft_cache.py
## 关联关系
- Git 协同层（依赖）
- API/git 接口集（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（0. 现状基线（V1.x 已实现））
