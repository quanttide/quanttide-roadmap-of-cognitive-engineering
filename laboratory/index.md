# 实验室路线图

## 现状

实验室已重建为全新的 Rust 项目，依赖 `qtcloud-think-cli` / `quanttide-think` / `quanttide-agent`。无遗留代码。

## 目标

将 `data/insight/index.md` 的哲学理念工程化——用正式库的 API 构建端到端认知提取流水线，验证"灵魂编译器"的可实现性。

## 阶段

### Phase 1: 基础集成（M1）

将实验室接入正式库，验证 API 完整可用。

- [x] 初始化 Rust 项目，配置 path 依赖
- [ ] `main.rs` 使用 `Config::from_env()` + `Repo::open()` 读取 journal
- [ ] `Repo::worlds()` / `periods()` / `domains()` 基本查询
- [ ] `analyze::track_evolution()` 演化追踪
- [ ] `Repo::intentions()` / `all_intentions()` / `intention_by_id()` 意向查询

### Phase 2: 周报生成

从 journal 数据生成结构化周报，验证"世界观层"的数据流水线。

- [ ] `Repo::describe()` 数据一致性描述
- [ ] Markdown 周报渲染
- [ ] 跨周差异分析（`diff`）
- [ ] 优先级/风险漂移检测（`drift`）

### Phase 3: 价值观编码（M2）

将 insight 定义的三条原则固化为可运行代码。

- [ ] 高密度过滤器：认知增量评分函数
- [ ] 意图提取器：从原始文本提取共识与意图
- [ ] 结构化约束：输出严格匹配模型字段

### Phase 4: 端到端流水线（M3）

组装完整编译链路。

- [ ] 输入：原始日志 / 混乱文本
- [ ] 世界观层：提取 Mental Model → Situation → Intent
- [ ] 价值观层：过滤 + 排序 + 结构化
- [ ] 输出：结构化 Markdown 报告

## 依赖

| 依赖 | 路径 | 用途 |
|------|------|------|
| `qtcloud-think-cli` | `../../apps/qtcloud-think/src/cli` | Repo 数据访问、analyze 分析 |
| `quanttide-think` | `../../packages/quanttide-think-toolkit/packages/rust` | 核心数据模型 |
| `quanttide-agent` | `../../packages/quanttide-agent-toolkit/packages/rust` | LLM 调用 |
