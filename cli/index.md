# CLI 总体设计

## 架构

```
apps/qtcloud-think/
  crates/
    qtcloud-think-thought/       # Thought（想法）— 流体域
    qtcloud-think-situation/     # Situation（情境）
    qtcloud-think-intention/     # Intention（意图）     — 固体域
    qtcloud-think-schema/        # Schema（图式）
```

四个 crate 互相独立，不引入编译期依赖。通过 `name` 字段（String）隐式关联。例如 Intention 通过 `situation_name` 指向 Situation，Schema 通过 `name` 指向 Situation。

## 两个域

| 域 | 操作对象 | 数据状态 | 命令 |
|----|---------|---------|------|
| **流体**（thought） | 原始想法 | 加工中，stdin pipe | `thought note` |
| **固体**（situation / intention / schema） | 已入库的结构化数据 | 定型后，gallery 只读 | 待定义 |

流体加工产出 staging YAML，人工确认后移入 gallery 成为固体。

## 四个数据类型

| 类型 | 本质 | 来源 | 粒度 |
|------|------|------|------|
| Thought | 原始输入的文本，带模糊标签 | 用户 stdin | 单条 |
| Intention | 目标表征：想要达成什么 | thought → clarify（从想法中提取） | 单条，可跨 situation |
| Situation | 环境表征：发生了什么 | intention + thought 聚类产生 | 按意图和想法聚类 |
| Schema | 模式表征：认知规律 | situation 聚类产生 | 跨 situation 沉淀 |

## 构件关系

```
Thought ──clarify──→ Intention
                          \
Thought ──clarify──→ Intention ──cluster──→ Situation
Thought ──clarify──→ Intention                          ↘
...                                                    Schema
Situation ──repeated──→ pattern ──abstract──→            ↗
```

- **Thought → Intention**：clarify 阶段从原始想法中提取意图。Intention 是 Thought 最直接的产物
- **Thought + Intention → Situation**：多条 thought 和 intention 指向同一个认知域时聚类为一个 situation
- **Situation → Situation**：situation 之间存在关系（支持/冲突/触发/演化等），由 later analyze 计算
- **Situation → Schema**：跨多个 situation 重复出现的模式抽象为 schema。共享 entities / causals / boundaries 的共同模式沉淀为 schema
- **Schema ⇄ Intention**：schema 的 mappings 字段记录 intent→action 映射，是 intention 的模式化总结

## 通用类型问题

四个 crate 都用到小型标签类型（Label / Level / Priority / Trigger / Risk）。选项：

A. 每个 crate 自包含，重复定义 → 简单，无共享依赖成本
B. 抽出 `qtcloud-think-types` 作为第五个 crate → 干净，但多了个 crate

当前倾向 A（等有代码后再评估是否抽取）。

## 序列化格式

crater 只定义类型 + 序列化/反序列化。所有 crate 支持：

- 从 YAML 反序列化（读 gallery）
- 序列化为 JSON（stdout 输出给 pipe）
- 序列化为 YAML（distill 产出去 staging）

## CLI 命令

（待定——这四个 crate 只是数据层，CLI 命令层还在设计中）
