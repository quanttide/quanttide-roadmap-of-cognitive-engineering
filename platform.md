# 单一系统路线图

## 方向

平台（qtcloud-think）与实验室（project-11）合并为单一 Rust CLI。CODE 管线作为上层包裹，实验室的数据分层作为下层引擎。Python Provider 保留为可选后端，智能体逻辑逐步迁移到 Rust。

## 阶段 0：合并代码库

将 project-11 设为基座，qtcloud-think 的 CLI 功能作为新模块并入。合并后的目录结构：

```
examples/project-11/
  src/
    models.rs         # 合并两个系统的类型定义
    loader.rs         # 保留，增加本地 SQLite 读取
    query.rs          # 保留，增加内存状态查询
    report.rs         # 保留
    repl.rs           # 扩展，增加 collect/express 命令
    llm.rs            # 保留
    pipeline/         # 新增：CODE 管线
      mod.rs
      clarify.rs
      organize.rs
      distill.rs
      express.rs
    agents/           # 新增：三智能体（从 Python Provider 移植）
      mod.rs
      sower.rs
      observer.rs
      meta.rs
  Cargo.toml          # 增加依赖：rusqlite, ratatui
```

任务 | 描述 | 优先级 | 依赖
|----|------|--------|------|
0.1 | 将 qtcloud-think CLI 的 Cargo.toml 依赖（ratatui, rusqlite, toml, etc.）并入 project-11 | 高 | 无
0.2 | 创建 pipeline/ 模块骨架，定义 CODE 特征和数据结构 | 高 | 0.1
0.3 | 将 Python Provider 的 Sower/Observer/Meta 智能体用 Rust 重写（先用 LLM 调用，逐步本地化） | 中 | 0.2

**产出**：一个能编译的单一 CLI，保留 `report` 等实验室命令，`collect` 等平台命令为 stub。

## 阶段 1：Collect → Report 闭环

用户输入原始想法 → Clarify → 内存中的 Situation/Intention 实例 → 即时调用实验室引擎做关系分析 → 输出结构化反馈。

任务 | 描述 | 优先级 | 依赖
|----|------|--------|------|
1.1 | 实现 Clarify 技能 Rust 版本：从原始文本提取 agenda/ecology/frame/dynamics | 高 | 阶段 0
1.2 | 实现 Organize 技能：新想法与已有 Situation 的关联检测（复用 report.rs 的关键词匹配/实体共享逻辑） | 高 | 1.1
1.3 | 实现 Express 技能：将内存中的 Situation/Intention 输出为 Markdown 笔记 | 中 | 1.1
1.4 | collect 命令完成后自动调用 relate/tension 分析当前领域网络 | 中 | 1.2
1.5 | 本地 SQLite 持久化：collect 的数据存入本地库，下次启动可恢复 | 低 | 0.1

**产出**：在同一个 REPL 里 collect 一条想法后，立即可以用 `relate` 看它和已有领域的关系。

## 阶段 2：Distill 方法论 + 实现

| 任务 | 描述 | 优先级 | 依赖 |
|------|------|--------|------|
| 2.1 | 定义"笔记→situation"映射：agenda/ecology/frame/dynamics 从 Clarify 产出的哪些字段推导 | 高 | 1.1 经验反馈 |
| 2.2 | 定义"笔记→intention"映射：agent/level/trigger 如何从对话上下文中确定 | 高 | 1.1 经验反馈 |
| 2.3 | 实现 Distill 技能：将 Clarify 产物精炼为符合 gallery schema 的结构化 YAML | 中 | 2.1, 2.2 |

**说明**：Distill 的方法论依赖于阶段 1 的实际使用经验——做了 Clarify 之后才知道哪些字段能映射。所以阶段 2 必须在阶段 1 之后。

## 阶段 3：Gallery 写入路径

当前 gallery 是只读的。合并后增加写入路径，但不是直接写 git，而是写入本地 staging 区域，人工确认后再提交。

| 任务 | 描述 | 优先级 | 依赖 |
|------|------|--------|------|
| 3.1 | 定义 gallery staging 格式：本地 YAML + 审核状态（staging/approved/rejected），与笔记的审核状态统一 | 高 | 阶段 2 |
| 3.2 | Distill 产出默认写入 staging 目录，标记为待审核 | 中 | 3.1 |
| 3.3 | 实现 gallery 提交指令：将 staging 中已审核的 YAML 复制到 gallery 并执行 git commit + push | 中 | 3.2 |

**说明**：当前子模块工作流的摩擦（双提交）不会消失，但被压缩到"确认提交"这一个环节。

## 阶段 4：废弃 Python Provider

| 任务 | 描述 | 优先级 | 依赖 |
|------|------|--------|------|
| 4.1 | 将 Provider 中剩下的技能（Organize 深度实现、Express 增强）移植完毕 | 低 | 1.2, 1.3 |
| 4.2 | 确认 Studio 客户端可以直接调用 Rust CLI 的 HTTP 接口（或者 Studio 改为独立项目） | 低 | 4.1 |
| 4.3 | 归档 app/qtcloud-think 子模块 | 低 | 4.2 |

## 子模块定位重新评估

合并后，九宫格子模块的角色发生变化：

| 子模块 | 当前角色 | 合并后角色 |
|--------|---------|-----------|
| data/archive | 终点 | 不变 |
| data/context | 起点 | 不变 |
| **data/insight** | 手动记录 | 改为系统自动从分析过程抽取 |
| **data/intention** | 手动记录 | 改为系统从 collect + clarify 产物自动生成 |
| **data/report** | 手动记录 | 改为 report 命令的附带产出 |
| **data/roadmap** | 手动记录 | 保留人工维护（路线图是决策，不是自动化产物） |
| **docs/gallery** | 唯一事实源 | 改为 staging + approved 双区；system 写 staging，human 确认后移入 approved |

## 时间线

- 阶段 0：优先推进（代码合并决定所有后续步骤）
- 阶段 1：阶段 0 后立即开始，预期产出周期 1-2 周
- 阶段 2：阶段 1 实际使用 1 周后开始方法论定义
- 阶段 3：阶段 2 完成后
- 阶段 4：无明确时间，取决于 Provider 的使用频率和迁移成本
