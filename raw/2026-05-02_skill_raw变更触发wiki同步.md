---
title: SKILL 修订：raw 方案新增或更新须触发 llmwiki 同步 wiki
date: 2026-05-02
status: approved
ready_for_execute: true
owner: zhishiku
scope:
  - .cursor/SKILL/zhishiku/SKILL.md
---

## 需求背景

- 避免将「落盘 raw」理解为仅首次创建才需编译；同一方案文件覆盖更新后也须同步 `wiki/`。

## 变更摘要

1. §1：增加 raw 新建/更新后与 `ingest_source` → `compile_wiki` 的强制绑定（来源限定为 Plan 或门禁已批准写入）。
2. §2 步骤2：触发语改为「新增或更新后」。
3. §3：开篇增加「raw 更新需反映到 wiki」触发条件；步骤 1 补充更新场景仍须校验 frontmatter 再走 MCP。

## 验收

- 至少两处写明「新增或更新」与 MCP 链路；不与「raw 不要随意手改」矛盾。
