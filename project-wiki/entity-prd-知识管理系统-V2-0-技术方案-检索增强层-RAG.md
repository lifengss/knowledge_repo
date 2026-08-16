---
title: "检索增强层(RAG)"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "模块"
uploadedAt: "2026-08-14T02:50:31.944Z"
---

# 检索增强层(RAG)

**类型**：模块

## 定义


## 属性
- 目录:retrieval/
- 子模块:embed.py/vector_store.py/hybrid.py/api.py
- 融合算法:RRF(Reciprocal Rank Fusion)+ 可选 rerank
- 检索模式:keyword|semantic|hybrid
## 关联关系
- 知识管理系统(KS)（属于）
- POST /api/search（实现）
- GBrain CLI（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（4. 检索增强（RAG）层）
