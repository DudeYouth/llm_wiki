---
name: llm-wiki-pattern
description: Use when building or maintaining a persistent agent-written markdown wiki, when defining wiki schema or AGENTS conventions, when running wiki health checks, when compounding knowledge instead of stateless RAG, when the user mentions Memex or second-brain wikis, multi-session wiki drift, RAG vs persistent wiki, or when the user references Karpathy LLM Wiki, raw/wiki/schema layering, index.md, log.md, wiki backfill, or Obsidian as the wiki reader. Also when the user mentions llmwiki MCP, ingest_source, compile_wiki, or wiki compile via MCP.
disable-model-invocation: true
---

# LLM Wiki 模式（Karpathy）

思想来源：[Andrej Karpathy — LLM Wiki（gist）](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。gist 是**理念稿**：传递高层模式，具体目录、模板与工具由你与 Agent **协作实例化**；下文各条**可选、可裁剪**，按域在 schema 中取舍。

精神上与 Vannevar Bush 的 **Memex** 相近：私人、策展、**文档之间的关联与轨迹**与正文同等重要；Bush 未解决「谁来维护关联」，由 **Agent 承担簿记与互链**。本文档将模式落实为 Agent 可执行约定；**具体路径、工具与 MCP 以项目 schema 与专规为准**（本仓库见 `.cursor/SKILL/zhishiku` 与 Plan Gate）。

## 快速对照

| 操作 | 触发 | 主要产出 |
|------|------|----------|
| **Ingest** | 新/更新的 raw 源需进入知识库 | 常触 **约 10–15** 个 wiki 页级更新、`index`、`log` |
| **Query** | 用户问题依赖已有沉淀 | 带引用的回答；高价值则回写 wiki |
| **Lint** | 定期或 wiki 明显膨胀后 | 问题清单；可选写入 `log` |

**路由**：有新源待入库 → **Ingest**；无新源、仅基于已有库作答 → **Query**；库已积累一段时间或出现导航困难 → **Lint**。

## 核心判断

- **与 RAG 式「每次现拼」**：上传多文件、问时再检索片段，**没有跨问答的积累**；多文档 subtle 综合每次都要重新找齐碎片。本模式要求知识**编译进持久 wiki**并**持续更新**，而非每问重新推导。
- **wiki 是可积累的产物**：互链、矛盾标注、综合随新源演进；**人类极少亲自写 wiki 正文**，以阅读与策展为主。
- **人机分工（gist 原文要点）**：人类——**策展源**、**把握探索方向**、**提出好问题**、**阐释意义**；Agent——摘录、互链、归档、一致性等**簿记与体力活**。
- **实践画面**：一侧 Agent 会话改文件，一侧 Obsidian（或等价）实时跟链、看图谱、读更新页：**Obsidian 像 IDE，Agent 像程序员，wiki 像代码库**（比喻义：可读可航行的结构化产物）。

## 典型适用场景（示例）

- **个人**：目标、健康、日记与文章剪藏等结构化自省。
- **研究**：数周数月读论文/报告，综合与论点随时间演进。
- **读书**：按章入库，人物/主题/情节线互联，形成伴读 wiki。
- **团队**：会议、Slack、文档、客户录音等喂库；**人类在环**审更新亦可。
- **其它**：竞品、尽调、行程、课程、爱好等——凡需**长期堆积且要条理**而非散落对话处。

## 适用边界

**适合**：中长期、人类策展的单库或小团队；研究/读书/个人知识等**愿意维护 schema**、接受 Agent 多文件更新的场景。

**不适合**：替代无专门治理的**企业级**知识库（权限、审计、并发、合规、敏感数据分级等不在本 skill 范围）；要求**零校验**的高风险决策仅凭 wiki 定论。

**溯源与派生层**：wiki 是对 raw 的**有损压缩**；重要结论须在页内**指向 raw 或源摘要页**，新源到达后应更新或标注过时论断；规模上来后应用**搜索/校验器/span 级引用**等由项目在 schema 中自选，本 skill 不规定具体实现。

**多会话 / 多 Agent**：在 schema 中固定**唯一目录与命名规则**；开放问题、待决议矛盾写入 **`log` 或专用页**，避免只留在聊天；**index** 定期去重合并同主题条目，减少并行会话产生的重复页。

## 三层架构

| 层 | 职责 | 人类 / Agent |
|----|------|----------------|
| **Raw 源** | 文章、论文、图像、数据文件等；**不可变**，真相源 | 人类放入；Agent **只读** |
| **Wiki** | 摘要、实体、概念、对比、概览、综合；互链 Markdown | Agent **全权撰写与维护**；人类以阅读器浏览 |
| **Schema** | 结构、约定、ingest/query/lint 流程 | `AGENTS.md` / `CLAUDE.md` 等；人与 Agent **共同演化**，使 Agent 成为「守纪律的维基维护者」而非泛聊天 |

## Schema 文档最小契约（写入 AGENTS / CLAUDE）

项目应在 schema 中**至少**写明：

- **目录与页型**：例如 `entities/`、`concepts/`、`sources/`、`synthesis/`（名称可按域替换）。
- **互链语法**：如 `[[slug]]` 或项目统一链接规则。
- **frontmatter 约定**（可选但需统一）：`title`、`tags`、`date`、关联 `sources` 等。
- **ingest 后必做**：更新内容索引（如 `wiki/index.md`）、追加 `wiki/log.md`；谁负责矛盾页、谁更新实体索引。
- **ingest 节奏（二选一写清）**：**单源 + 人类参与**（读摘要、指强调点）与 **批量少监督**；写入 schema 供后续会话遵守。
- **矛盾与过时**：新源与旧页冲突时——并列标注、修订日期、或单独「待决议」节，禁止静默覆盖而不留痕。

本仓库执行 wiki 编译与门禁时，**以 `zhishiku` + Plan Gate 专规为准**，本节为通用缺省心智模型。

## 三类操作（摘要）

**Ingest**：raw 入库 → 读源、可与人类对要点 → 更新摘要页、**跨多实体/概念页**、**index**、**log**；单源触及页数多属正常。

**Query**：对 **wiki** 搜/选页 → 阅读 → **带引用**综合；回答形态可为**独立 md 页、对比表、幻灯（如 Marp）、图表、画布**等，由问题决定。**高价值回答应落盘为 wiki 新页或更新**，与 ingest 一样让探索复利。

**Lint**：健康检查；并善用 Agent **提议下一步问题与新源**（gist：lint 可驱动研究议程）。

## MCP 实现轮廓（可选）：llmwiki

**专规优先**：若项目另有完整 wiki 专规（本仓库为 `.cursor/SKILL/zhishiku`），以专规为准；本节为可移植摘要，便于在无冗长专规时仍能对齐 llmwiki 工具名与顺序。Skill **不**实现 MCP 服务端；仅约定 Agent **如何**调用已暴露工具。

### 适用条件

- Cursor `mcp.json`（或等价配置）已连接 **llmwiki** 服务器（或项目内约定等价名）。各工具参数以项目内 MCP 描述符为准（例如 `mcps/user-llmwiki/tools/*.json`）。
- **无**该 MCP 时跳过本节：Ingest / Query / Lint 仍按上文工具无关清单，由 Agent 按 schema 写文件完成。

### Ingest 硬路径

1. **`ingest_source`**：`source` 为待编译的**单个** URL 或本地 `.md`/`.txt`；本地路径宜用**绝对路径**（Windows 含盘符）。
2. **`compile_wiki`**：通常无参；增量编译管线（成页、互链、索引等；依赖 LLM 时按项目配置生效）。
3. 产出与 gist 心智一致：单源常触发**多页** wiki 更新，与上文「单源触十多页」同级。

**勿**用 Agent 批量手抄多页 wiki **替代**上述编译管线作为主体知识页的默认路径（除非项目专规明确允许）。

### 工具矩阵（摘要）

| 能力 | MCP 工具 | 说明 |
|------|-----------|------|
| 导入源 | `ingest_source` | URL 或本地 md/txt → 编译器 sources 侧 |
| 编译生成 | `compile_wiki` | 增量编译；依赖 LLM 时按项目生效 |
| 库摘要 | `wiki_status` | 页数、变更等只读摘要 |
| 规则质检 | `lint_wiki` | 坏链、孤儿等；无 LLM |
| 按问题取页 | `search_pages` | 语义/索引选页；需 LLM 时按项目生效 |
| 按 slug 读页 | `read_page` | 精读单页；无 LLM |
| 问答 | `query_wiki` | 选页、综合作答；可选落盘以工具 schema 为准 |

**参数、必填字段与例外**以各工具 JSON schema 及项目专规（如 `zhishiku`）为准。

### 与「Agent 直接写 wiki」的分工

- **主体知识页**：以 `compile_wiki` 产出为权威；无 MCP 写入工具时，**导航/概览**（如 `wiki/index.md`）可在专规允许范围内人工或与编译结果对齐后手改。
- **raw** 仍为真相源：归纳反映在 wiki 与 log，**不就地改写 raw**（与本 skill 前文一致）。

### 常见陷阱（跨项目）

- **勿**将「总簿式」多节方案（如整本 `plan-log.md`）作为默认 `ingest_source` 目标，以免无关章节一并进入编译；应使用**单行路径对应单文件**的可编译源，详见 [.cursor/rules/plan-gate.mdc](.cursor/rules/plan-gate.mdc)（若仓库启用）。
- MCP 不可用时在专规允许下于 `wiki/log.md` 记录原因并降级；勿声称已编译却未执行约定步骤。

### Query / Lint 与 MCP

- **Query**：除 `index.md` 外，可优先 `search_pages`、`read_page`、`query_wiki`；是否落盘以工具描述与专规为准；高价值结果仍建议回写 wiki 并记 log（见上文 Query 清单）。
- **Lint**：规则质检可走 `lint_wiki`；与工具无关清单中的矛盾/陈旧/重复等**语义 lint**互补，结论可写入 `log`。

### 专规入口（本仓库）

- `.cursor/SKILL/zhishiku`、Plan Gate：`.cursor/rules/plan-gate.mdc`；图谱与 MCP 分工见 `.cursor/rules/graphify.mdc`（细节不重复于此）。

## Ingest 检查清单（工具无关）

- [ ] 确认新源已落在 **raw**（或项目规定的只读源区），**不就地改写 raw**。
- [ ] 若 schema 要求：体积过大、敏感字段、许可证——先拆分、脱敏或拒收，并在 **log** 说明。
- [ ] 读完全文或约定范围；可选与人类对齐**要点与强调方向**（单源参与式流程）。
- [ ] 提取与现有 wiki **可对齐的实体/概念**；预期多文件联动（量级约十页级属常见）。
- [ ] 新建或更新 **源摘要页**（若项目有 `sources/` 约定）。
- [ ] 更新相关 **实体页、概念页、综合页**，补互链；冲突处按 schema 显式标注。
- [ ] 更新 **`index.md`（或等价目录页）** 中的条目与一行摘要。
- [ ] **追加 `log.md`**，类型标为 `ingest`，并列出主要触及路径。
- [ ] 若项目规定 lint/query 验收，在 ingest 后执行一轮（见 Lint / Query 清单）。

## Query 检查清单（工具无关）

- [ ] **先**读 `index.md` 或执行项目约定的搜索/选页，再下钻正文；索引不足时再 **全文/语义搜索**（若项目已配置）。
- [ ] 回答中 **引用 wiki 页名或链接**，关键断言能追溯到源或源摘要。
- [ ] 若 wiki **无**相关页或证据不足，在回答中**显式说明缺口**，避免用聊天层幻觉填补。
- [ ] 若产出高价值结构（对比表、决策树、跨源综合、幻灯/图表达），评估 **回写** 为新页或更新 `synthesis/` 等；禁止仅留在会话。
- [ ] 回写后更新 **index** 与 **log**（类型 `query` 或项目约定的标签）。

## Lint 检查清单（工具无关）

- [ ] **矛盾**：相关页对同一事实是否不一致；是否需并列或修订说明。
- [ ] **陈旧**：是否有被新源否定的旧论断仍呈「当前事实」语气。
- [ ] **重复**：标题或主题极近的多页是否应合并或互链到权威页。
- [ ] **孤儿**：重要页是否缺少入链；索引是否漏列。
- [ ] **断链**：目标页不存在的出站链接。
- [ ] **缺互链**：gist 所列「应有交叉引用却未链」的相邻概念/实体是否补链。
- [ ] **缺页**：正文多次提及但无独立页的重要概念是否建 stub 或合并。
- [ ] **数据缺口**：是否列出可通过**进一步检索或新源**填补的空白（不强制联网，由 schema 决定）。
- [ ] **index / log**：最近一次 ingest/query 是否在 log 中有记录；index 是否与库内分类一致。
- [ ] 输出 lint 结论到 **log**（类型 `lint`），可含**建议追问与新源方向**（可选）。

## 导航与日志：职能区分（gist）

- **`index.md`——内容向目录**：全库（或分域）页面的链接 + **一行摘要** + 可选元数据（日期、源数量等）；按类组织（实体、概念、源等）。**每次 ingest 后更新**；**回答 query 时先读 index 再下钻**。在**中等规模**（约百级源、数百页量级）常已够用，可减轻对 **embedding 式 RAG 基建**的依赖。
- **`log.md`——时序 append-only**：记录 ingest / query / lint 何时发生。**统一条目前缀**便于脚本解析，例如 Unix：`grep "^## \\[" wiki/log.md | tail -5` 看最近五条；帮助 Agent 理解**近期已做操作**。

### `wiki/log.md` 条目示例

```markdown
## [2026-05-02] ingest | 某论文标题

- raw: `raw/2026-05-02_paper-slug.md`
- touched: `wiki/sources/paper-slug.md`, `wiki/concepts/foo.md`, `wiki/index.md`
- notes: 与 [[wiki/concepts/bar]] 并列保留两种观点

## [2026-05-02] query | 用户问题摘要

- pages_used: [[concepts/foo]], [[sources/paper-slug]]
- backfilled: `wiki/synthesis/foo-vs-bar.md`（若本次回写）

## [2026-05-02] lint | 周检

- findings: 断链 2；孤儿 1；建议合并 [[concepts/old-a]] 与 [[concepts/old-b]]
```

### `wiki/index.md` 片段示例

```markdown
## Sources

| 页 | 摘要 |
|----|------|
| [[sources/paper-slug]] | 方法：…；结论：… |

## Concepts

| 页 | 摘要 |
|----|------|
| [[concepts/foo]] | 与 bar 的差异见 synthesis/… |
```

## Obsidian 与实践提示（gist 浓缩）

- **Obsidian Web Clipper**（扩展）：网页转 Markdown，快速进入 **raw**。
- **剪藏进 raw**：与上配合，形成稳定 ingest 入口。
- **附件本地化**：固定附件目录（如 `raw/assets/`）+「下载当前文件附件」类快捷键，减少外链失效；**先读文再按需读图**（inline 图常不宜单 pass 读完）。
- **图谱**：看枢纽、桥、孤儿，辅助 lint。
- **Marp / Dataview**：幻灯与基于 frontmatter 的动态列表（与 schema 一致即可）。
- **Git**：wiki 即 Markdown 仓，版本、分支、协作可沿用 Git 工作流。

## 可选增强（gist：CLI / 搜索）

- 小规模：**index + 互链**优先；变大后为 Agent 配**/wiki 级搜索**（自建脚本或现成引擎）。
- gist 提及 **qmd**：本地 Markdown、混合 **BM25 + 向量**、可 **LLM 重排**，带 CLI 与 **MCP**，便于 Agent 以工具或 shell 调用；亦可自写最小搜索脚本。

## Skill + qmd：衍生功能点（可选）

本节在**不依赖其它 MCP** 的前提下，把本 skill 的 Query 纪律与 **qmd**（见上文「可选增强」）组合后的可落地能力点单列，便于写入项目 schema 或对外说明。

### 流程骨架

```mermaid
flowchart LR
  index[index_md浏览]
  qmd[qmd混合检索]
  read[打开候选页精读]
  answer[带引用作答]
  backfill[可选回写wiki]
  log[更新index与log]
  index --> qmd
  qmd --> read
  read --> answer
  answer --> backfill
  backfill --> log
```

### 1. 路由与分层检索

| 功能点 | 说明 |
|--------|------|
| **Index-first** | 先读 `wiki/index.md`（或项目等价目录），再下钻；中等规模下减轻对纯向量栈的依赖（见上文「导航与日志」）。 |
| **Scale-out 检索** | 索引不足时启用 **wiki 级**（及 schema 允许时含 `raw/`）全文/语义检索；qmd 作为现成实现：**BM25 + 向量 + 可选 LLM 重排**（见上文「可选增强」）。 |
| **本地优先** | 数据不离开本机 Markdown 树；适合私人 Memex / 离线库（与本 skill 精神一致）。 |

### 2. Agent 可调用面

| 功能点 | 说明 |
|--------|------|
| **MCP 或 CLI** | qmd 提供 **MCP** 或 shell，Agent 以工具调用完成检索，而非裸扫目录猜路径（见上文「可选增强」）。 |
| **候选页/片段定位** | 返回可下钻的候选文件与片段，支撑「选页 → 精读 → 综合」的 Query 形态。 |

### 3. 回答质量与纪律（skill 强制，qmd 不替代）

| 功能点 | 说明 |
|--------|------|
| **带引用回答** | 回答中引用 wiki 页名或链接；关键断言追溯到源或源摘要页（见上文 Query 检查清单）。 |
| **缺口显式化** | 库内无相关页或证据不足时，在回答中说明缺口，不用对话层填补（见上文 Query 检查清单）。 |
| **高价值回写** | 对比表、决策树、跨页综合等可评估落盘为 `synthesis/` 等并更新 index（见上文 Query 检查清单）。 |

### 4. 可观测与复利

| 功能点 | 说明 |
|--------|------|
| **Query 日志** | 在 `wiki/log.md` 记 query 类型条目（示例见上文「`wiki/log.md` 条目示例」：`pages_used`、若有 `backfilled` 路径）。 |
| **与 Lint 协同** | 检索发现的重复主题、孤儿页可反哺 Lint 清单中的合并/补链项（见上文 Lint 检查清单与 Obsidian 实践提示）。 |

### 5. 明确不在本组合内（避免误期望）

- **不**包含：`ingest_source` / `compile_wiki`、自动多页 wiki 编译、raw→wiki 的结构化入库管线。
- qmd 定位为 **读路径检索增强**；沉淀仍依赖本 skill 中的 **Ingest** 约定（手写或项目自配其它管线）。

### 与 skill 原文的对应关系（便于审计）

- Query 节奏与引用纪律：**Query 检查清单（工具无关）**。
- index / log 职能：**导航与日志：职能区分**。
- qmd 能力摘要：**可选增强（gist：CLI / 搜索）**。

## 为何有效（ gist 压缩）

- 知识库难在**簿记**（互链、摘要同步、矛盾标注、跨页一致），而非阅读与思考；人弃维常因**维护负担增长快于收益**。
- Agent **不易厌倦**，单次可触及**多文件**批量更新，使维护边际成本趋低；人类专注策展与意义。

## 常见误区

- 把 wiki 当一次性摘要堆：**不更新索引与相关实体页** → 后续 query 仍像裸 RAG。
- **改写 raw** 作为「整理」：破坏源真相；变更应体现在 wiki 层并可在 log 中说明依据。
- 只问不沉淀：**不复用 ingest/query 产出更新 wiki**，则无法复利。

## 与本仓库的关系

若当前仓库已有面向 **llmwiki MCP / Plan Gate / Graphify** 的专规（如 `.cursor/SKILL/zhishiku`），**专规优先**；本 skill 提供与 gist 对齐的**通用语义**与无专规时的默认行为框架。已接 llmwiki 时，Ingest / 编译的调用顺序与工具细节见上文 **「MCP 实现轮廓（可选）：llmwiki」** 与 `zhishiku`；两处若有冲突，以 **`zhishiku`** 为准。
