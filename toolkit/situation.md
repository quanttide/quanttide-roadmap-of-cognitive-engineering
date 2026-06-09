# Situation 模块设计

## 定位

Situation（情境）是认知构件之一，属于**固体域**——从 gallery 中只读的结构化数据，用于分析"当前发生了什么"。

Situation 由 intention + thought 聚类产生——当多条 intention 和 thought 指向同一个认知域时，它们聚类为一个 situation。

## 数据模型

```
Situation {
    id: Uuid,
    name: String,           // 程序化标识，如 org / infra / think
    label: String,          // 中文标签，如 组织管理 / 基础设施 / 认知工程
    content: SituationContent,
}

SituationContent {
    agenda: String,     // 目标：这个情境要达成什么
    ecology: String,    // 环境状态：当前发生了什么
    frame: String,      // 认知框架：如何理解这个情境
    dynamics: String,   // 演化：这个情境如何变化
}
```

## CLI 中的角色

作为独立 crate，提供 Situation 类型的序列化/反序列化和基础查询。

```
qtcloud-think-situation/     # dependencies: 无
```

与 intention / schema 通过 `name` 字段隐式关联。不引入编译期依赖。

## 开放问题

Situation 是 gallery 的入口点——YAML 文件按 situation name 组织（`docs/gallery/situation/2026-W23/org.yaml`）。reivew 等命令以 situation 列表为骨架展开。数据模型本身不需要变更。
