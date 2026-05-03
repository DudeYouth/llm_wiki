---
title: SKILL 修订：llmwiki MCP 统一承担 wiki 相关能力
date: 2026-05-02
status: approved
ready_for_execute: true
owner: zhishiku
scope:
  - .cursor/SKILL/zhishiku/SKILL.md
---

## 需求背景

- 将知识库维护 Skill 与 llmwiki MCP 对齐：除正文编译外，凡 MCP 已提供的 wiki 相关能力均通过工具调用完成。
- 消除 `mcp.json` 中 `llmwiki` 与仓库 `user-llmwiki` 描述符命名的混淆。

## 变更摘要

1. **§1**：补充命名对照；生成唯一入口 `ingest_source` → `compile_wiki`；非生成类能力默认走 MCP 工具矩阵；禁止 CLI/手搓/用裸读替代 MCP 语义；列明例外（`wiki/log.md`、Graphify、Plan Gate 读 `raw/`、无写入工具时的导航页手改、用户声明 MCP 不可用）。
2. **§2 步骤2**：编译须通过已连接 MCP `llmwiki`，禁止默认用本地 CLI。
3. **§3**：开篇命名说明；步骤 4 升格为编译后默认 MCP 检查路径；步骤 5 明确手改边界。
4. **§4**：可选一句说明图谱与 llmwiki 互补、互不替代。

## 验收

- §1 含工具矩阵与例外；§3 步骤 4 为默认路径；全文有一处 `llmwiki` / `user-llmwiki` 对照说明。
