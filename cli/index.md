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
| Situation | 环境表征：发生了什么 | thought → clarify | 按领域聚合 |
| Intention | 目标表征：想要达成什么 | thought → clarify | 按 situation 归属 |
| Schema | 模式表征：认知规律 | thought → clarify + 跨情境沉淀 | 按 situation 归属 |

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
