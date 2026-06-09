# 实验室路线图

## 现状

两个独立实验工具共存：

| 项目 | 路径 | 核心能力 |
|------|------|---------|
| project-11 | `apps/cli/` | gallery 数据驱动的周报/关系/张力/漂移分析 |
| qtcloud-think-situation | `apps/situation/` | 图检索 + LLM 母题发现 |

## 目标

将 situation app 的母题发现能力整合到 project-11 中，放弃 situation 的旧图模型（`situation-graph.json` + petgraph），改用 gallery 数据 + `SituationRelation` 作为数据源。

## 整合计划

1. **关键词索引** — 将 situation 的 CJK 分词器迁移到 cli，对 gallery 的 situation/schema 数据建立可搜索索引
2. **图导航** — 用 `gallery/situation-relation/` 数据替代 `situation-graph.json`，实现 BFS/邻居/冲突路径查询
3. **LLM 母题发现** — 将 situation 的 prompt 模板适配到 gallery 数据，复用 `quanttide-agent` 做 LLM 调用
4. **REPL 发现命令** — 在现有 REPL 中加 `discover` / `explore` 命令入口

## 放弃

- `apps/situation/` 的宠物图（petgraph）+ JSON 图模型
- 硬编码的 `situation-graph.json` 资产
- 独立的 `crates/llm/`（由 `quanttide-agent` 替代）
- 独立的 `crates/core/` 中的数据类型（由 `quanttide-think` 替代）
