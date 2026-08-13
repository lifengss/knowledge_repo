# 知识管理系统-V2.0-技术方案（原始文档）

# 知识管理系统（KS）V2.0 技术方案

> 版本：V2.0（规划基线：docs/知识管理系统-V2.0-规划.md）
> 配套文档：TCGF V2.0 技术方案（testcase-gen-frontend/docs/）
> 定位：本文档是 V2.0 的**工程落地方案**，定义 KS 后端/前端在 V2.0 的模块拆分、接口契约、数据模型与迁移步骤。所有改动须通过 `script-check-skill` 与 `docs/API-INTERFACE-DOC.md` + `web/src/api-docs-data.js`（单一数据源）保持一致。

---

## 0. 现状基线（V1.x 已实现）

| 能力 | V1.x 现状 | 来源 |
|------|-----------|------|
| 多层 Prompt 分离 + 双路径对接 | 已实现（路径 A 经 AI 平台生成 / 路径 B 直连 KS REST :3000） | server.js 代理、`/api/generate-cases` |
| 质量门控 | 已实现（总分 ≥ 60 入库；短响应/低可信度拒收） | `quality_gate` |
| 双链路 + 双通路入库 | 已实现（human_edit / exec_backflow + batch-commit / single commit；无变更不写库） | `batch_commit.py`、`draft_cache.py` |
| 多项目知识库隔离/共享 | 已实现（`projects.json` + `drafts.db`，私有库 `brains/<project>/` + 共享库 `_shared_brain/`） | `api/projects.py`、`cache/draft_cache.py` |
| 业务依赖图谱 | 已实现（`/api/business-graph`，确定性 + AI 双模式） | `skills/business_graph_builder.py` |
| 冲突处理闭环 | 已实现（merge/overwrite/keep_both/discard 回写 drafts 表） | `cache/conflict_queue.py` |
| AI 通道 | 已实现（ai/codebuddy、ai/openai、none；gbrain 段目前仅配置存储，未接活 LLM） | `ai/ai_adapter.py`、`ai/ai_config.py` |
| 检索 | keyword 模式已实现；**semantic 模式仅有占位**（embedding endpoint 为空，无 rerank/向量库） | `cache/search_cache.py` |

**关键缺口（V2.0 必须解决）**：
1. KS **完全无 Git 能力**（无 `git pull / status / push / commit / log / conflicts / branch / clone / init`），`scripts/dream-cycle-cron.sh` 现有 `gbrain sync`、`embed` 步骤会因 `gbrain` 不是真实 CLI 而失败，且**无 `git pull`**。（即：Dream Cycle 复用 Git 能力，但不以 Git 在线为前置条件——见 §3）
2. **`.gitignore` 当前排除 `brains/`**（见 `test-knowledge-system/.gitignore` 第 11 行 `brains/`），与 V2.0「only track `brains/`」目标相反，须反转。
3. 前端 `web/index.html` 导航（22–79 行）**无 Git 配置/冲突/状态页**；`web/src/app.js` 无 git 调用。
4. 检索增强（向量/BM25/RRF/rerank）尚未落地；gbrain 智能增强（摘要/实体抽取）未接 LLM。

---

## 1. 总体架构（V2.0）

```
┌─────────────────────────────────────────────────────────┐
│  知识系统 KS (Express :3000 + Python 业务层)             │
│                                                         │
│  api/server.js ──callPython──> cache/*.py (SQLite)     │
│       │                          skills/*.py           │
│       ├── Git 协同层  (新增 git_adapter.py + /api/git/*) │
│       ├── 检索增强层  (新增 retrieval/ : 向量+BM25+RRF)  │
│       └── 业务脚本层  (business_graph / quality_rule)   │
│                                                         │
│  scripts/dream-cycle-cron.sh (V2.0 改造: git pull+sync)  │
│       └──> gbrain CLI (V2.0 新增: sync/embed/export)    │
└───────────────────────────┬─────────────────────────────┘
                            │ /api/* (已通配代理)
                    testcase-gen-frontend (BFF :4123)
                            │ git 操作 UI (V2.0 新增)
```

设计原则延续 V1.0：**Markdown + 文件系统先跑通闭环，再迭代工具**；Git 协同作为「知识版本化底座」叠加，不破坏既有入库/质量门控链路。

---

## 2. Git 协同层（V2.0 核心新增）

### 2.1 设计动机
- 会议纪要明确「面向数据再利用/再生成」，而 `brains/` 是真正被复用/再生成的知识资产。用 Git 给 `brains/` 加版本历史，可使：团队协同（多机 push/pull）、回溯（任意 commit 恢复旧版本）、审计（谁何时改了哪条知识）。
- 与 Dream Cycle 解耦：规划强调「Dream Cycle 不能依赖 `git pull` 在线拉取」，故 Git 协同是**独立的运维/协同能力**，cron 内 `git pull` 失败必须**跳过而非中断**整轮。

### 2.2 模块拆分（后端）
新增 `git/git_adapter.py`（薄封装，调用系统 `git` 命令，统一走 `subprocess`，Windows 用 `shell:false` + `PYTHONUTF8=1`，与 `callPython` 一致）：

| 子命令/函数 | 职责 |
|------|------|
| `git_init(repo, branch)` | 在 `brains/` 初始化仓库（若未初始化），默认 `main` |
| `git_status(repo)` | 返回未跟踪/已修改/已暂存文件列表 |
| `git_add_commit(repo, files, msg)` | 暂存 `brains/` 变更并提交（commit message 含操作类型+时间） |
| `git_pull(repo, remote, branch)` | `git pull --rebase`，**失败返回结构化错误，不抛异常中断** |
| `git_push(repo, remote, branch)` | 推送（需凭证/SSH，失败时回退本地提交，记日志） |
| `git_log(repo, limit)` | 提交历史（hash/author/date/message） |
| `git_diff(repo, file)` | 单文件差异（供冲突页展示） |
| `git_branch(repo)` / `git_clone(url, dir)` | 分支管理 / 克隆远程库 |

> 安全约束：所有 git 操作**限定在 `brains/` 目录内**，禁止 `git` 作用于仓库根（避免误提交源码/密钥）。仓库根 `.git` 仅跟踪 `brains/`（`.gitignore` 反转，见 §2.4）。

### 2.3 新增接口（`api/server.js` + `docs/API-INTERFACE-DOC.md`）
统一前缀 `/api/git`，全部透传 `project`（多项目隔离）：

| 方法 | 端点 | 说明 | 响应结构（信封） |
|------|------|------|------|
| GET | `/api/git/config?project=` | 读取该项目的 git 远程/分支/是否已初始化 | `{success, data:{initialized,remote,branch,status}}` |
| PUT | `/api/git/config` | 设置远程仓库 URL / 分支（body 含 project, remote, branch） | `{success, data:{...}}` |
| POST | `/api/git/init` | 初始化仓库（body 含 project, branch） | `{success, data:{initialized}}` |
| POST | `/api/git/commit` | 提交当前 `brains/` 变更（body 含 project, message） | `{success, data:{commitHash}}` |
| POST | `/api/git/pull` | 拉取远程（body 含 project, remote, branch） | `{success, data:{updated, conflicts[], error?}}` |
| POST | `/api/git/push` | 推送（body 含 project, remote, branch） | `{success, data:{pushed, error?}}` |
| GET | `/api/git/status?project=` | 工作区状态 | `{success, data:{untracked[], modified[], staged[]}}` |
| GET | `/api/git/log?project=&limit=` | 提交历史 | `{success, data:{items:[...], total}}` |
| GET | `/api/git/conflicts?project=` | 当前冲突文件列表 | `{success, data:{items:[...], total}}` |
| GET | `/api/git/diff?project=&file=` | 单文件 diff | `{success, data:{file, diff}}` |

> 所有列表类接口返回 `{success, data:{items, total}}` 信封，禁止裸数组（符合 `script-check-skill` 契约）。
> 分页参数统一 `pageSize`（默认 20），不认 `limit`。

### 2.4 `.gitignore` 反转（关键）
当前 `.gitignore` 第 11 行 `brains/` 排除知识库——**必须改为只跟踪 `brains/`**：

```gitignore
# V2.0: 仅版本化知识库内容，源码/运行产物不入库
*
!brains/
!brains/**
!*.md
!.gitignore
```

这样 `git` 仅管理 `brains/` 下 Markdown，源码、node_modules、cache、logs 等不入版本。

### 2.5 前端集成（V2.0 新增页面）
`web/index.html` 导航新增三类入口（补齐规划 22–79 行缺口）：
- **Git 配置页**（data-page=git-config）：设置远程 URL/分支、初始化仓库、连通性测试。
- **Git 状态页**（data-page=git-status）：展示 status/log、手动 commit/push/pull 按钮。
- **Git 冲突页**（data-page=git-conflicts）：列出冲突文件、`/api/git/diff` 对比、选择保留版本。

`web/src/app.js` 新增对应 pageTemplate + 渲染函数，复用既有 `apiGet/apiPost` 并注入 `project`。

---

## 3. Dream Cycle 复用 Git 能力（失败不中断）

> 本节 Git 调用是「尽力而为」，与第 1 节 Git 协同层（人工主动操作 UI）**解耦**：cron 不因 `git` 失败而中断。两者复用同一个 `git/git_adapter.py`，但 Dream Cycle 不以 Git 在线为前置条件。

### 3.1 改造 `scripts/dream-cycle-cron.sh`
V1.x 该脚本已有 `gbrain sync`、`embed`、`alert_monitor`、`cleanup_stale_drafts`，但**缺 `git pull` 且 `gbrain` 非真实 CLI**。V2.0 改造（git pull 为可选、可失败跳过）：

```bash
# 1) Git 拉取（独立、可失败跳过）
python git/git_adapter.py pull --project "$P" || echo "[warn] git pull 失败，跳过（不影响本轮）"

# 2) GBrain 同步（V2.0 真实 CLI 才执行）
if command -v gbrain >/dev/null 2>&1; then
  gbrain sync && gbrain embed
else
  echo "[warn] gbrain CLI 未安装，跳过 sync/embed"
fi

# 3) 既有监控/清理（保留）
bash scripts/alert_monitor.sh
python cache/draft_cache.py cleanup-stale-drafts
```

**关键约束**：`git pull` 失败 → 仅告警，`Dream Cycle` 继续用本地 `brains/` 运行，**绝不中断**。

### 3.2 GBrain CLI（V2.0 新增）
新增 `gbrain` 命令行入口（Python），子命令：
- `gbrain sync`：将 `brains/` 同步到 GBrain 存储（`.gbrain`），调用 `ai/ai_config.py` 的 storage 段。
- `gbrain embed`：对 `brains/` 做向量化（依赖 §4 检索增强层）。
- `gbrain export`：导出知识库快照（供离线交付）。

> 部署依赖：cron 机须 `pip install` gbrain 依赖且 `gbrain` 在 PATH；否则 cron 静默跳过（见 §3.1）。

---

## 4. 检索增强（RAG）层

### 4.1 设计动机
V1.x 仅 keyword（BM25 类）检索；V2.0 规划要求「检索增强」以支撑更准的用例生成上下文。现状 `ai/ai_config.py` 已有 `embedding` 段（endpoint/apiKey/model），但为空、无 rerank、无向量库。

### 4.2 模块拆分
新增 `retrieval/` 目录：
- `retrieval/embed.py`：调用 `embedding` 配置（兼容 OpenAI-style `/v1/embeddings` 或自定义端点；支持 `none` → 回退 keyword）。
- `retrieval/vector_store.py`：向量索引（优先 ChromaDB；内网友商环境若无 ChromaDB 则落本地 SQLite + numpy 近似检索作为可移植回退）。
- `retrieval/hybrid.py`：融合检索 = 向量召回（dense）+ BM25（sparse），经 **RRF（Reciprocal Rank Fusion）** 合并，可选 **rerank**（调用 gbrain/embedding 模型重排 top-K）。
- `retrieval/api.py`：封装 `/api/search?mode=semantic|hybrid|keyword`。

### 4.3 接口扩展
`POST /api/search` 现有 `mode: keyword` 扩展支持 `semantic` / `hybrid`：
```json
{ "query":"登录失败如何测试", "mode":"hybrid", "limit":8, "project":"demo", "rerank":true }
```
响应保持 `{success, data:{items:[{content, score, source}], total}}`。

### 4.4 内网友商环境适配
- 向量库默认 ChromaDB；若部署机无法装 ChromaDB，自动回退 SQLite+numpy（检索质量略降但功能可用）。
- embedding 端点可指向用户自有模型（与 `ai.model` 自定义模型机制一致：`models.json` + project/local setting-sources）。

### 4.5 本地 BGE 模型自带方案（方案 A，含安装包分发）

**目标**：在内网友商环境完全离线、无外网下载的前提下，KS 自带 BGE 中文嵌入模型，本地启动 embedding 服务，供 GBrain 配置并支撑混合搜索（hybrid = 向量召回 + BM25 + RRF）。

#### 4.5.1 选型（方案 A 决策）
- **模型**：BGE 系列开源模型（BAAI，Apache-2.0 授权，可商用、可随包分发）。默认打包 **`bge-small-zh-v1.5`**（~130MB，12 层，内网性价比最优）；如需更高精度可换 `bge-base-zh-v1.5`（~400MB）或 `bge-m3`（多语言，~2.3GB）。
- **推理后端**：**fastembed**（Qdrant 官方轻量库，ONNX 运行时，依赖仅几十 MB，启动快、内存占用低，适配内网/Windows）。不采用 sentence-transformers（依赖 torch ~1.2GB，过重）。
- **向量库**：ChromaDB 以 fastembed 作为 embedding 函数（`client = chromadb.Client(settings=Settings(embedding_function=fastembed.EmbeddingFunction(model_name=...)))`），二者天然共存。

#### 4.5.2 目录结构（模型入库）
模型权重随安装包分发，置于 `models/` 目录（**不进 git**，避免仓库膨胀；由 installer 从安装包释放到部署目录）：

```
test-knowledge-system/
├── models/
│   └── bge-small-zh-v1.5/        # 安装包释放，含 model.onnx + tokenizer.json + config
├── retrieval/
│   ├── embed.py                  # 优先加载本地 models/，失败回退 embedding 段配置
│   ├── vector_store.py           # ChromaDB(fastembed) / SQLite+numpy 回退
│   ├── hybrid.py                 # RRF 融合
│   └── embedding_service.py      # 本地 embedding HTTP 服务（见 4.5.3）
└── .gitignore                    # 追加 models/ 排除（权重走安装包，不入库）
```

> `.gitignore` 反转（§2.4）只跟踪 `brains/`；`models/` 同样排除在 git 外，由 installer 落地。

#### 4.5.3 本地 embedding 服务（GBrain 对接）
新增 `retrieval/embedding_service.py`：基于 fastembed 在本地启动一个 OpenAI-style `/v1/embeddings` 服务（默认 `http://127.0.0.1:8777`），从 `models/bge-small-zh-v1.5` 加载权重，**无需外网**。

```bash
# 随 KS 启动（或 Dream Cycle 前置）
python retrieval/embedding_service.py --model models/bge-small-zh-v1.5 --port 8777
```

`ai/ai_config.py` 的 `embedding` 段默认指向该本地服务：
```json
{
  "embedding": {
    "provider": "local-fastembed",
    "endpoint": "http://127.0.0.1:8777/v1/embeddings",
    "model": "bge-small-zh-v1.5",
    "apiKey": ""
  }
}
```
GBrain 配置（gbrain sync/embed）读取该段，将 `brains/` 向量化入库，实现混合搜索。

#### 4.5.4 启动顺序与回退
1. KS 启动 → 拉起本地 embedding 服务（进程内或子进程，失败仅告警，不阻断 KS 主服务）。
2. `retrieval/embed.py` 优先用 `embedding` 段（本地服务）；服务不可达则按 `provider` 回退：
   - `provider=local-fastembed` 且服务挂 → 回退 keyword（混合搜索降级为纯 BM25，功能不中断）。
   - `provider=none` → 纯 keyword（与 V1.0 一致，保证极小部署也能跑）。
3. installer 在安装阶段校验 `models/` 是否释放成功，缺失则提示「离线语义检索不可用，仅关键字检索」。

#### 4.5.5 安装包集成
- `installer/` 构建脚本将 `models/bge-small-zh-v1.5/` 打进 `KnowledgeOS-Installer.exe`，安装时释放到 `test-knowledge-system/models/`。
- 安装包体积增量约 130–500MB（取决于所选 BGE 尺寸），已在规划预算内。
- 首次启动 `pip install fastembed onnxruntime chromadb`（写入 `requirements.txt`，内网可配离线 PyPI 镜像）。

---

## 5. 内网部署与模型切换（基础设施层）

### 5.1 现状
`ai/ai_config.py` 已支持 `ai`（codebuddy/openai/none）+ `gbrain` + `embedding` 三段；TCGF 侧已验证「自定义模型免登录直连」。KS 的 `call_codebuddy` 已修复 Windows gbk 解码 bug（2026-07-27）。

### 5.2 V2.0 增强
- **统一模型注册**：`~/.codebuddy/models.json` 或 `cwd/.codebuddy/models.json` 注册自定义模型（kimi/glm/deepseek 等），`ai.model` 设为自定义 id 即直连，无需 CLI 登录。
- **离线优先**：`ai.provider=none` 时全链路回退确定性骨架（业务图谱/质量规则已有回退），保证无外网也能跑通。
- **部署脚本**：`installer/` 增加内网部署说明（环境变量种子 `AI_PROVIDER/AI_ENDPOINT/AI_API_KEY/AI_MODEL` + `EMBEDDING_*` + `GBRAIN_*`；embedding 默认本地 fastembed 服务，无需外网）。
- **本地 BGE 自带**：见 §4.5 —— 模型随安装包分发，`retrieval/embedding_service.py` 本地启动，GBrain 经 `embedding` 段配置实现混合搜索。

---

## 6. 回测分流（与 TCGF 协同）

规划要求「回测分流」：历史失败用例（缺陷经验）优先回流重测。KS 侧职责：
- `GET /api/brain/pages?category=defect-experience&project=` 已存在，TCGF 回测视图读取。
- V2.0 新增：`POST /api/retest/plan`（KS 生成回测计划：按 defect-experience 命中 + 最近失败权重排序），TCGF 回测视图调用该端点获取优先级列表。
- 回测结果经既有 `exec_backflow` 通路（路径 B）写回 `defect-experience`，形成「失败→经验→重测」闭环。

---

## 7. V2.0 实施里程碑（建议顺序）

| 阶段 | 任务 | 依赖 | 验收 |
|------|------|------|------|
| M1 | `.gitignore` 反转 + `git_adapter.py` 基础（init/status/commit） | 无 | 仓库根仅跟踪 `brains/`；手动 commit 成功 |
| M2 | `/api/git/*` 端点 + 前端三页 | M1 | 页面可配置/提交/查看 log |
| M3 | Dream Cycle 加 `git pull`（可失败跳过）+ `gbrain` CLI | M1 | cron 跑通，gbrain 缺失时静默跳过 |
| M4 | 检索增强层（embed/vector/hybrid/RRF） | ai_config embedding | `/api/search?mode=hybrid` 返回融合结果 |
| M5 | 回测分流 `POST /api/retest/plan` | 既有 defect-experience | TCGF 回测视图读优先级 |
| M6 | 内网友商模型切换文档 + installer 更新 | M4/M5 | 离线 + 自定义模型端到端验证 |

---

## 8. 风险与约束

1. **git 凭证**：内网 push 需 SSH/令牌，KS 不存密钥，失败仅本地提交（安全优先）。
2. **冲突治理**：git 冲突与既有 `conflict_queue`（草稿冲突）是两套概念——git 冲突是文件级版本冲突，草稿冲突是知识内容冲突；V2.0 仅暴露 git 冲突页，不联动草稿冲突。
3. **契约一致性**：所有新增 `/api/git/*` 与 `/api/search` 扩展必须同步 `docs/API-INTERFACE-DOC.md` + `web/src/api-docs-data.js`，并经 `script-check-skill` 校验。
4. **不破坏 V1.0 闭环**：Git 协同/检索增强为叠加层，入库/质量门控/双链路逻辑不变。

