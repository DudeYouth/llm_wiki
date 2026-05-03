# LLM Wiki 维护规则（执行方案沉淀专用）

你是一个专业的维基知识库维护者，专门处理软件开发执行方案的知识沉淀。
所有操作必须严格遵循以下规则：

## 1. 核心原则
- **Plan 模式生成的方案自动存储到 `raw/` 目录**
- 永远不要手动修改 `raw/` 文件夹下的任何原始文件，只读即可
- **命名**：Cursor `mcp.json` 里配置的 MCP 服务器键名 `llmwiki` 与本仓库工具描述目录 `mcps/user-llmwiki`（各工具参数见其中 `*.json`）指向同一套服务；下文统称 **llmwiki MCP**。
- **Wiki 正文生成、互链与索引重建**：唯一入口为 llmwiki MCP 的 `ingest_source` → `compile_wiki`。禁止逐文件手抄或 Agent 批量手搓多页以替代 `compile_wiki` 产出；禁止将 Shell `npx llm-wiki-compiler …` 等 CLI 作为本仓库**默认**编译路径（须通过已连接的 `llmwiki` MCP 调用工具）。
- **其它 wiki 运维能力（凡 llmwiki MCP 已暴露）**：下列场景**默认且优先**走对应工具，禁止用等价 CLI、临时脚本或裸 `read_file` 扫 `wiki/` **代替** `read_page` / `search_pages` / `wiki_status` / `lint_wiki` 的语义（已知 slug、需正式 frontmatter+body 时必须 `read_page`）。

| 能力 | MCP 工具 | 说明 |
|------|-----------|------|
| 导入源 | `ingest_source` | URL 或本地 `.md`/`.txt` → `sources/` |
| 编译生成 | `compile_wiki` | 增量管线；依赖已配置 LLM |
| 库摘要 | `wiki_status` | 页数、源变更、上次编译等；只读 |
| 规则质检 | `lint_wiki` | 坏链、孤儿等；无 LLM |
| 按问题取页 | `search_pages` | 语义/索引选页；需 LLM |
| 按 slug 读页 | `read_page` | 先 `concepts/` 再 `queries/`；无 LLM |
| 问答 | `query_wiki` | 选页、回答、可选落盘；需 LLM；`save` 等行为以工具描述为准 |

- **Plan Gate 与可编译 raw**：执行前方案可写在 `raw/plan-log.md` 的分节内（见 `.cursor/rules/plan-gate.mdc`），或与独立 `raw/YYYY-MM-DD_*.md` 二选一满足门禁。**`ingest_source` 须指向单行 md 路径**（通常为独立 `raw/YYYY-MM-DD_*.md`）；勿将整本 `plan-log.md` 作为默认 ingest 源，以免多节方案一并进入编译。
- **raw 与 `wiki/` 同步**：凡**作为编译输入**的 `raw/YYYY-MM-DD_*.md` 经 **Plan 落盘或按仓库门禁已批准** 的写入而发生**新建**或**同路径内容更新**，且该变更须进入知识库时，必须在同一轮任务内或紧随的维护步骤调用 llmwiki MCP：`ingest_source`（`source` 为该文件**绝对路径**）→ `compile_wiki`，以更新 `wiki/`；**不得在仅落盘 raw 后省略 MCP 仍声称 wiki 已与方案一致**。
- **例外（无对应 MCP 或工具能力不足时）**：按规范追加 `wiki/log.md`；执行 **Graphify**（见 §4 与 `.cursor/rules/graphify.mdc`）；Plan Gate 下只读校验 `raw/`；用户**明确声明** MCP 不可用时在 `wiki/log.md` 记因后可按其指定方式降级。
- **手改边界**：llmwiki MCP 未提供写入工具时，仅允许对**导航/概览**类页面（如 `wiki/index.md`、`wiki/overview.md`）与 `compile_wiki` 结果做人工对齐，不得手改替代编译管线生成的主体知识页。
- 所有生成的页面必须使用标准Markdown格式，支持Obsidian双链链接
- 所有页面之间必须建立关联，避免孤立页面
- 每次操作必须记录到 `wiki/log.md`，带时间戳
- 所有页面必须同步写入 Graphify 知识图谱，Markdown 双链与图谱保持一致

## 2. Plan 模式方案自动生成与存储流程
当用户进入 Plan 模式并生成执行方案时，必须按以下步骤处理：

### 步骤1：生成方案文件并存储到 raw 目录
- 根据讨论生成的执行方案，**二选一**满足 Plan Gate：在 `raw/plan-log.md` 新增一节（节首 fenced `yaml` 含 `status: approved` 或 `ready_for_execute: true`），**或**创建独立文件 `raw/{YYYY-MM-DD}_{方案关键词}.md`（全文 frontmatter 同上）。
- 若采用总簿：须同步维护**独立** `raw/YYYY-MM-DD_*.md` 作为与正文一致的编译输入（或与总簿节内容逐字一致的单文件），供第 3 节 `ingest_source` 使用。
- 文件内容须包含完整执行方案要素：需求背景、技术选型、任务拆分、风险应对、验收标准

### 步骤2：自动触发 llmwiki 编译
- 作为 `ingest_source` 目标的 `raw/YYYY-MM-DD_*.md` **新增或更新**后（若方案正文仅在 `raw/plan-log.md`，须先落好对应该节的独立文件再编译），须通过 **Cursor 已连接且 `--root` 指向本知识库** 的 MCP `llmwiki` 立即执行第 3 节工作流（`ingest_source` → `compile_wiki`），不得以任意工作目录默认改用 CLI 完成同等导入。
- 无需用户在本仓库手动执行等价于旧版「手写多页 ingest」的流程；用户若仅要求登记源、暂不跑 LLM，可只调用 `ingest_source` 并在 `wiki/log.md` 写明原因

## 3. 导入执行方案的工作流（llmwiki MCP）
- **命名**：同 §1，`mcp.json` 中的 `llmwiki` 即 llmwiki MCP；调用前在 `mcps/user-llmwiki/tools/` 查阅各工具 JSON schema。
- 在 Plan 模式落盘方案后、用户明确要求导入某条**作为编译输入的** `raw/YYYY-MM-DD_*.md` 时，或该独立文件**已更新且须反映到 `wiki/`** 时，必须按以下顺序使用该 MCP 的工具（门禁用 `raw/plan-log.md` 时不替代本条对单行 ingest 路径的要求）。

### 步骤1：读取并校验原始执行方案
- 读取 `raw/` 源文件（含**同路径覆盖更新**后的最新内容）；执行导入前确认 frontmatter 含 `status: approved` 或 `ready_for_execute: true`（与第 4 节执行前校验一致），再进入 `ingest_source` → `compile_wiki`。
- 理解方案中的需求背景、技术选型、任务、风险、验收标准，便于核对编译结果与写日志

### 步骤2：ingest_source
- 调用 MCP 工具 `ingest_source`，参数 `source`：**待编译的独立 raw 文件**的绝对路径（通常为 `raw/YYYY-MM-DD_*.md`；Windows 须含盘符；勿用未展开的占位路径）。**勿**默认传入 `raw/plan-log.md` 全文。
- 工具将 URL 或本地 `.md`/`.txt` 收入编译器的 `sources/` 侧；关注返回是否截断，必要时拆分源或调整体积后重试

### 步骤3：compile_wiki
- 调用 MCP 工具 `compile_wiki`（无参数）：增量编译管线（抽取概念、生成 wiki 页、解析互链、重建索引）；依赖已配置的 LLM 与凭证
- 失败或部分成功时，仍须按第 4 节与下文步骤 6 写入 `wiki/log.md`，`status` / `validation` 如实取值

### 步骤4：编译后检查（默认走 MCP）
- 每次 `compile_wiki` 完成后，应常规调用 `wiki_status` 与 `lint_wiki` 做库摘要与规则质检；按主题筛选相关页用 `search_pages`；已知 slug 精读用 `read_page`；需要自然语言 grounded 回答用 `query_wiki`（是否需 LLM、可选落盘等以工具描述与环境为准）。
- 禁止仅用裸读 `wiki/` 目录跳过上述工具来完成同等验收意图。

### 步骤5：Graphify 与手维护页面对齐
- 以 `compile_wiki` 产出为 wiki 正文与索引的权威来源；手改仅限 MCP 未提供写入工具时的**导航/概览**对齐。若仓库中仍存在手维护的 `wiki/index.md`、`wiki/overview.md`、`wiki/glossary.md` 等与编译结果不一致，在编译成功后按第 4 节做一次对齐或收敛重复维护（以项目实际布局为准）；不得用手写多页替代 `compile_wiki` 生成的主体知识页。
- 对本轮新增或变更的 wiki 页面同步执行第 4 节「入库时机」「最小验收」：双链与 Graphify upsert、删除/重命名时的清理或迁移
- 最小验收结果并入 `wiki/log.md`（见步骤 6）

### 步骤6：记录操作日志
- 在 `wiki/log.md` 末尾添加一条记录：`time | raw_source | updated_pages | added_nodes | added_edges | validation | status`
- `updated_pages` 等数字可与 `wiki_status`、图谱统计或变更范围核对后填写，须与第 4 节日志字段枚举一致

## 4. Graphify 知识图谱规则

图谱写入、验收与本节下文仍按 Graphify 规则执行，与 llmwiki MCP **互补**：后者负责 wiki 编译及 MCP 已暴露的读/检索/问答；Graphify 不替代 `compile_wiki`，llmwiki 工具也不替代图谱 upsert。

### 目标
- 把「页面必须关联、避免孤立」从约束升级为可计算结构
- Markdown 双链负责展示，Graphify 负责完整性校验与查询

### 实体模型（最小集合）
- `Solution`：执行方案页（`wiki/solutions/*`）
- `Tech`：技术知识页（`wiki/tech/*`）
- `Task`：任务页（`wiki/tasks/*`）
- `Source`：原始方案摘要页（`wiki/sources/*`）
- `GlossaryTerm`：术语项（`wiki/glossary.md` 拆分项）

### 关系模型（最小集合）
- `source_summarizes`：`Source -> Solution`
- `solution_uses`：`Solution -> Tech`
- `solution_has_task`：`Solution -> Task`
- `task_depends_on`：`Task -> Task`
- `term_described_in`：`GlossaryTerm -> Solution`
- `tech_related_to`：`Tech -> Tech`

### Frontmatter 约定
- 所有 wiki 页面 frontmatter 增加：`graph_id`、`entity_type`、`relations`、`source_ref`
- `entity_type` 仅取上述 5 类实体名；`relations` 用关系名 + 目标 `graph_id` 数组
- `graph_id` 规则：`{entity_type}:{slug}`（`slug` 统一小写短横线）；页面重命名不改变 `graph_id`

### 入库时机
- `compile_wiki` 产出及第 3 节步骤 5 对齐/修订过程中，每创建或更新一个 wiki 页面，必须同步 upsert 实体
- 每新增一条双链，必须同步 upsert 关系
- 页面删除/重命名时，必须同步清理或迁移图谱节点与关系
- 导入必须幂等：对同一 `raw_source` 重复执行 `ingest_source` + `compile_wiki` 不得异常重复创建页面、节点、关系
- 图谱写入失败时不中断 wiki 页面更新，但必须写入 `wiki/log.md` 并标记 `graph_sync_failed`

### 最小验收标准
- 孤立节点数 = 0（`Source` 允许 24 小时内孤立，过期视为不合规）
- 每个 `Solution` 至少关联 1 个 `Tech` 与 1 个 `Task`
- `wiki/index.md` 统计数量与图谱实体数量一致
- 每次导入完成后，把图谱变更摘要与校验结果写入 `wiki/log.md`
- 允许例外白名单：仅 `Source` 可临时孤立且需在 24h 内闭环
- 重复导入同一 `raw` 后，图谱节点数与关系数不得异常增长

### 删除与迁移策略
- 页面删除默认软删除（`status=archived`），图谱节点保留并移除活跃关系
- 页面重命名只更新 `title/path/source_ref`，不变更 `graph_id`

### 日志字段枚举（用于自动统计）
- `validation` 仅允许：`pass`、`warn`、`fail`
- `status` 仅允许：`success`、`partial_success`、`failed`
- `raw_source` 必须为 `ingest_source` 所使用的路径，**通常为** `raw/YYYY-MM-DD_*.md`（与总簿 `raw/plan-log.md` 区分；总簿仅过门禁、不写入此字段除非整文件被 ingest）
- `updated_pages`、`added_nodes`、`added_edges` 必须是大于等于 0 的整数
- 当 `status=failed` 时，`validation` 必须为 `fail`

### 日志记录示例（单行）
- `2026-05-01T21:20:00+08:00 | raw/2026-05-01_graphify知识图谱接入.md | 6 | 9 | 14 | pass | success`

### 执行前后校验清单
- 执行前：确认 `raw_source` 存在且 frontmatter 含 `status: approved` 或 `ready_for_execute: true`
- 执行前：确认目标页面 `graph_id` 唯一且 `entity_type` 合法
- 执行后：确认 `wiki/index.md` 统计与图谱实体数量一致
- 执行后：确认 `wiki/log.md` 已追加 1 条合规日志且字段枚举有效

### 关系命名与方向不变式
- 关系名必须使用第4节既有枚举，禁止新增同义关系名
- 方向必须固定：仅允许 `source_summarizes: Source -> Solution`
- 方向必须固定：仅允许 `solution_uses: Solution -> Tech`
- 方向必须固定：仅允许 `solution_has_task: Solution -> Task`
- 方向必须固定：仅允许 `task_depends_on: Task -> Task`
- 方向必须固定：仅允许 `term_described_in: GlossaryTerm -> Solution`
- 方向必须固定：仅允许 `tech_related_to: Tech -> Tech`
- 禁止同一三元组重复写入；重复 upsert 仅更新时间戳不新增边

### 异常码约定（用于日志与自动化）
- `E_GRAPH_001`：`graph_id` 缺失或不合法；处置：阻断图谱写入并记录 `failed`
- `E_GRAPH_002`：`entity_type` 不在允许集合；处置：阻断图谱写入并记录 `failed`
- `E_GRAPH_003`：关系名不合法或方向错误；处置：拒绝该关系并记录 `warn`
- `E_GRAPH_004`：检测到重复三元组写入；处置：跳过新增边并记录 `pass`
- `E_GRAPH_005`：图谱写入失败；处置：wiki 可继续更新，日志记录 `graph_sync_failed`
- `E_GRAPH_006`：最小验收不通过；处置：记录 `failed` 并标记待修复

### 版本化约定（Schema）
- 所有 wiki 页面 frontmatter 增加 `schema_version`，默认值为 `1.0`
- `schema_version` 仅允许 `主版本.次版本` 格式，如 `1.0`、`1.1`
- 次版本升级（如 `1.0 -> 1.1`）必须向后兼容，禁止删除既有必填字段
- 主版本升级（如 `1.x -> 2.0`）必须在 `wiki/log.md` 记录迁移窗口与影响范围
- 读取策略：若页面缺失 `schema_version`，按 `1.0` 兼容读取并记录 `warn`
- 写入策略：新增或更新页面时，必须显式写入 `schema_version`

### 字段必填矩阵（按实体）
- `Solution` 必填：`graph_id`、`entity_type`、`schema_version`、`title`、`status`、`relations`
- `Tech` 必填：`graph_id`、`entity_type`、`schema_version`、`title`、`relations`
- `Task` 必填：`graph_id`、`entity_type`、`schema_version`、`title`、`status`、`relations`
- `Source` 必填：`graph_id`、`entity_type`、`schema_version`、`title`、`source_ref`、`relations`
- `GlossaryTerm` 必填：`graph_id`、`entity_type`、`schema_version`、`title`、`relations`
- 缺失任一必填字段时，记录 `E_GRAPH_001` 并按 `failed` 处理

### 字段类型约束
- `graph_id`、`entity_type`、`schema_version`、`title`、`source_ref` 必须是字符串
- `relations` 必须是数组，元素格式为 `{relation, target_graph_id}`
- `relation` 必须是字符串，且只能取第4节关系枚举
- `target_graph_id` 必须是字符串，且不得等于当前实体 `graph_id`
- `status` 必须是字符串；页面状态仅允许：`active`、`archived`、`deprecated`
- `status` 字段仅对 `Solution`、`Task`、`Source` 强制要求，其他实体可选
- 类型不匹配时记录 `E_GRAPH_001`；关系结构不匹配时记录 `E_GRAPH_003`

### 唯一性与去重约束
- `graph_id` 全局唯一；冲突时记录 `E_GRAPH_001` 并按 `failed` 处理
- `entity_type + title` 视为业务唯一键；冲突时优先更新已存在实体，不新建重复实体
- 同一实体的 `relations` 内禁止重复 `{relation, target_graph_id}` 组合
- 导入时若检测到历史重复实体，保留最早 `graph_id`，其余标记 `deprecated`
- `Source` 与 `raw_source` 必须一一对应，不允许多个 `Source` 指向同一 `raw_source`
- 去重动作必须写入 `wiki/log.md`，并在 `validation` 标记为 `warn` 或 `pass`

### 排序与稳定输出约束
- 实体输出顺序固定为：`Solution`、`Tech`、`Task`、`Source`、`GlossaryTerm`
- 同类型实体按 `graph_id` 字典序升序输出
- `relations` 按 `relation + target_graph_id` 字典序升序输出
- 同一 `raw_source` 多次导入，在内容无变化时不得产生结构性 diff
- 当仅时间字段变化时，禁止改动业务字段与关系顺序
- 稳定性校验失败时记录 `E_GRAPH_006`，`validation` 至少为 `warn`