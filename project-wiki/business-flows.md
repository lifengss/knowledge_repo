# 业务流程与依赖知识图谱

> 版本 1.0 · 生成于 2026-08-16 · 来源 


## 一、业务域

| 域 | 含义 | 颜色 |
|----|------|------|
| git | git | #6366f1 |
| api | api | #10b981 |
| architecture | architecture | #f59e0b |
| engineering | engineering | #ef4444 |
| backend | backend | #0ea5e9 |
| frontend | frontend | #a855f7 |
| data | data | #14b8a6 |
| page_prd_知识管理系统_v2_0_技术方案 | page_prd_知识管理系统_v2_0_技术方案 | #ec4899 |
| infra | infra | #84cc16 |
| sync | sync | #f97316 |
| embedding | embedding | #6366f1 |

## 二、业务步骤节点（57）

| 步骤 | 域 | 接口 | 角色 | 摘要 | 产出 | 前置资源 |
|------|----|------|------|------|------|----------|
| **sc_git_status_commit** Git 工作区提交 | git | `—` | scenario | 查看工作区状态，提交变更并记录提交历史 | — | — |
| **api_git_status** 查看工作区状态 | git | `/api/git/status` | endpoint | 查询 Git 工作区当前变更状态 | envelope_contract | project_param |
| **api_git_commit** 提交变更 | git | `/api/git/commit` | endpoint | 将变更提交并生成 commit hash | envelope_contract | project_param |
| **api_git_log** 查询提交历史 | git | `/api/git/log` | endpoint | 查询提交历史记录，分页参数 pageSize 默认 20 | envelope_contract | project_param、pagination_contract |
| **project_param** project 参数 | git | `—` | parameter | 多项目隔离参数 | — | — |
| **envelope_contract** 信封装约 | git | `—` | contract | 统一返回格式 {success,data:{items,total}} | — | — |
| **pagination_contract** 分页参数 pageSize | git | `—` | contract | 分页参数 pageSize，默认 20，不认 limit | — | — |
| **api_git_pull** 拉取远程更新接口 | git | `/api/git/pull` | interface | 执行 git pull --rebase，透传 project 参数拉取远程更新 | application/json、{success,data} 信封 | project 参数 |
| **api_git_conflicts** 冲突检测接口 | git | `/api/git/conflicts` | interface | 检测 pull --rebase 后是否存在合并冲突 | application/json、{success,data} 信封 | project 参数 |
| **api_git_push** 推送本地提交接口 | git | `/api/git/push` | interface | 无冲突时将本地提交推送到远程仓库 | application/json、{success,data} 信封 | project 参数 |
| **git_adapter** Git 适配器 | git | `git_adapter.py` | dependency | 底层 Git 操作适配模块，被 /api/git/* 接口依赖 | — | — |
| **script_check_skill** 脚本检查技能契约 | git | `script-check-skill` | contract | script-check-skill 契约，约束接口行为与校验规则 | — | — |
| **api_git_pull** Git 拉取/合并 | api | `/api/git/pull` | endpoint | 执行 pull 或合并操作，可能产生冲突 | — | — |
| **api_git_conflicts** 冲突文件列表 | api | `/api/git/conflicts` | endpoint | 列出当前冲突文件 | — | — |
| **api_git_diff** 单文件差异 | api | `/api/git/diff` | endpoint | 查看单个冲突文件的 diff | — | — |
| **api_git_commit** 提交 | api | `/api/git/commit` | endpoint | 冲突解决后提交变更 | — | — |
| **git_collaboration_layer** Git 协同层 | architecture | `按 docs/API-INTERFACE-DOC.md 定义` | V2.0 新增架构层 | 为 KS 补齐 git pull/status/push/commit/log/conflicts/branch/clone/init 等真实 Git 能力，解决 KS 完全无 Git 能力的缺口。 | git_operations、conflict_resolution、branch_management | — |
| **retrieval_augmentation_layer** 检索增强层 | architecture | `按 docs/API-INTERFACE-DOC.md 定义` | V2.0 新增架构层 | 在 V1.x keyword 检索基础上补充 semantic 检索、embedding、rerank 与向量库能力。 | semantic_search、reranked_results | embeddings |
| **dream_cycle** Dream Cycle | architecture | `—` | 复用 Git 能力的核心流程 | 知识沉淀自动化循环，V2.0 通过复用 Git 协同层完成同步、提交与入库。 | knowledge_sync、automated_commit | git_operations |
| **dream_cycle_cron** Dream Cycle 定时脚本 | engineering | `—` | 调度入口 | 定时触发 Dream Cycle 的执行脚本，现有步骤包含 gbrain sync 与 gbrain embed。 | sync_job、embed_job | — |
| **gbrain_sync** gbrain sync | engineering | `—` | Dream Cycle 执行步骤 | 原 gbrain sync 步骤因 gbrain 不是真实 CLI 会失败，V2.0 改由 Git 协同层实现。 | sync_result | git_pull、git_status、git_push |
| **gbrain_embed** gbrain embed | engineering | `—` | Dream Cycle 执行步骤 | 原 gbrain embed 步骤，V2.0 接入检索增强层生成并写入 embedding。 | embeddings | vector_store |
| **ks_backend** KS 后端 | backend | `/api/*` | V1.x 基线 / V2.0 接入方 | 知识管理系统后端服务，V2.0 接入 Git 协同层与检索增强层。 | rest_api | git_operations、search_results |
| **ks_frontend** KS 前端 | frontend | `—` | V1.x 基线 | 知识管理系统前端，调用 KS 后端 REST 接口并维护 api-docs-data.js。 | ui_interaction | rest_api |
| **api_interface_doc** API 接口文档 | engineering | `—` | 单一数据源 | 定义 V2.0 接口契约，与 web/src/api-docs-data.js 共同作为唯一来源。 | api_contract | — |
| **script_check_skill** script-check-skill | engineering | `—` | 校验工具 | 校验脚本文件与 V2.0 技术方案及 API 接口文档的一致性。 | consistency_report | api_contract、tech_spec |
| **search_cache** 检索缓存 | data | `—` | V1.x 基线 | keyword 模式已实现，semantic 模式仅有占位。 | keyword_search_result | query |
| **quality_gate** 质量门控 | backend | `—` | V1.x 基线 | 总分 ≥ 60 入库，短响应/低可信度拒收。 | quality_score | generated_cases |
| **page_prd_知识管理系统_v2_0_技术方案** 知识管理系统-V2.0-技术方案 | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_0_现状基线_v1_x_已实现** 0. 现状基线（V1.x 已实现） | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_1_总体架构_v2_0** 1. 总体架构（V2.0） | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增** 2. Git 协同层（V2.0 核心新增） | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_2_1_设计动机** 2.1 设计动机 | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_2_2_模块拆分_后端** 2.2 模块拆分（后端） | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_2_3_新增接口_api_server_js_docs_api_interface_doc_md** 2.3 新增接口（`api/server.js` + `docs/API-INTERFACE-DOC.md`） | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_2_4_gitignore_反转_关键** 2.4 `.gitignore` 反转（关键） | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_2_5_前端集成_v2_0_新增页面** 2.5 前端集成（V2.0 新增页面） | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_3_dream_cycle_复用_git_能力_失败不中断** 3. Dream Cycle 复用 Git 能力（失败不中断） | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_3_1_改造_scripts_dream_cycle_cron_sh** 3.1 改造 `scripts/dream-cycle-cron.sh` | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_3_2_gbrain_cli_v2_0_新增** 3.2 GBrain CLI（V2.0 新增） | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层** 4. 检索增强（RAG）层 | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_4_1_设计动机** 4.1 设计动机 | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_4_2_模块拆分** 4.2 模块拆分 | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_4_3_接口扩展** 4.3 接口扩展 | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_4_4_内网友商环境适配** 4.4 内网友商环境适配 | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_4_5_本地_bge_模型自带方案_方案_a_含安装包分发** 4.5 本地 BGE 模型自带方案（方案 A，含安装包分发） | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_5_内网部署与模型切换_基础设施层** 5. 内网部署与模型切换（基础设施层） | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_5_1_现状** 5.1 现状 | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_5_2_v2_0_增强** 5.2 V2.0 增强 | page_prd_知识管理系统_v2_0_技术方案 | `—` | step |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_6_回测分流_与_tcgf_协同** 6. 回测分流（与 TCGF 协同） | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_7_v2_0_实施里程碑_建议顺序** 7. V2.0 实施里程碑（建议顺序） | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **page_prd_知识管理系统_v2_0_技术方案_8_风险与约束** 8. 风险与约束 | page_prd_知识管理系统_v2_0_技术方案 | `—` | module |  | — | — |
| **node_dream_cycle_cron** Dream Cycle Cron脚本 | infra | `—` | scheduler | 定时触发知识管理系统的同步、嵌入与清理流程 | — | — |
| **node_git_pull** Git Pull | sync | `—` | sync-step | 从远端拉取最新知识库变更 | — | — |
| **node_failure_alert** Git失败告警 | infra | `—` | monitoring | git pull失败时输出告警 | — | — |
| **node_embed** Embed | embedding | `—` | embedding-step | 执行embedding步骤 | — | — |
| **node_cleanup** 清理步骤 | infra | `—` | maintenance | 执行后续清理操作 | — | — |

## 三、依赖关系（59）

| 前置(From) | 类型 | 后继(To) | 说明 |
|-----------|------|---------|------|
| sc_git_status_commit | sequence | api_git_status | 查看工作区状态 |
| sc_git_status_commit | sequence | api_git_commit | 提交变更 |
| sc_git_status_commit | sequence | api_git_log | 查询提交历史 |
| project_param | data | api_git_status | 透传 project 参数 |
| project_param | data | api_git_commit | 透传 project 参数 |
| project_param | data | api_git_log | 透传 project 参数 |
| pagination_contract | data | api_git_log | 分页参数 |
| api_git_status | depends | envelope_contract | 返回信封装约 |
| api_git_commit | depends | envelope_contract | 返回信封装约 |
| api_git_log | depends | envelope_contract | 返回信封装约 |
| api_git_pull | sequence | api_git_conflicts | 拉取后检测冲突 |
| api_git_conflicts | sequence | api_git_push | 无冲突则推送 |
| api_git_pull | depends | git_adapter | 依赖 Git 适配器 |
| api_git_push | depends | git_adapter | 依赖 Git 适配器 |
| api_git_conflicts | depends | git_adapter | 依赖 Git 适配器 |
| git_adapter | depends | script_check_skill | 依赖脚本检查契约 |
| api_git_conflicts | sequence | api_git_diff | 选择文件查看差异 |
| api_git_diff | sequence | api_git_commit | 确定保留版本后提交 |
| dream_cycle | depends | git_collaboration_layer | 复用 Git 能力 |
| dream_cycle | sequence | dream_cycle_cron | 由定时脚本触发 |
| dream_cycle_cron | sequence | gbrain_sync | 执行同步步骤 |
| dream_cycle_cron | sequence | gbrain_embed | 执行嵌入步骤 |
| gbrain_sync | depends | git_collaboration_layer | 依赖真实 Git 操作 |
| gbrain_embed | depends | retrieval_augmentation_layer | 依赖语义/向量能力 |
| retrieval_augmentation_layer | depends | search_cache | 增强检索缓存 |
| ks_backend | depends | git_collaboration_layer | 后端接入 Git 协同 |
| ks_backend | depends | retrieval_augmentation_layer | 后端接入检索增强 |
| ks_frontend | call | ks_backend | 调用后端服务 |
| ks_backend | depends | api_interface_doc | 遵循接口契约 |
| ks_frontend | depends | api_interface_doc | 前端单一数据源 |
| script_check_skill | depends | api_interface_doc | 校验 API 一致性 |
| script_check_skill | depends | ks_backend | 校验后端实现 |
| quality_gate | depends | ks_backend | 后端质量校验 |
| page_prd_知识管理系统_v2_0_技术方案_0_现状基线_v1_x_已实现 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_1_总体架构_v2_0 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_2_1_设计动机 | context | page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_2_2_模块拆分_后端 | context | page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_2_3_新增接口_api_server_js_docs_api_interface_doc_md | context | page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_2_4_gitignore_反转_关键 | context | page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_2_5_前端集成_v2_0_新增页面 | context | page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_3_dream_cycle_复用_git_能力_失败不中断 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_3_1_改造_scripts_dream_cycle_cron_sh | context | page_prd_知识管理系统_v2_0_技术方案_3_dream_cycle_复用_git_能力_失败不中断 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_3_2_gbrain_cli_v2_0_新增 | context | page_prd_知识管理系统_v2_0_技术方案_3_dream_cycle_复用_git_能力_失败不中断 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_4_1_设计动机 | context | page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_4_2_模块拆分 | context | page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_4_3_接口扩展 | context | page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_4_4_内网友商环境适配 | context | page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_4_5_本地_bge_模型自带方案_方案_a_含安装包分发 | context | page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_5_内网部署与模型切换_基础设施层 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_5_1_现状 | context | page_prd_知识管理系统_v2_0_技术方案_5_内网部署与模型切换_基础设施层 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_5_2_v2_0_增强 | context | page_prd_知识管理系统_v2_0_技术方案_5_内网部署与模型切换_基础设施层 | 属于 |
| page_prd_知识管理系统_v2_0_技术方案_6_回测分流_与_tcgf_协同 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_7_v2_0_实施里程碑_建议顺序 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| page_prd_知识管理系统_v2_0_技术方案_8_风险与约束 | context | page_prd_知识管理系统_v2_0_技术方案 | 属于文档 |
| node_dream_cycle_cron | sequence | node_git_pull | 定时触发 |
| node_git_pull | depends | node_failure_alert | 失败时告警 |
| node_embed | sequence | node_cleanup | 嵌入后清理 |

## 四、可测试场景（10）

### sc_git_config_init · Git 仓库初始化与配置



步骤链：

### sc_git_status_commit · Git 工作区提交

通过 /api/git 接口集查看工作区状态、提交变更并查询提交历史

步骤链：api_git_status → api_git_commit → api_git_log

### sc_git_pull_push · Git 远程同步

执行 git pull --rebase 拉取远程更新，并在无冲突时推送本地提交，失败时返回结构化错误且不中断其他流程。

步骤链：api_git_pull → api_git_conflicts → api_git_push

### sc_git_conflict_resolve · Git 冲突查看与解决

pull 或合并产生冲突后，列出冲突文件、查看单文件 diff，并选择保留版本完成冲突解决

步骤链：api_git_pull → api_git_conflicts → api_git_diff → api_git_commit

### sc_v2_architecture · V2.0 总体架构落地

验证 V2.0 新增 Git 协同层、检索增强层与 Dream Cycle 复用 Git 能力的总体架构按技术方案部署。

步骤链：git_collaboration_layer → retrieval_augmentation_layer → dream_cycle → dream_cycle_cron → gbrain_sync → gbrain_embed → ks_backend → ks_frontend → api_interface_doc → script_check_skill → search_cache → quality_gate

### sc_gitignore_reverse · .gitignore 反转与版本化范围



步骤链：

### sc_retrieval_hybrid_search · 混合检索与 RRF 排序



步骤链：page_prd_知识管理系统_v2_0_技术方案 → page_prd_知识管理系统_v2_0_技术方案_0_现状基线_v1_x_已实现 → page_prd_知识管理系统_v2_0_技术方案_1_总体架构_v2_0 → page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 → page_prd_知识管理系统_v2_0_技术方案_2_1_设计动机 → page_prd_知识管理系统_v2_0_技术方案_2_2_模块拆分_后端 → page_prd_知识管理系统_v2_0_技术方案_2_3_新增接口_api_server_js_docs_api_interface_doc_md → page_prd_知识管理系统_v2_0_技术方案_2_4_gitignore_反转_关键 → page_prd_知识管理系统_v2_0_技术方案_2_5_前端集成_v2_0_新增页面 → page_prd_知识管理系统_v2_0_技术方案_3_dream_cycle_复用_git_能力_失败不中断 → page_prd_知识管理系统_v2_0_技术方案_3_1_改造_scripts_dream_cycle_cron_sh → page_prd_知识管理系统_v2_0_技术方案_3_2_gbrain_cli_v2_0_新增 → page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 → page_prd_知识管理系统_v2_0_技术方案_4_1_设计动机 → page_prd_知识管理系统_v2_0_技术方案_4_2_模块拆分 → page_prd_知识管理系统_v2_0_技术方案_4_3_接口扩展 → page_prd_知识管理系统_v2_0_技术方案_4_4_内网友商环境适配 → page_prd_知识管理系统_v2_0_技术方案_4_5_本地_bge_模型自带方案_方案_a_含安装包分发 → page_prd_知识管理系统_v2_0_技术方案_5_内网部署与模型切换_基础设施层 → page_prd_知识管理系统_v2_0_技术方案_5_1_现状 → page_prd_知识管理系统_v2_0_技术方案_5_2_v2_0_增强 → page_prd_知识管理系统_v2_0_技术方案_6_回测分流_与_tcgf_协同 → page_prd_知识管理系统_v2_0_技术方案_7_v2_0_实施里程碑_建议顺序 → page_prd_知识管理系统_v2_0_技术方案_8_风险与约束

### sc_local_embedding_service · 本地 BGE Embedding 服务



步骤链：page_prd_知识管理系统_v2_0_技术方案 → page_prd_知识管理系统_v2_0_技术方案_0_现状基线_v1_x_已实现 → page_prd_知识管理系统_v2_0_技术方案_1_总体架构_v2_0 → page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 → page_prd_知识管理系统_v2_0_技术方案_2_1_设计动机 → page_prd_知识管理系统_v2_0_技术方案_2_2_模块拆分_后端 → page_prd_知识管理系统_v2_0_技术方案_2_3_新增接口_api_server_js_docs_api_interface_doc_md → page_prd_知识管理系统_v2_0_技术方案_2_4_gitignore_反转_关键 → page_prd_知识管理系统_v2_0_技术方案_2_5_前端集成_v2_0_新增页面 → page_prd_知识管理系统_v2_0_技术方案_3_dream_cycle_复用_git_能力_失败不中断 → page_prd_知识管理系统_v2_0_技术方案_3_1_改造_scripts_dream_cycle_cron_sh → page_prd_知识管理系统_v2_0_技术方案_3_2_gbrain_cli_v2_0_新增 → page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 → page_prd_知识管理系统_v2_0_技术方案_4_1_设计动机 → page_prd_知识管理系统_v2_0_技术方案_4_2_模块拆分 → page_prd_知识管理系统_v2_0_技术方案_4_3_接口扩展 → page_prd_知识管理系统_v2_0_技术方案_4_4_内网友商环境适配 → page_prd_知识管理系统_v2_0_技术方案_4_5_本地_bge_模型自带方案_方案_a_含安装包分发 → page_prd_知识管理系统_v2_0_技术方案_5_内网部署与模型切换_基础设施层 → page_prd_知识管理系统_v2_0_技术方案_5_1_现状 → page_prd_知识管理系统_v2_0_技术方案_5_2_v2_0_增强 → page_prd_知识管理系统_v2_0_技术方案_6_回测分流_与_tcgf_协同 → page_prd_知识管理系统_v2_0_技术方案_7_v2_0_实施里程碑_建议顺序 → page_prd_知识管理系统_v2_0_技术方案_8_风险与约束

### sc_dream_cycle_git_pull · Dream Cycle 失败跳过式 Git Pull

cron脚本执行git pull失败时仅告警并继续后续sync/embed/清理步骤，不因Git在线状态中断

步骤链：node_dream_cycle_cron → node_git_pull → node_gbrain_sync → node_embed → node_cleanup

### sc_retest_plan · 缺陷经验回测分流



步骤链：page_prd_知识管理系统_v2_0_技术方案 → page_prd_知识管理系统_v2_0_技术方案_0_现状基线_v1_x_已实现 → page_prd_知识管理系统_v2_0_技术方案_1_总体架构_v2_0 → page_prd_知识管理系统_v2_0_技术方案_2_git_协同层_v2_0_核心新增 → page_prd_知识管理系统_v2_0_技术方案_2_1_设计动机 → page_prd_知识管理系统_v2_0_技术方案_2_2_模块拆分_后端 → page_prd_知识管理系统_v2_0_技术方案_2_3_新增接口_api_server_js_docs_api_interface_doc_md → page_prd_知识管理系统_v2_0_技术方案_2_4_gitignore_反转_关键 → page_prd_知识管理系统_v2_0_技术方案_2_5_前端集成_v2_0_新增页面 → page_prd_知识管理系统_v2_0_技术方案_3_dream_cycle_复用_git_能力_失败不中断 → page_prd_知识管理系统_v2_0_技术方案_3_1_改造_scripts_dream_cycle_cron_sh → page_prd_知识管理系统_v2_0_技术方案_3_2_gbrain_cli_v2_0_新增 → page_prd_知识管理系统_v2_0_技术方案_4_检索增强_rag_层 → page_prd_知识管理系统_v2_0_技术方案_4_1_设计动机 → page_prd_知识管理系统_v2_0_技术方案_4_2_模块拆分 → page_prd_知识管理系统_v2_0_技术方案_4_3_接口扩展 → page_prd_知识管理系统_v2_0_技术方案_4_4_内网友商环境适配 → page_prd_知识管理系统_v2_0_技术方案_4_5_本地_bge_模型自带方案_方案_a_含安装包分发 → page_prd_知识管理系统_v2_0_技术方案_5_内网部署与模型切换_基础设施层 → page_prd_知识管理系统_v2_0_技术方案_5_1_现状 → page_prd_知识管理系统_v2_0_技术方案_5_2_v2_0_增强 → page_prd_知识管理系统_v2_0_技术方案_6_回测分流_与_tcgf_协同 → page_prd_知识管理系统_v2_0_技术方案_7_v2_0_实施里程碑_建议顺序 → page_prd_知识管理系统_v2_0_技术方案_8_风险与约束
