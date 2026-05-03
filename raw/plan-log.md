---
description: 执行前方案总簿（供 Plan Gate）；每节内 fenced yaml 表示该需求是否已批准执行
schema_version: "1.0"
---

# 方案总簿（plan-log）

本文件用于**集中记录**经 Plan 讨论后、拟进入 Agent 执行前的方案摘要与批准信息。Cursor 对话内自动生成的 `*.plan.md` 仍由产品保存；此处为仓库 **`raw/` 侧**门禁依据。

## 使用说明

1. **新需求**：新增 `## YYYY-MM-DD — 关键词` 一节，节首紧跟 fenced `yaml` 块，内含 `status: approved` 或 `ready_for_execute: true` 后再写正文。
2. **执行前**：确认本节 yaml 已批准；不要依赖文件顶栏 frontmatter 作为「全本已批」。
3. **llmwiki**：`ingest_source` 请仍指向**独立** `raw/YYYY-MM-DD_*.md`（与需进 wiki 的正文一致），勿默认 ingest 本总簿全文。
4. **历史**：既有 `raw/YYYY-MM-DD_*.md` 单文件在过渡期内仍单独满足门禁，可与本总簿并存。

---

## 2026-05-02 — plan-log 门禁与兼容策略（示例）

```yaml
status: approved
ready_for_execute: true
owner: zhishiku
scope:
  - .cursor/rules/plan-gate.mdc
  - raw/plan-log.md
```

本节为示例：meta 方案见 `raw/2026-05-02_plan-log门禁.md`；正式需求请按同结构追加新 `##` 节。
