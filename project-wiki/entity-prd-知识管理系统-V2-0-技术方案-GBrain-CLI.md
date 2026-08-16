---
title: "GBrain CLI"
type: entity
source: "prd-知识管理系统-V2-0-技术方案"
entityType: "模块"
uploadedAt: "2026-08-14T02:50:31.942Z"
---

# GBrain CLI

**类型**：模块

## 定义

V2.0 新增的 Python 命令行工具，提供 sync/embed/export 子命令实现 brains/ 与 GBrain 存储的同步、向量化与快照导出
## 属性
- 子命令:sync/embed/export
- 依赖:ai/ai_config.py 的 storage 段、retrieval/embed.py
- 部署依赖:pip install 且在 PATH,否则 cron 静默跳过
## 关联关系
- Dream Cycle（触发）
- 检索增强层(RAG)（依赖）

## 出处

> 来源文档：知识管理系统-V2.0-技术方案（3.2 GBrain CLI（V2.0 新增））
