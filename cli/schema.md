# Schema 模块设计

## 定位

Schema（图式）是认知构件之一，属于**固体域**——从 gallery 中只读的结构化数据，用于分析"认知规律是什么"。

Schema 由 situation 聚类产生——跨多个 situation 中重复出现的模式沉淀为 schema。

## 数据模型

```
Schema {
    id: Uuid,
    name: String,           // 程序化标识，与 Situation 的 name 对应
    label: String,          // 中文标签
    content: SchemaContent,
}

SchemaContent {
    usage: String,                      // 适用场景
    entities: Vec<Entity>,              // 核心概念
    causals: Vec<Causal>,              // 因果规则
    boundaries: Vec<String>,            // 边界条件
    properties: Vec<KeyValue>,         // 属性
    mappings: Vec<Mapping>,            // intent→action 映射
    biases: Vec<Bias>,                 // 常见偏差
}

Entity { name: String, attributes: Vec<String> }
Causal { condition: String, outcome: String }
KeyValue { key: String, value: String }
Mapping { intent: String, action: Value }
Bias { id: Uuid, belief: String, fact: String }
```

## CLI 中的角色

作为独立 crate，提供 Schema 类型的序列化/反序列化和基础查询。

```
qtcloud-think-schema/        # dependencies: 无
```

与 situation 通过 `name` 字段关联。不引入编译期依赖。

## 开放问题

1. Entity 的 attributes 当前为 `Vec<String>`，是否需要更精确的类型？
2. Mapping 的 action 为 `Value`（任意 JSON/YAML），是否需要约束？
