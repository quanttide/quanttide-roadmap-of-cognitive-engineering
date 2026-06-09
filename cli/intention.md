# Intention 模块设计

## 定位

Intention（意图）是认知构件之一，属于**固体域**——从 gallery 中只读的结构化数据，用于分析"人想要达成什么"。

## 数据模型

```
Intention {
    id: Uuid,
    title: String,          // 标题
    description: String,    // 描述
    motivation: String,     // 动因
    agent: Label,           // agent: founder / founder_ai / ...
    level: Level,           // top / middle / bottom
    priority: Priority,     // high / medium / low
    trigger: Trigger,       // persistent / conditional
    risk: Risk,             // high / medium / low
}

Label { name: String, label: String }
Level { name: String, label: String, description: String }
Priority { name: String, label: String, description: String }
Trigger { name: String, label: String, description: String }
Risk { name: String, label: String, description: String }
```

## CLI 中的角色

作为独立 crate，提供 Intention 类型的序列化/反序列化和基础查询。不包含分析逻辑（分析在 CLI 命令中）。

```
qtcloud-think-situation/     # dependencies: 无
qtcloud-think-intention/     # dependencies: 无（与 situation 通过 name 字段隐式关联）
qtcloud-think-schema/        # dependencies: 无
qtcloud-think-thought/       # dependencies: 无
```

四个 crate 互相独立，通过 `name` 字段隐式关联。不引入编译期依赖。

## 问题

1. Label / Level / Priority / Trigger / Risk 这些小类型是否应该抽出为共享 crate？当前设计每个 crate 自包含，但 intention 和 schema 都用到 Label。
2. `level` / `priority` / `risk` 等字段在 Rust 类型中应该用枚举还是字符串？现有 gallery YAML 数据使用 name+label+description 结构，用枚举会丢失 description 字段。
