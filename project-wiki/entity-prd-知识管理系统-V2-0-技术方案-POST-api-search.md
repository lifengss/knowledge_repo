---
title: "POST /api/search"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "接口"
uploadedAt: "2026-08-14T02:50:31.945Z"
---

# POST /api/search

**类型**：接口

## 定义


## 属性
- 请求参数:query, mode, limit, project, rerank
- 响应字段:items[{content,score,source}], total
- embedding兼容:OpenAI-style /v1/embeddings 或 none 回退 keyword
## 关联关系
- retrieval/hybrid.py（依赖）
- retrieval/embed.py（依赖）
- retrieval/vector_store.py（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（4.3 接口扩展）
